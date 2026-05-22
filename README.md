# TESKEL

TESKEL is a planned global-grade digital commerce OS for creators and teams selling digital products, licenses, subscriptions, services, bundles, communities, and API-powered storefronts.

This repository currently contains the build specification pack. Implementation must start from the documentation in this order:

1. `TESKEL_BUILD_INDEX.md`
2. `00_README_BUILD_ORDER.md`
3. `08_AUTONOMOUS_BUILD_PLAYBOOK.md`
4. `11_BUILD_READINESS_AUDIT.md`
5. `12_ENGINEERING_EXCELLENCE.md`
6. `13_TESTING_STRATEGY.md`
7. `14_OBSERVABILITY_AND_INCIDENTS.md`
8. `15_SECURITY_AND_COMPLIANCE.md`
9. `16_RELEASE_AND_PERFORMANCE.md`
10. `17_LAUNCH_AND_GROWTH.md`
11. `01_PRODUCT_OVERVIEW.md`
12. `02_DATABASE_SCHEMA.md`
13. `03_API_SPEC.md`
14. `04_FRONTEND_UIUX_SPEC.md`
15. `05_BUILD_DEPLOY_PLAN.md`
16. `06_INFRA_SECURITY_DEVOPS.md`
17. `07_ENGINEERING_DETAILS.md`
18. `09_TOP_GLOBAL_FEATURES.md`
19. `10_UIUX_MODERN_CLEAN.md`

Build principle:

```text
Auth → Org → Download Product → File Upload → Storefront → Checkout → Order → Delivery → Dashboard → Product Types → Growth Tools → Platform API → Production Hardening → Launch
```

Do not build every advanced feature at once. The first shippable milestone is a creator creating a downloadable product and a buyer paying for and receiving it end-to-end.

## Documentation Layers

The plan is grouped into seven layers so a single document never tries to own everything:

| Layer | Files | Purpose |
|-------|-------|---------|
| Entry | `README.md`, `TESKEL_BUILD_INDEX.md`, `BUILD_STATUS.md` | Where to start; what is the current state |
| Build Discipline | `00_README_BUILD_ORDER.md`, `08_AUTONOMOUS_BUILD_PLAYBOOK.md`, `11_BUILD_READINESS_AUDIT.md` | How to build and stay safe |
| Engineering Standards | `12_ENGINEERING_EXCELLENCE.md`, `13_TESTING_STRATEGY.md` | Senior-grade day-to-day practices and quality bar |
| Operations | `14_OBSERVABILITY_AND_INCIDENTS.md`, `16_RELEASE_AND_PERFORMANCE.md` | Telemetry, on-call, release, performance |
| Trust | `15_SECURITY_AND_COMPLIANCE.md` | Security, privacy, regulatory readiness |
| Product Specs | `01_PRODUCT_OVERVIEW.md`, `02_DATABASE_SCHEMA.md`, `03_API_SPEC.md`, `04_FRONTEND_UIUX_SPEC.md`, `07_ENGINEERING_DETAILS.md` | What we are building |
| Quality & GTM | `05_BUILD_DEPLOY_PLAN.md`, `06_INFRA_SECURITY_DEVOPS.md`, `09_TOP_GLOBAL_FEATURES.md`, `10_UIUX_MODERN_CLEAN.md`, `17_LAUNCH_AND_GROWTH.md` | How we ship and how we win the market |
