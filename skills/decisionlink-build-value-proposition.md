---
name: decisionlink-build-value-proposition
description: Create a customer value proposition in Xfactor.io (formerly DecisionLink) — validate the buyer's company, create the value prop, answer discovery, activate benefits, and calculate the ROI metrics.
api: Xfactor.io Value Proposition API
base_url: https://api.xfactor.io
generated: '2026-08-13'
method: generated
source: openapi/_original/decisionlink-value-proposition-openapi.json
operations:
  - validate_company_v1_value_proposition_companies_validate_post
  - search_companies_v1_value_proposition_companies_search_post
  - create_company_v1_value_proposition_companies_post
  - create_value_proposition_v1_value_proposition_post
  - get_value_proposition_v1_value_proposition__valuePropId__get
  - get_discovery_questions_v1_value_proposition__valuePropId__discovery_questions_get
  - update_discovery_questions_v1_value_proposition__valuePropId__discovery_questions_post
  - get_benefits_v1_value_proposition__valuePropId__benefits_get
  - update_benefit_active_status_v1_value_proposition__valuePropId__benefits_active_patch
  - calculate_metrics_v1_value_proposition__valuePropId__calculate_metrics_post
  - get_value_proposition_vc_chart_v1_value_proposition__valuePropId__vc_chart_get
---

# Build a value proposition

Xfactor.io's Value Proposition service is the Customer Value Management core DecisionLink built:
you name the buyer, answer discovery about their situation, switch on the benefits that apply, and
the service computes the economic case.

> **Access.** Every operation below requires an `Authorization: Bearer <token>` header carrying an
> Auth0-issued JWT (issuer `https://xf-prd.us.auth0.com/`, audience `https://api.xfactor.io/`), or a
> customer-issued API key. Xfactor.io publishes no self-service signup — credentials come from your
> account team. Without a credential every call returns `401` or `403` with
> `{"detail":"Not authenticated"}`. See `authentication/decisionlink-authentication.yml`.

## Steps

1. **Resolve the buyer's company.**
   `POST /v1/value-proposition/companies/validate` (`validate_company_v1_value_proposition_companies_validate_post`)
   with a `CompanyValidatePayload`. If it is not known, search first with
   `POST /v1/value-proposition/companies/search`
   (`search_companies_v1_value_proposition_companies_search_post`), and create it with
   `POST /v1/value-proposition/companies`
   (`create_company_v1_value_proposition_companies_post`) using a `CreateCompanyPayload`.

2. **Create the value proposition.**
   `POST /v1/value-proposition` (`create_value_proposition_v1_value_proposition_post`) with a
   `CreateValuePropPayload`. Keep the returned `valuePropId` — 67 of this service's operations take
   it as a path parameter.

3. **Answer discovery.**
   Read the question set with
   `GET /v1/value-proposition/{valuePropId}/discovery-questions`
   (`get_discovery_questions_v1_value_proposition__valuePropId__discovery_questions_get`), then post
   the answers with
   `POST /v1/value-proposition/{valuePropId}/discovery-questions`
   (`update_discovery_questions_v1_value_proposition__valuePropId__discovery_questions_post`).
   Discovery answers are what drive which benefits and factors become relevant.

4. **Activate the benefits that apply.**
   `GET /v1/value-proposition/{valuePropId}/benefits`
   (`get_benefits_v1_value_proposition__valuePropId__benefits_get`) returns the candidate set; turn
   them on or off with
   `PATCH /v1/value-proposition/{valuePropId}/benefits-active`
   (`update_benefit_active_status_v1_value_proposition__valuePropId__benefits_active_patch`) using a
   `BenefitActivePayload`. Do not activate a benefit the discovery answers do not support — the
   whole point of the model is that the buyer can defend it.

5. **Calculate the case.**
   `POST /v1/value-proposition/{valuePropId}/calculate-metrics`
   (`calculate_metrics_v1_value_proposition__valuePropId__calculate_metrics_post`), then read the
   chart with
   `GET /v1/value-proposition/{valuePropId}/vc-chart`
   (`get_value_proposition_vc_chart_v1_value_proposition__valuePropId__vc_chart_get`).

## Rules the API does not enforce for you

- **Retries are not safe.** No operation accepts an idempotency key, and there is no de-duplication
  header anywhere in the spec. A retried `POST /v1/value-proposition` creates a second value
  proposition. Confirm with `GET /v1/value-proposition/{valuePropId}` before retrying a create.
  See `conventions/decisionlink-conventions.yml`.
- **List responses are unbounded.** No operation accepts `limit`, `offset`, `page` or a cursor.
  `GET /v1/value-proposition/list` returns everything the token can see; budget for it.
- **Errors carry no codes.** A failure returns `{"detail": ...}` — an array of
  `{loc, msg, type}` on `422`, a string on `409` and `500`. There is no error code to branch on.
  See `errors/decisionlink-problem-types.yml`.
- **There is no rate-limit signal.** No `RateLimit-*` or `Retry-After` header is returned and no
  `429` is declared. Throttle yourself; you will get no warning.
