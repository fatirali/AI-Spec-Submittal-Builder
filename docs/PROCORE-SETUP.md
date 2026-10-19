# Spectre — Procore Setup and Validation Guide

Date: 2026-10-18. Planning guide; no app, credentials or production access have been created by this document.

## 1. Owner setup

1. Open the [Procore Developer Portal](https://developers.procore.com/) and establish the appropriate developer account using your business identity.
2. Create a Spectre application. Keep the product's commercial, reusable SaaS purpose accurate when completing registration.
3. Create an app version and add a Data Connector with **User Level Authentication**. Spectre's MVP should operate within the signed-in user's permissions, as described in the [Data Connector guide](https://developers.procore.com/documentation/content/building-data-connection-apps).
4. Configure the authorized OAuth redirect URI supplied by the eventual Spectre implementation. Separate sandbox and production configuration. Do not invent a live callback URL before a deployment exists.
5. Make sandbox credentials available through the deployment's secret-management process. Do not paste secrets into the PRD, repository, chat, frontend settings, logs or screenshots.
6. Save the version and install it in the Developer Sandbox. Provision representative users/projects there. An existing project in a live company is not automatically a Developer Sandbox project; verify its environment before testing writes.

The application should be registered once for Spectre; customers follow the appropriate install/consent path, rather than each customer writing their own integration. Every Spectre user still authenticates individually so workspace membership does not broaden Procore access.

## 2. Production distribution dependency

Procore currently distinguishes sandbox access from production access. Its [verification guidance](https://developers.procore.com/documentation/content/verification-and-production-access) places reusable commercial SaaS products, including one-customer pilots intended for later distribution, on the Marketplace Partner path. Arrange verification, certification or an applicable approved pilot with Procore. A user's access to a test project alone does not satisfy the app's production requirements.

Record the approved distribution path and environment before enabling a production connection. Do not promise certification dates or assume approval. No Marketplace application has been submitted by this PRD work.

## 3. Engineering validation checklist

- Discover the connected user's accessible companies and projects and bind each workspace to one company. Use the applicable Procore regional endpoints.
- Verify which API mechanisms expose project/tool/granular permissions and private-record visibility. Record unsupported cases; fail closed where access cannot be established.
- Test an extract-only Spectre user, a reviewer with create permission, a reviewer without it, an admin without project access, and a user whose access is revoked during a job.
- Inspect the target project's required/configurable fields and allowed values. Map approved candidate fields to valid Procore values; collect missing inputs from a reviewer. Do not assume every company uses the same field configuration.
- Confirm supported create behavior, required manager/type/title and other fields, numbering, specification-section references, default status/privacy and notifications. Choose supported stable endpoints after reading their exact contracts; do not infer create support from a list endpoint.
- Test duplicate matching, pagination, rate limits, token expiry/refresh, create timeouts, reconciliation and partial successes.
- Check whether the provider supports a suitable idempotency feature. Regardless, persist Spectre intent/receipt records and prevent blind repeat creates after unknown outcomes.
- Do not introduce update/delete methods into the business adapter. Verify no cleanup, rollback or cancellation flow can call them. Creation of packages/workflows/projects/sections is out of scope.
- Publish only explicit reviewer-confirmed payloads into a selected test project. Credentials alone are not permission to choose an arbitrary production destination.

## 4. Deliverables from the integration investigation

Produce a capability matrix containing exact endpoint/API version, fields, required permission, tenant/region behavior, pagination/rate constraints, test environment and result for each needed operation. Include sanitized examples, unresolved limitations and a permission mapping used by server checks. Keep secrets and customer data out of the matrix.

Useful official references: [Data Connector setup](https://developers.procore.com/documentation/content/building-data-connection-apps), [verification and production access](https://developers.procore.com/documentation/content/verification-and-production-access), [submittal field configuration](https://dev.support.procore.com/faq-which-fields-in-the-submittals-tool-can-be-configured-as-required-optional-or-hidden/), and [submittal permissions](https://dev.support.procore.com/product-manuals/submittals-project/permissions).
