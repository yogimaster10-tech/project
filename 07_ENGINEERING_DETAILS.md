# TESKEL — Backend Logic & Frontend Deep Specification
## Every Service, Every Flow, Every Component — Engineer-Level Detail

---

## 1. BACKEND SERVICE LOGIC — Every Service

### 1.1 Checkout Service (checkout.service.ts)

```typescript
// apps/api/src/services/checkout.service.ts

export class CheckoutService {
  
  async createSession(input: CreateCheckoutInput): Promise<CheckoutSession> {
    // 1. Validate product exists and is active
    const product = await db.query.products.findFirst({
      where: and(eq(products.id, input.productId), eq(products.status, 'active')),
      with: { prices: true, org: true },
    });
    if (!product) throw new NotFoundError('Product not found or not active');
    
    // 2. Validate price belongs to product
    const price = product.prices.find(p => p.id === input.priceId);
    if (!price) throw new ValidationError('Price does not belong to product');
    
    // 3. Validate discount code if provided
    let discountAmount = 0;
    if (input.discountCode) {
      const discount = await this.validateDiscount({
        code: input.discountCode,
        orgSlug: product.org.slug,
        productId: input.productId,
        customerEmail: input.customerEmail,
      });
      discountAmount = this.calculateDiscount(discount, price.priceCents);
    }
    
    // 4. Calculate tax (if Stripe Tax enabled for this org)
    let taxAmount = 0;
    if (product.org.metadata?.stripeTaxEnabled) {
      taxAmount = await this.calculateTax({
        amount: price.priceCents - discountAmount,
        currency: price.currency,
        productType: product.type,
        customerCountry: input.customerCountry, // from IP geolocation
      });
    }
    
    // 5. Calculate processing fee (if passed to customer)
    const processingFee = Math.round(price.priceCents * 0.029 + 30); // Stripe fee
    
    // 6. Create Stripe Checkout Session
    const stripeSession = await stripe.checkout.sessions.create({
      mode: price.interval ? 'subscription' : 'payment',
      payment_method_types: ['card'],
      allow_promotion_codes: !input.discountCode, // show Stripe promo input only if no code
      line_items: [{
        price: price.stripePriceId,
        quantity: input.quantity || 1,
      }],
      discounts: input.discountCode ? [{ coupon: await this.getStripeCouponId(input.discountCode) }] : undefined,
      customer_email: input.customerEmail,
      success_url: input.successUrl,
      cancel_url: input.cancelUrl,
      metadata: {
        teskel_product_id: input.productId,
        teskel_price_id: input.priceId,
        teskel_org_id: product.orgId,
        teskel_funnel_id: input.funnelId || '',
        teskel_funnel_step_id: input.funnelStepId || '',
        teskel_affiliate_code: input.affiliateCode || '',
        teskel_utm_source: input.utmSource || '',
        teskel_utm_medium: input.utmMedium || '',
        teskel_utm_campaign: input.utmCampaign || '',
      },
      // Stripe Connect: destination charge (creator receives net)
      payment_intent_data: price.interval ? undefined : {
        application_fee_amount: this.calculatePlatformFee(price.priceCents, product.org.plan),
        transfer_data: { destination: product.org.stripeAccountId },
      },
      subscription_data: price.interval ? {
        application_fee_percent: this.getPlatformFeePercent(product.org.plan),
        transfer_data: { destination: product.org.stripeAccountId },
      } : undefined,
    });
    
    // 7. Store session in Redis (TTL 30 min)
    await redis.setex(
      `checkout:session:${stripeSession.id}`,
      1800,
      JSON.stringify({
        productId: input.productId,
        priceId: input.priceId,
        orgId: product.orgId,
        discountCode: input.discountCode,
        affiliateCode: input.affiliateCode,
        funnelId: input.funnelId,
      })
    );
    
    // 8. Log analytics
    await this.logAnalytics(product.orgId, 'checkout_started', {
      productId: input.productId,
      priceId: input.priceId,
      totalCents: price.priceCents,
    });
    
    return { sessionId: stripeSession.id, url: stripeSession.url };
  }
  
  async handleStripeWebhook(event: Stripe.Event): Promise<void> {
    switch (event.type) {
      case 'checkout.session.completed':
        await this.handleCheckoutCompleted(event.data.object as Stripe.Checkout.Session);
        break;
      case 'checkout.session.expired':
        await this.handleCheckoutExpired(event.data.object as Stripe.Checkout.Session);
        break;
      // ... other events handled by subscription.service, payout.service
    }
  }
  
  async handleCheckoutCompleted(session: Stripe.Checkout.Session): Promise<void> {
    const sessionData = JSON.parse(await redis.get(`checkout:session:${session.id}`));
    if (!sessionData) {
      logger.error('Checkout session data not found in Redis', { sessionId: session.id });
      return; // Idempotent: if already processed, skip
    }
    
    // Check idempotency: has this session been processed?
    const existingOrder = await db.query.orders.findFirst({
      where: eq(orders.stripeSessionId, session.id),
    });
    if (existingOrder) return; // Already processed
    
    const { productId, priceId, orgId, discountCode, affiliateCode, funnelId, funnelStepId } = sessionData;
    
    // 1. Find or create customer
    const customer = await this.findOrCreateCustomer({
      orgId,
      email: session.customer_email!,
      name: session.customer_details?.name,
      stripeCustomerId: session.customer as string,
    });
    
    // 2. Get product + price
    const product = (await db.query.products.findFirst({
      where: eq(products.id, productId),
      with: { prices: true },
    }))!;
    const price = product.prices.find(p => p.id === priceId)!;
    
    // 3. Calculate amounts
    const subtotalCents = price.priceCents;
    const discountCents = session.total_details?.amount_discount ? Math.abs(session.total_details.amount_discount) : 0;
    const taxCents = session.total_details?.amount_tax || 0;
    const totalCents = session.amount_total;
    
    // 4. Create order
    const [order] = await db.insert(orders).values({
      orgId,
      customerId: customer.id,
      customerEmail: customer.email,
      customerName: customer.name,
      status: 'paid',
      subtotalCents,
      discountCents,
      taxCents,
      totalCents,
      currency: price.currency,
      stripeSessionId: session.id,
      stripePaymentIntentId: session.payment_intent as string,
      funnelId: funnelId || null,
      funnelStepId: funnelStepId || null,
      affiliateCode: affiliateCode || null,
      utmSource: session.metadata?.teskel_utm_source || null,
      utmMedium: session.metadata?.teskel_utm_medium || null,
      utmCampaign: session.metadata?.teskel_utm_campaign || null,
    }).returning();
    
    // 5. Create order item
    const [orderItem] = await db.insert(orderItems).values({
      orderId: order.id,
      productId,
      priceId,
      productName: product.name,
      priceName: price.name,
      amountCents: subtotalCents,
      productType: product.type,
    }).returning();
    
    // 6. Handle product-type-specific logic
    await this.processProductDelivery(product, price, order, orderItem, customer);
    
    // 7. Calculate affiliate commission
    if (affiliateCode) {
      await this.processAffiliateCommission(affiliateCode, order, orgId);
    }
    
    // 8. Create payout schedule
    await payoutService.schedulePayout(order, orgId);
    
    // 9. Send receipt email
    await emailService.sendReceipt(customer, order, orderItem, product);
    
    // 10. Fire creator webhooks
    await webhookService.fireEvent(orgId, 'order.paid', { order, orderItem, customer });
    
    // 11. Log analytics
    await analyticsService.log(orgId, 'checkout_completed', { orderId: order.id, totalCents });
    
    // 12. Update denormalized product stats
    await db.update(products)
      .set({ 
        totalSales: sql`${products.totalSales} + 1`,
        totalRevenueCents: sql`${products.totalRevenueCents} + ${totalCents}`,
      })
      .where(eq(products.id, productId));
    
    // 13. Update customer stats
    await db.update(customers)
      .set({
        lifetimeSpendCents: sql`${customers.lifetimeSpendCents} + ${totalCents}`,
        purchaseCount: sql`${customers.purchaseCount} + 1`,
      })
      .where(eq(customers.id, customer.id));
    
    // 14. Clean up Redis session
    await redis.del(`checkout:session:${session.id}`);
  }
  
  async processProductDelivery(
    product: Product, price: ProductPrice,
    order: Order, orderItem: OrderItem, customer: Customer
  ): Promise<void> {
    switch (product.type) {
      case 'download':
      case 'prompt_pack':
      case 'pwyw':
        // Create access grant
        await db.insert(accessGrants).values({
          customerId: customer.id,
          productId: product.id,
          grantType: 'purchase',
          sourceId: order.id,
          scope: 'full',
          status: 'active',
        });
        // Create delivery record
        await db.insert(deliveries).values({
          orderId: order.id,
          customerId: customer.id,
          productId: product.id,
          deliveryType: 'file_download',
          payload: { fileUrls: 'pending' }, // Generated on demand
          status: 'delivered',
          deliveredAt: new Date(),
        });
        break;
        
      case 'license':
        // Generate license key
        const licenseKey = generateLicenseKey(); // TSKL-XXXX-XXXX-XXXX
        const [license] = await db.insert(licenseKeys).values({
          orgId: product.orgId,
          productId: product.id,
          orderItemId: orderItem.id,
          key: licenseKey,
          status: 'active',
          maxActivations: product.metadata?.maxActivations || 1,
          features: product.metadata?.licenseFeatures || {},
          expiresAt: product.metadata?.licenseExpiresAt || null,
        }).returning();
        // Update order item with license key
        await db.update(orderItems)
          .set({ licenseKeyId: license.id })
          .where(eq(orderItems.id, orderItem.id));
        // Create delivery
        await db.insert(deliveries).values({
          orderId: order.id,
          customerId: customer.id,
          productId: product.id,
          deliveryType: 'license_key',
          payload: { licenseKey, licenseKeyId: license.id },
          status: 'delivered',
          deliveredAt: new Date(),
        });
        break;
        
      case 'community':
        // Create access grant
        await db.insert(accessGrants).values({
          customerId: customer.id,
          productId: product.id,
          grantType: 'purchase',
          sourceId: order.id,
          status: 'active',
        });
        // Send invite via bot
        const inviteUrl = await communityService.sendInvite(product, customer);
        await db.insert(deliveries).values({
          orderId: order.id,
          customerId: customer.id,
          productId: product.id,
          deliveryType: 'invite_link',
          payload: { inviteUrl, platform: product.metadata?.platform },
          status: 'delivered',
          deliveredAt: new Date(),
        });
        break;
        
      case 'micro_saas':
        // Create access grant
        await db.insert(accessGrants).values({
          customerId: customer.id,
          productId: product.id,
          grantType: 'purchase',
          sourceId: order.id,
          status: 'active',
        });
        // Fire provision webhook
        if (product.metadata?.provisionWebhookUrl) {
          await deliveryQueue.add('webhook_provision', {
            url: product.metadata.provisionWebhookUrl,
            payload: { customer: customer.email, product: product.name, orderId: order.id },
          });
        }
        await db.insert(deliveries).values({
          orderId: order.id,
          customerId: customer.id,
          productId: product.id,
          deliveryType: 'account_access',
          payload: { message: 'Account will be provisioned shortly' },
          status: 'pending',
        });
        break;
        
      case 'service':
        // Create delivery with booking info
        await db.insert(deliveries).values({
          orderId: order.id,
          customerId: customer.id,
          productId: product.id,
          deliveryType: 'booking_confirm',
          payload: { 
            calendarLink: product.metadata?.calendarLink,
            duration: product.metadata?.bookingDurationMinutes,
          },
          status: 'delivered',
          deliveredAt: new Date(),
        });
        break;
        
      case 'bundle':
        // Get bundle items
        const bundleItems = await db.query.productBundleItems.findMany({
          where: eq(productBundleItems.bundleProductId, product.id),
        });
        // Recursively process each bundle item
        for (const item of bundleItems) {
          const itemProduct = (await db.query.products.findFirst({
            where: eq(products.id, item.itemProductId),
            with: { prices: true },
          }))!;
          const itemPrice = item.itemPriceId 
            ? itemProduct.prices.find(p => p.id === item.itemPriceId)!
            : itemProduct.prices[0];
          await this.processProductDelivery(itemProduct, itemPrice, order, orderItem, customer);
        }
        break;
        
      case 'course':
        await db.insert(accessGrants).values({
          customerId: customer.id,
          productId: product.id,
          grantType: 'purchase',
          sourceId: order.id,
          status: 'active',
        });
        await db.insert(deliveries).values({
          orderId: order.id,
          customerId: customer.id,
          productId: product.id,
          deliveryType: 'stream_access',
          payload: { contentUrls: product.metadata?.contentUrls || [] },
          status: 'delivered',
          deliveredAt: new Date(),
        });
        break;
        
      case 'tiered':
        await db.insert(accessGrants).values({
          customerId: customer.id,
          productId: product.id,
          grantType: 'purchase',
          sourceId: order.id,
          scope: 'partial',
          tier: price.name.toLowerCase(),
          features: price.metadata?.features || [],
          status: 'active',
        });
        break;
    }
  }
  
  calculatePlatformFee(amountCents: number, plan: string): number {
    const fees = { free: 0.05, pro: 0.03, scale: 0.015, enterprise: 0.005 };
    return Math.round(amountCents * (fees[plan] || fees.free));
  }
  
  getPlatformFeePercent(plan: string): number {
    const fees = { free: 5, pro: 3, scale: 1.5, enterprise: 0.5 };
    return fees[plan] || fees.free;
  }
}
```

