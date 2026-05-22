# TESKEL

TESKEL is a planned global-grade digital commerce OS for creators and teams selling digital products, licenses, subscriptions, services, bundles, communities, and API-powered storefronts.

This repository currently contains the build specification pack. Implementation must start from the documentation in this order:

1. `TESKEL_BUILD_INDEX.md`
2. `00_README_BUILD_ORDER.md`
3. `08_AUTONOMOUS_BUILD_PLAYBOOK.md`
4. `11_BUILD_READINESS_AUDIT.md`
5. `01_PRODUCT_OVERVIEW.md`
6. `02_DATABASE_SCHEMA.md`
7. `03_API_SPEC.md`
8. `04_FRONTEND_UIUX_SPEC.md`
9. `05_BUILD_DEPLOY_PLAN.md`
10. `06_INFRA_SECURITY_DEVOPS.md`
11. `07_ENGINEERING_DETAILS.md`
12. `09_TOP_GLOBAL_FEATURES.md`
13. `10_UIUX_MODERN_CLEAN.md`

Build principle:

```text
Auth → Org → Download Product → File Upload → Storefront → Checkout → Order → Delivery → Dashboard → Product Types → Growth Tools → Platform API → Production Hardening → Launch
```

Do not build every advanced feature at once. The first shippable milestone is a creator creating a downloadable product and a buyer paying for and receiving it end-to-end.

