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
11. `18_LANDING_AND_MARKETING_SITE.md`
12. `19_BRAND_COPY_AND_VOICE.md`
13. `20_EMAIL_TEMPLATES.md`
14. `21_ERROR_AND_STATE_CATALOG.md`
15. `22_GLOSSARY_AND_DATA_DICTIONARY.md`
16. `23_FINAL_BUILD_CHECKLIST.md`
17. `01_PRODUCT_OVERVIEW.md`
18. `02_DATABASE_SCHEMA.md`
19. `03_API_SPEC.md`
20. `04_FRONTEND_UIUX_SPEC.md`
21. `05_BUILD_DEPLOY_PLAN.md`
22. `06_INFRA_SECURITY_DEVOPS.md`
23. `07_ENGINEERING_DETAILS.md`
24. `09_TOP_GLOBAL_FEATURES.md`
25. `10_UIUX_MODERN_CLEAN.md`

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
| Surface & Content | `18_LANDING_AND_MARKETING_SITE.md`, `19_BRAND_COPY_AND_VOICE.md`, `20_EMAIL_TEMPLATES.md`, `21_ERROR_AND_STATE_CATALOG.md` | What every public surface says |
| Reference | `22_GLOSSARY_AND_DATA_DICTIONARY.md`, `23_FINAL_BUILD_CHECKLIST.md` | Vocabulary, enums, IDs, master build checklist |
| Product Specs | `01_PRODUCT_OVERVIEW.md`, `02_DATABASE_SCHEMA.md`, `03_API_SPEC.md`, `04_FRONTEND_UIUX_SPEC.md`, `07_ENGINEERING_DETAILS.md` | What we are building |
| Quality & GTM | `05_BUILD_DEPLOY_PLAN.md`, `06_INFRA_SECURITY_DEVOPS.md`, `09_TOP_GLOBAL_FEATURES.md`, `10_UIUX_MODERN_CLEAN.md`, `17_LAUNCH_AND_GROWTH.md` | How we ship and how we win the market |

## Single Source of "What's Left"

`23_FINAL_BUILD_CHECKLIST.md` is the master checklist for every file, endpoint, page, email, infra item, and launch task. `BUILD_STATUS.md` records current progress at a glance. If the checklist is fully green, TESKEL is ready to launch.