### 1.2 License Service (license.service.ts)

```typescript
export class LicenseService {
  
  async validate(key: string): Promise<LicenseValidationResult> {
    // 1. Check Redis cache
    const cached = await redis.get(`license:validate:${key}`);
    if (cached) return JSON.parse(cached);
    
    // 2. Query database
    const license = await db.query.licenseKeys.findFirst({
      where: eq(licenseKeys.key, key),
      with: { product: { columns: { id: true, name: true } } },
    });
    
    if (!license) {
      const result = { valid: false, reason: 'not_found' };
      await redis.setex(`license:validate:${key}`, 60, JSON.stringify(result));
      return result;
    }
    
    // 3. Check status
    const isValid = license.status === 'active' 
      && (!license.expiresAt || license.expiresAt > new Date());
    
    const result = {
      valid: isValid,
      productId: license.productId,
      productName: license.product.name,
      status: license.status,
      features: license.features,
      expiresAt: license.expiresAt,
      activationCount: license.activationCount,
      maxActivations: license.maxActivations,
    };
    
    // 4. Cache result
    await redis.setex(`license:validate:${key}`, 60, JSON.stringify(result));
    
    return result;
  }
  
  async activate(key: string, input: ActivateInput): Promise<ActivateResult> {
    // 1. Get license
    const license = await db.query.licenseKeys.findFirst({
      where: eq(licenseKeys.key, key),
    });
    if (!license) throw new NotFoundError('License not found');
    if (license.status !== 'active') throw new ValidationError('License is not active');
    
    // 2. Check max activations
    if (license.maxActivations && license.activationCount >= license.maxActivations) {
      throw new ValidationError('Max activations reached', { 
        maxActivations: license.maxActivations,
        current: license.activationCount,
      });
    }
    
    // 3. Check if already activated on this machine
    const existing = await db.query.licenseActivations.findFirst({
      where: and(
        eq(licenseActivations.licenseKeyId, license.id),
        eq(licenseActivations.fingerprint, input.fingerprint),
        eq(licenseActivations.status, 'active'),
      ),
    });
    if (existing) {
      // Update heartbeat
      await db.update(licenseActivations)
        .set({ lastHeartbeat: new Date() })
        .where(eq(licenseActivations.id, existing.id));
      return { success: true, activationId: existing.id, remaining: this.getRemaining(license) };
    }
    
    // 4. Create activation
    const [activation] = await db.insert(licenseActivations).values({
      licenseKeyId: license.id,
      fingerprint: input.fingerprint,
      ipAddress: input.ipAddress,
      userAgent: input.userAgent,
      label: input.label,
      status: 'active',
    }).returning();
    
    // 5. Increment activation count
    await db.update(licenseKeys)
      .set({ activationCount: sql`${licenseKeys.activationCount} + 1` })
      .where(eq(licenseKeys.id, license.id));
    
    // 6. Invalidate cache
    await redis.del(`license:validate:${key}`);
    
    // 7. Fire webhook
    await webhookService.fireEvent(license.orgId, 'license.activated', {
      key: license.key, fingerprint: input.fingerprint, label: input.label,
    });
    
    return { success: true, activationId: activation.id, remaining: this.getRemaining(license) - 1 };
  }
  
  async deactivate(activationId: string, key: string): Promise<void> {
    await db.update(licenseActivations)
      .set({ status: 'deactivated' })
      .where(eq(licenseActivations.id, activationId));
    
    await db.update(licenseKeys)
      .set({ activationCount: sql`GREATEST(${licenseKeys.activationCount} - 1, 0)` })
      .where(eq(licenseKeys.key, key));
    
    await redis.del(`license:validate:${key}`);
  }
  
  async detectAbuse(key: string): Promise<boolean> {
    // Check if key activated from >5 unique IPs in last hour
    const recentActivations = await db.query.licenseActivations.findMany({
      where: and(
        eq(licenseActivations.licenseKeyId, (await this.getByKey(key)).id),
        gt(licenseActivations.activatedAt, sql`NOW() - INTERVAL '1 hour'`),
        eq(licenseActivations.status, 'active'),
      ),
    });
    const uniqueIPs = new Set(recentActivations.map(a => a.ipAddress));
    if (uniqueIPs.size > 5) {
      // Suspend license and notify creator
      await db.update(licenseKeys)
        .set({ status: 'suspended' })
        .where(eq(licenseKeys.key, key));
      await redis.del(`license:validate:${key}`);
      await webhookService.fireEvent(/* ... */, 'license.suspended', { key, reason: 'suspected_sharing' });
      return true;
    }
    return false;
  }
  
  private getRemaining(license: LicenseKey): number {
    if (!license.maxActivations) return Infinity;
    return license.maxActivations - license.activationCount;
  }
}
```

### 1.3 Payout Service (payout.service.ts)

```typescript
export class PayoutService {
  
  async schedulePayout(order: Order, orgId: string): Promise<void> {
    const org = await db.query.organizations.findFirst({
      where: eq(organizations.id, orgId),
    });
    
    // Calculate amounts
    const grossAmount = order.subtotalCents;
    const platformFee = this.calculatePlatformFee(grossAmount, org!.plan);
    const processingFee = Math.round(grossAmount * 0.029 + 30); // Stripe fee
    const netAmount = grossAmount - platformFee - processingFee;
    
    // Check for bundle splits
    const splits: PayoutSplit[] = [];
    const orderItemsList = await db.query.orderItems.findMany({
      where: eq(orderItems.orderId, order.id),
    });
    
    for (const item of orderItemsList) {
      const bundleItems = await db.query.productBundleItems.findMany({
        where: eq(productBundleItems.bundleProductId, item.productId),
      });
      for (const bi of bundleItems) {
        if (bi.ownerOrgId && bi.ownerOrgId !== orgId && bi.ownerSharePercent) {
          const splitAmount = Math.round(item.amountCents * Number(bi.ownerSharePercent) / 100);
          const splitOrg = await db.query.organizations.findFirst({
            where: eq(organizations.id, bi.ownerOrgId),
          });
          if (splitOrg?.stripeAccountId) {
            splits.push({
              recipientOrgId: bi.ownerOrgId,
              recipientStripeAccountId: splitOrg.stripeAccountId,
              amountCents: splitAmount,
              description: `Bundle split for ${item.productName}`,
              type: 'creator',
            });
          }
        }
      }
    }
    
    // Affiliate split
    if (order.affiliateId) {
      const affiliate = await db.query.affiliates.findFirst({
        where: eq(affiliates.id, order.affiliateId),
      });
      if (affiliate) {
        const commission = affiliate.commissionType === 'percentage'
          ? Math.round(grossAmount * affiliate.commissionValue / 100)
          : affiliate.commissionValue;
        splits.push({
          recipientOrgId: orgId, // Affiliate commission goes to org, tracked separately
          recipientStripeAccountId: '', // Paid from org's balance
          amountCents: commission,
          description: `Affiliate commission for ${affiliate.code}`,
          type: 'affiliate',
        });
      }
    }
    
    // Schedule payout based on org's payout schedule
    const scheduledAt = this.getNextPayoutDate(org!.payoutSchedule);
    
    await db.insert(payoutSchedules).values({
      orgId,
      orderId: order.id,
      grossAmountCents: grossAmount,
      platformFeeCents: platformFee,
      processingFeeCents: processingFee,
      netAmountCents: netAmount,
      splits,
      status: 'pending',
      scheduledAt,
    });
  }
  
  async processScheduledPayouts(): Promise<void> {
    // Called by cron worker every hour
    const pending = await db.query.payoutSchedules.findMany({
      where: and(
        eq(payoutSchedules.status, 'pending'),
        lte(payoutSchedules.scheduledAt, new Date()),
      ),
      limit: 100,
    });
    
    for (const payout of pending) {
      try {
        // Get org's Stripe Connect account
        const org = (await db.query.organizations.findFirst({
          where: eq(organizations.id, payout.orgId),
        }))!;
        
        if (org.stripeAccountStatus !== 'active') {
          logger.warn('Org Stripe account not active, skipping payout', { orgId: payout.orgId });
          continue;
        }
        
        // Process main payout via Stripe Transfer
        const transfer = await stripe.transfers.create({
          amount: payout.netAmountCents,
          currency: 'usd',
          destination: org.stripeAccountId!,
          metadata: { teskel_payout_id: payout.id, teskel_order_id: payout.orderId },
        });
        
        // Process bundle splits
        for (const split of payout.splits) {
          if (split.type === 'creator' && split.recipientStripeAccountId) {
            await stripe.transfers.create({
              amount: split.amountCents,
              currency: 'usd',
              destination: split.recipientStripeAccountId,
              transfer_group: `teskel_${payout.orderId}`,
              metadata: { teskel_payout_id: payout.id, split_org: split.recipientOrgId },
            });
          }
        }
        
        // Update payout status
        await db.update(payoutSchedules)
          .set({ status: 'completed', stripeTransferId: transfer.id, completedAt: new Date() })
          .where(eq(payoutSchedules.id, payout.id));
        
        // Fire webhook
        await webhookService.fireEvent(payout.orgId, 'payout.completed', { payoutId: payout.id, amount: payout.netAmountCents });
        
      } catch (error) {
        logger.error('Payout failed', { payoutId: payout.id, error });
        await db.update(payoutSchedules)
          .set({ status: 'failed' })
          .where(eq(payoutSchedules.id, payout.id));
      }
    }
  }
  
  private getNextPayoutDate(schedule: string): Date {
    const now = new Date();
    switch (schedule) {
      case 'daily': return new Date(now.getTime() + 24 * 60 * 60 * 1000);
      case 'weekly': // Next Monday
        const daysUntilMonday = (8 - now.getDay()) % 7 || 7;
        return new Date(now.getTime() + daysUntilMonday * 24 * 60 * 60 * 1000);
      case 'monthly': // 1st of next month
        return new Date(now.getFullYear(), now.getMonth() + 1, 1);
      default: return new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);
    }
  }
}
```

### 1.4 Access Service (access.service.ts)

```typescript
export class AccessService {
  
  async check(input: AccessCheckInput): Promise<AccessCheckResult> {
    const cacheKey = `access:check:${input.customerEmail || input.customerId}:${input.productId}`;
    const cached = await redis.get(cacheKey);
    if (cached) return JSON.parse(cached);
    
    // 1. Find customer
    let customerId = input.customerId;
    if (!customerId && input.customerEmail) {
      const customer = await db.query.customers.findFirst({
        where: eq(customers.email, input.customerEmail),
      });
      customerId = customer?.id;
    }
    
    // 2. Check access grants
    if (customerId) {
      const grants = await db.query.accessGrants.findMany({
        where: and(
          eq(accessGrants.customerId, customerId),
          eq(accessGrants.productId, input.productId),
          eq(accessGrants.status, 'active'),
        ),
      });
      
      // Filter out expired grants
      const activeGrants = grants.filter(g => !g.expiresAt || g.expiresAt > new Date());
      
      if (activeGrants.length > 0) {
        const grant = activeGrants[0]; // Use first active grant
        const result = {
          allowed: true,
          grantType: grant.grantType,
          tier: grant.tier,
          features: grant.features,
          expiresAt: grant.expiresAt,
          upgradeOptions: [],
        };
        await redis.setex(cacheKey, 300, JSON.stringify(result));
        return result;
      }
    }
    
    // 3. Check license key if provided
    if (input.licenseKey) {
      const validation = await licenseService.validate(input.licenseKey);
      if (validation.valid && validation.productId === input.productId) {
        const result = {
          allowed: true,
          grantType: 'license',
          tier: null,
          features: validation.features,
          expiresAt: validation.expiresAt,
          upgradeOptions: [],
        };
        await redis.setex(cacheKey, 300, JSON.stringify(result));
        return result;
      }
    }
    
    // 4. No access — return upgrade options
    const product = await db.query.products.findFirst({
      where: eq(products.id, input.productId),
      with: { prices: true },
    });
    
    const upgradeOptions = product?.prices
      .filter(p => p.priceCents > 0)
      .map(p => ({ priceId: p.id, name: p.name, amountCents: p.priceCents })) || [];
    
    const result = {
      allowed: false,
      grantType: null,
      tier: null,
      features: [],
      expiresAt: null,
      upgradeOptions,
    };
    
    await redis.setex(cacheKey, 300, JSON.stringify(result));
    return result;
  }
}
```

### 1.5 Email Service (email.service.ts)

```typescript
export class EmailService {
  
  async sendReceipt(customer: Customer, order: Order, item: OrderItem, product: Product): Promise<void> {
    const html = await render(PurchaseReceiptEmail({ 
      customerName: customer.name || customer.email,
      productName: product.name,
      priceName: item.priceName,
      amount: formatCurrency(item.amountCents, order.currency),
      orderId: order.id,
      downloadUrl: `${process.env.NEXT_PUBLIC_APP_URL}/purchases/${order.id}`,
    }));
    
    await resend.emails.send({
      from: `${process.env.EMAIL_FROM_NAME} <${process.env.EMAIL_FROM}>`,
      to: customer.email,
      subject: `Receipt for ${product.name}`,
      html,
    });
  }
  
  async sendLicenseDelivery(customer: Customer, licenseKey: string, product: Product): Promise<void> {
    const html = await render(LicenseDeliveryEmail({
      customerName: customer.name || customer.email,
      productName: product.name,
      licenseKey,
      validateUrl: `${process.env.NEXT_PUBLIC_API_URL}/v1/licenses/${licenseKey}/validate`,
    }));
    
    await resend.emails.send({
      from: `${process.env.EMAIL_FROM_NAME} <${process.env.EMAIL_FROM}>`,
      to: customer.email,
      subject: `Your license key for ${product.name}`,
      html,
    });
  }
  
  async processAbandonedCheckouts(): Promise<void> {
    // Called by cron every hour
    // Find checkout sessions that were created >1h ago but not completed
    const sessions = await redis.keys('checkout:session:*');
    for (const sessionKey of sessions) {
      const data = JSON.parse((await redis.get(sessionKey))!);
      const createdAt = new Date(data.createdAt);
      if (Date.now() - createdAt.getTime() > 60 * 60 * 1000) {
        // Send abandoned checkout email
        if (data.customerEmail) {
          await this.sendAbandonedCheckoutEmail(data.customerEmail, data.productId);
        }
        await redis.del(sessionKey);
      }
    }
  }
  
  async processEmailSequences(): Promise<void> {
    // Called by cron every 15 minutes
    // Find sequence steps that should be sent now
    // Check: customer triggered sequence, delay_hours elapsed, not yet sent
    // Implementation: track sequence progress in Redis per customer
  }
}
```

### 1.6 Webhook Service (webhook.service.ts)

```typescript
export class WebhookService {
  
  async fireEvent(orgId: string, eventType: string, data: any): Promise<void> {
    const endpoints = await db.query.webhookEndpoints.findMany({
      where: and(
        eq(webhookEndpoints.orgId, orgId),
        eq(webhookEndpoints.active, true),
      ),
    });
    
    for (const endpoint of endpoints) {
      if (!endpoint.events.includes(eventType) && !endpoint.events.includes('*')) continue;
      
      const payload = {
        id: crypto.randomUUID(),
        type: eventType,
        timestamp: new Date().toISOString(),
        data,
      };
      
      const signature = crypto
        .createHmac('sha256', endpoint.secret)
        .update(JSON.stringify(payload))
        .digest('hex');
      
      await db.insert(webhookDeliveries).values({
        endpointId: endpoint.id,
        eventType,
        payload,
        status: 'pending',
      });
      
      // Add to delivery queue
      await deliveryQueue.add('webhook', {
        endpointId: endpoint.id,
        url: endpoint.url,
        payload,
        signature,
        attempts: 0,
        maxAttempts: 5,
      });
    }
  }
  
  async deliverWebhook(job: WebhookJob): Promise<void> {
    try {
      const response = await fetch(job.url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-TESKEL-Signature': `sha256=${job.signature}`,
          'X-TESKEL-Event': job.payload.type,
          'X-TESKEL-Delivery': job.payload.id,
        },
        body: JSON.stringify(job.payload),
        signal: AbortSignal.timeout(10000), // 10s timeout
      });
      
      if (response.ok) {
        await db.update(webhookDeliveries)
          .set({ 
            status: 'delivered', 
            responseStatusCode: response.status,
            deliveredAt: new Date(),
          })
          .where(eq(webhookDeliveries.endpointId, job.endpointId));
      } else {
        throw new Error(`HTTP ${response.status}`);
      }
    } catch (error) {
      const attempts = job.attempts + 1;
      if (attempts >= job.maxAttempts) {
        await db.update(webhookDeliveries)
          .set({ status: 'failed', attempts, responseStatusCode: 0 })
          .where(eq(webhookDeliveries.endpointId, job.endpointId));
      } else {
        // Retry with exponential backoff: 1s, 2s, 4s, 8s, 16s
        const delay = Math.pow(2, attempts) * 1000;
        await deliveryQueue.add('webhook', { ...job, attempts }, { delay });
      }
    }
  }
}
```

---

## 2. FRONTEND — Complete Component Tree

### 2.1 Landing Page Components (Detailed)

```tsx
// components/marketing/hero.tsx
// Props: none (hardcoded for now, CMS later)
// Layout: Full-width, min-h-[70vh], centered content
// Content:
//   - Badge: <Badge variant="outline">✨ The OS for digital commerce</Badge>
//   - H1: text-5xl font-bold tracking-tight
//   - Subtitle: text-xl text-muted-foreground
//   - CTAs: <Button size="lg"> + <Button variant="outline" size="lg">
//   - Social proof: text-sm text-muted-foreground with animated counter
// Animation: Fade-in on mount (framer-motion, 150ms)
// Responsive: Stack vertically on mobile

// components/marketing/features-grid.tsx
// Props: features: { icon: LucideIcon, title: string, description: string }[]
// Layout: CSS Grid, grid-cols-1 md:grid-cols-2 lg:grid-cols-3
// Each card: <Card> with icon + title + description
// Hover: subtle shadow elevation
// Animation: Stagger fade-in

// components/marketing/pricing-table.tsx
// Props: plans: { name, price, features, cta, highlighted }[]
// Layout: Grid, 4 columns on desktop, 2 on tablet, 1 on mobile
// Highlighted plan: ring-2 ring-primary, "Most Popular" badge
// Each plan: <Card> with plan name, price, feature checklist, CTA button
// Toggle: Monthly / Yearly (with savings badge)
// Animation: Price number transition on toggle

// components/marketing/testimonials.tsx
// Props: testimonials: { name, role, avatar, quote }[]
// Layout: 3 cards on desktop, carousel on mobile (embla-carousel)
// Each card: Avatar + name + role + quote
// Auto-rotate every 5s on mobile

// components/marketing/faq.tsx
// Props: items: { question, answer }[]
// Component: <Accordion type="single" collapsible>
// Each item: <AccordionItem> with question as trigger
// Answer: rendered markdown
// Animation: Smooth expand/collapse
```

### 2.2 Dashboard Components (Detailed)

```tsx
// components/dashboard/metric-card.tsx
// Props: { title, value, change, changeLabel, icon, trend }
// Layout: <Card> with flex row
// Left: icon in colored circle
// Right: title (text-sm text-muted-foreground), value (text-2xl font-bold), change badge
// Change badge: green if positive, red if negative
// Tooltip: "Compared to previous period"
// Loading: <Skeleton>

// components/dashboard/revenue-chart.tsx
// Props: { data: { date: string, revenue: number, orders: number }[], period: string }
// Component: <AreaChart> from Recharts
// Two areas: Revenue (primary color, filled), Orders (secondary color, line only)
// Tooltip: Date + Revenue + Orders
// X-axis: Date (formatted based on period)
// Y-axis: Revenue ($)
// Period selector: <ToggleGroup> (7d, 30d, 90d, 1y)
// Loading: <Skeleton className="h-[300px]">

// components/dashboard/funnel-editor.tsx
// Props: { funnel: Funnel, steps: FunnelStep[], onSave: () => void }
// Component: React Flow canvas
// Node types (custom):
//   - product_page: Rectangle with product name + image
//   - checkout: Rectangle with "Checkout" label
//   - order_bump: Small rectangle with "Order Bump" + price
//   - upsell: Rectangle with "Upsell" + product name
//   - downsell: Rectangle with "Downsell" + product name
//   - thank_you: Rectangle with "Thank You"
//   - email: Rectangle with envelope icon + subject
// Edge types:
//   - default: Solid arrow
//   - conditional: Dashed arrow with condition label
// Sidebar (right panel):
//   - Step config form (dynamic based on step type)
//   - Product selector
//   - Price selector
//   - Condition editor
//   - Delete step button
// Palette (left panel):
//   - Drag-and-drop step types
// Toolbar (top):
//   - Save, Undo, Redo, Zoom controls, Auto-layout
// State: Local state (React Flow), save to API on Save click

// components/dashboard/file-uploader.tsx
// Props: { productId: string, onUploadComplete: () => void }
// Component: react-dropzone + progress bars
// Flow:
//   1. User drops files → validate type + size
//   2. Request presigned URL from API (POST /v1/orgs/:slug/products/:id/files/presign)
//   3. Upload directly to R2 via presigned URL (axios PUT)
//   4. Show progress bar per file
//   5. On complete: notify parent to refresh file list
// Max file size: 500MB
// Allowed types: .zip, .pdf, .fig, .sketch, .notion, .md, .txt, .json, .png, .jpg, .mp4, .mov
// Multi-file: Yes
// Drag state: Border dashed, bg-primary/5

// components/dashboard/pricing-editor.tsx
// Props: { prices: ProductPrice[], onChange: (prices) => void }
// Layout: Vertical stack of price cards + [Add Price] button
// Each price card:
//   - Drag handle (reorder)
//   - Name input
//   - Amount input ($)
//   - Currency dropdown
//   - Interval toggle (One-time / Monthly / Yearly) — only if product.pricingModel supports
//   - Trial period input (if subscription)
//   - Is Default radio
//   - Features list (add/remove items)
//   - Badge input (optional: "Most Popular")
//   - CTA text input
//   - Delete button
// Live preview: Right side shows rendered pricing table
// Validation: At least 1 price, at least 1 isDefault, amount > 0 (except free)

// components/dashboard/theme-customizer.tsx
// Props: { theme: OrgTheme, onChange: (theme) => void }
// Layout: Two-panel — controls (left) + preview (right)
// Controls:
//   - Primary Color: <Input type="color"> + preset swatches
//   - Font Family: <Select> (Inter, Plus Jakarta Sans, Space Grotesk, DM Sans, Outfit)
//   - Border Radius: <Slider> (0-24px)
//   - Mode: <ToggleGroup> (Light, Dark, System)
//   - Cover Image: Upload
//   - Custom CSS: <Textarea> (monospace, 10 rows)
// Preview: Mini storefront preview that updates in real-time
// Actions: [Save] [Reset to Default]
```

### 2.3 Storefront Components (Detailed)

```tsx
// components/store/product-card.tsx (PUBLIC)
// Props: { product: { id, name, featuredImageUrl, price, rating, salesCount, type } }
// Layout: <Card> with image (aspect-video), content area
// Image: next/image with hover scale effect (transform: scale(1.02), transition 150ms)
// Content:
//   - Product name (font-semibold, line-clamp-1)
//   - Price: "From $X" if multiple prices, "$X" if single
//   - Rating: ★★★★★ (4.7) — small, muted
//   - Sales: "847 sold" — small, muted
//   - Type badge: <Badge variant="secondary"> — small, top-right of image
// Hover: Buy button appears (slide up from bottom of image)
// Click: Navigate to product detail page
// Responsive: Full width on mobile

// components/store/buy-button.tsx (PUBLIC)
// Props: { productId, priceId, amount, currency }
// Flow:
//   1. Click → set loading state
//   2. Call POST /v1/checkout/sessions
//   3. Redirect to session.url (Stripe Checkout)
//   4. On error: show toast with error message
// States: default, loading (spinner), disabled (if product unavailable)
// Variant: primary (default), outline (for secondary price)
// Size: default, lg (on product detail page)

// components/store/email-capture.tsx (PUBLIC)
// Props: { orgId: string }
// Type: Popup (after 10s) or inline (bottom of store)
// Popup:
//   - Trigger: 10s after page load, only if not already subscribed
//   - Content: "Get notified when I launch new products"
//   - Input: email + [Subscribe] button
//   - Success: "You're in! 🎉" + auto-close after 2s
//   - Dismiss: × button, remember dismissal in localStorage (7 days)
// Inline:
//   - Same content, always visible at bottom
// API: POST /v1/orgs/:slug/emails/subscribers (source: 'popup' or 'inline')

// components/checkout/payment-form.tsx
// Props: { sessionId: string, amount: number, currency: string }
// Component: Stripe Elements (CardElement)
// Flow:
//   1. Mount Stripe Elements
//   2. Show Apple Pay / Google Pay if available
//   3. On submit: confirm payment via stripe.confirmPayment()
//   4. On success: redirect to success page
//   5. On error: show inline error message
// Security: Stripe handles all card data (PCI compliant)
```

---

## 3. AUTH PERMISSIONS — Complete Matrix

### 3.1 Route-Level Permissions

```
┌──────────────────────────────────────────────────┬──────────────────────────────┐
│ Route                                            │ Required Role / Auth        │
├──────────────────────────────────────────────────┼──────────────────────────────┤
│ / (landing)                                      │ None                        │
│ /pricing                                         │ None                        │
│ /docs/*                                          │ None                        │
│ /blog/*                                          │ None                        │
│ /login, /signup                                  │ None (redirect if logged in) │
│ /onboarding/*                                    │ Authenticated (new user)    │
│ /dashboard/*                                     │ org member (any role)        │
│ /dashboard/settings/billing                      │ owner                       │
│ /dashboard/settings/team                         │ admin+                      │
│ /dashboard/settings/payouts                      │ admin+                      │
│ /dashboard/products/new                          │ member+                     │
│ /dashboard/products/[id]/settings (delete)        │ admin+                      │
│ /dashboard/orders/[id]/refund                    │ admin+                      │
│ /dashboard/affiliates/*                          │ admin+                      │
│ /dashboard/emails/*                              │ admin+                      │
│ /dashboard/settings/webhooks                     │ member+                     │
│ /dashboard/settings/api-keys                     │ admin+                      │
│ /[storeSlug]/*                                   │ None (public)               │
│ /explore/*                                       │ None (public)               │
│ /checkout/*                                      │ None (buyer-facing)         │
│ /purchases/*                                     │ Authenticated (buyer)       │
│ /subscriptions/*                                 │ Authenticated (buyer)       │
│ /licenses/*                                      │ Authenticated (buyer)       │
└──────────────────────────────────────────────────┴──────────────────────────────┘
```

### 3.2 API-Level Permissions

```
┌──────────────────────────────┬─────────────┬──────────────────────────────────────┐
│ API Endpoint Group           │ Min Role    │ Scope Required (API Key)            │
├──────────────────────────────┼─────────────┼──────────────────────────────────────┤
│ GET /v1/auth/me             │ user        │ -                                    │
│ GET /v1/orgs/:slug/products │ viewer      │ products:read                        │
│ POST /v1/orgs/:slug/products│ member      │ products:write                       │
│ PATCH /v1/orgs/:slug/products│ member     │ products:write                       │
│ DELETE /v1/orgs/:slug/products│ admin     │ products:write                       │
│ GET /v1/orgs/:slug/orders   │ viewer      │ orders:read                          │
│ POST /v1/orgs/:slug/orders/:id/refund │ admin │ orders:write                  │
│ GET /v1/orgs/:slug/customers│ viewer      │ customers:read                       │
│ GET /v1/orgs/:slug/analytics│ viewer      │ analytics:read                        │
│ POST /v1/orgs/:slug/licenses│ admin       │ licenses:write                       │
│ GET /v1/licenses/:key/validate │ none    │ - (public)                           │
│ POST /v1/licenses/:key/activate │ none    │ - (public)                           │
│ POST /v1/checkout/sessions  │ none        │ - (public)                           │
│ POST /v1/access/check       │ none        │ - (public)                           │
│ GET /v1/marketplace/*       │ none        │ - (public)                           │
│ GET /v1/orgs/:slug/webhooks │ admin       │ webhooks:read                        │
│ POST /v1/orgs/:slug/webhooks│ admin       │ webhooks:write                       │
│ GET /v1/orgs/:slug/api-keys │ admin       │ - (not accessible via API key)       │
│ GET /v1/me/*                │ buyer       │ - (session only)                     │
└──────────────────────────────┴─────────────┴──────────────────────────────────────┘
```

---

## 4. DATABASE OPTIMIZATION

### 4.1 Index Strategy

```
Already defined in 02_DATABASE_SCHEMA.md, but here's the reasoning:

High-Frequency Queries:
  - License validation: license_keys.key (unique index) → <1ms
  - Access check: access_grants.customer_id + product_id + status → <5ms
  - Order list: orders.org_id + created_at → <10ms
  - Product list: products.org_id + status → <5ms
  - Marketplace: marketplace_listings.category + status → <10ms

Composite Indexes (add if slow queries detected):
  - orders(org_id, status, created_at DESC) — dashboard order list
  - analytics_events(org_id, event_type, created_at DESC) — analytics queries
  - customers(org_id, lifetime_spend_cents DESC) — top customers
  - products(org_id, total_revenue_cents DESC) — top products
```

### 4.2 Partitioning

```sql
-- Partition analytics_events by month (high-volume table)
-- Use pg_partman extension

-- Enable pg_partman
CREATE EXTENSION IF NOT EXISTS pg_partman;

-- Create parent table
CREATE TABLE analytics_events_partitioned (
  LIKE analytics_events INCLUDING ALL
) PARTITION BY RANGE (created_at);

-- Setup pg_partman to auto-create monthly partitions
SELECT partman.create_parent(
  p_parent_table := 'public.analytics_events_partitioned',
  p_control := 'created_at',
  p_type := 'range',
  p_interval := '1 month',
  p_premake := 3  -- pre-create 3 future partitions
);

-- Drop old partitions after 90 days
-- pg_partman handles this with p_retention config
```

### 4.3 Connection Pooling

```
Neon provides built-in connection pooling:
  - Pooler mode: Transaction (PgBouncer)
  - Max connections: 20 (free), 100 (pro)
  - Connection string: pooler connection URL (different from direct)

Application config:
  - API: Use pooler URL for all queries
  - Migrations: Use direct URL (needs real connection)
  - Analytics: Use read replica connection
```

---

## 5. EDGE CASES & ERROR HANDLING

### 5.1 Payment Edge Cases

```
1. Double payment (same checkout session):
   - Idempotency: Check if order already exists for this stripe_session_id
   - If exists, return existing order instead of creating new

2. Payment succeeds but webhook fails:
   - Stripe retries webhook for up to 3 days
   - Fallback: Cron job checks Stripe for completed sessions without orders
   - Run every 5 minutes

3. Partial refund:
   - Track refunded_cents separately from total_cents
   - Partially refunded orders keep status 'partially_refunded'
   - Access grants: revoke only for refunded items

4. Dispute (chargeback):
   - Set order status to 'disputed'
   - Suspend access grants (but don't revoke yet)
   - Notify creator immediately
   - If dispute resolved in creator's favor: restore access
   - If resolved in buyer's favor: revoke access + refund

5. Currency conversion:
   - All internal amounts stored in cents + currency code
   - Stripe handles conversion for international payments
   - Creator receives payout in their Stripe account currency
   - FX rates: Stripe's rate at time of payment

6. Free products (price = 0):
   - Skip Stripe entirely
   - Create order with totalCents = 0, status = 'paid'
   - Create access grant + delivery immediately
   - No payout scheduled
```

### 5.2 License Edge Cases

```
1. License key collision:
   - Generate with crypto.randomBytes(12).toString('hex').toUpperCase()
   - Remove ambiguous chars: 0,O,1,I,L
   - Check uniqueness before insert (extremely unlikely collision)
   - If collision: regenerate

2. Max activations reached:
   - Return clear error: "Max activations reached (3/3)"
   - Suggest: deactivate an existing device or contact creator
   - Creator can increase max_activations manually

3. License sharing detection:
   - If >5 unique IPs activate same key in 1 hour → suspend
   - Notify creator with details
   - Creator can unsuspend or revoke

4. License expiry:
   - Cron job checks for expired licenses every hour
   - Set status to 'expired'
   - Revoke associated access grants
   - Fire webhook: license.expired
   - Invalidate Redis cache
```

### 5.3 File Delivery Edge Cases

```
1. Download link expired:
   - Generate new signed URL on access
   - Show "Link expired, generating new link..." → auto-refresh

2. File updated after purchase:
   - Buyer always gets latest version
   - Version tracked in product_files table
   - Changelog shown to buyer

3. Very large files (>500MB):
   - Use multipart upload to R2
   - Download via signed URL (direct from R2, not through API)
   - Show download progress in UI

4. R2 outage during upload:
   - Queue upload in Redis
   - Retry with exponential backoff
   - Show "Upload pending" status in dashboard
```

---

## 6. TESTING CHECKLIST — Before Launch

### 6.1 Critical Path Tests

```
□ Signup → email verification → login
□ Create org → onboarding → Stripe Connect
□ Create product (each type) → publish → verify storefront
□ Upload file → download via signed URL
□ Checkout → payment → order created → delivery → download
□ License key generated → validate → activate → deactivate
□ Discount code → apply in checkout → verify discount
□ Subscription → renew → cancel → access revoked
□ Refund → access revoked → license suspended
□ Affiliate → referral → commission calculated
□ Bundle → purchase → all items delivered → split payout
□ Funnel → checkout → order bump → upsell → success
□ Email receipt received
□ Webhook fired and delivered
□ Custom domain → DNS → SSL → store loads
□ Marketplace → search → browse → buy
□ Buyer portal → purchases → downloads → licenses
□ API key → authenticate → scoped access → rate limited
□ Mobile responsive → all pages
□ Dark mode → all pages
□ 404 → error boundary → recovery
```

---

*This document covers: complete service logic (checkout, license, payout, access, email, webhook), component tree with props/flow/validation, auth permission matrices (route + API), database optimization (indexes, partitioning, pooling), edge cases for payments/licenses/files, and pre-launch test checklist.*
