---
name: decisionlink-share-value-proposition-with-buyer
description: Share an Xfactor.io value proposition with the buyer and work it collaboratively — create the share and collaboration from the seller side, then read and update discovery, benefits and factors through the buyer-facing Collaboration Manager.
api: Xfactor.io Value Proposition API + Collaboration Manager API
base_url: https://api.xfactor.io
generated: '2026-08-13'
method: generated
source: openapi/_original/decisionlink-value-proposition-openapi.json, openapi/_original/decisionlink-collaboration-openapi.json
operations:
  - create_value_prop_share_v1_value_proposition__valuePropId__shares_post
  - get_value_prop_shares_v1_value_proposition__valuePropId__shares_get
  - update_value_prop_share_v1_value_proposition__valuePropId__shares__shareId__patch
  - delete_value_prop_share_v1_value_proposition__valuePropId__shares__shareId__delete
  - create_collaboration_share_v1_value_proposition__valuePropId__collaborations_post
  - get_collaborations_v1_value_proposition__valuePropId__collaborations_get
  - update_customer_collaboration_v1_value_proposition__valuePropId__collaborations__collaborationId__put
  - delete_customer_collaboration_v1_value_proposition__valuePropId__collaborations__collaborationId__delete
  - collaboration_login_v1_collaboration_login_post
  - create_collaboration_v1_collaboration__post
  - get_discovery_questions_v1_collaboration_valueProps_discovery_get
  - create_discovery_questions_v1_collaboration_valueProps_discovery_patch
  - get_benefits_v1_collaboration_valueProps_benefits_get
  - update_benefit_active_status_v1_collaboration_valueProps_benefits_active_patch
  - get_factors_v1_collaboration_valueProps_factors_get
  - update_factors_v1_collaboration_valueProps_factors_patch
  - get_asset_v1_collaboration_valueProps_asset_get
---

# Share a value proposition with the buyer

Two services are involved and they are **not** the same API. The seller side lives in the Value
Proposition service; the buyer side is the separate Collaboration Manager, which has its own login.

> **Access.** Seller-side calls take an Auth0 bearer JWT. The Collaboration Manager exposes its own
> `POST /v1/collaboration/login` and then expects `HTTPBearer` on the remaining operations. See
> `authentication/decisionlink-authentication.yml`.

## Seller side — Value Proposition service

1. **Create the share.**
   `POST /v1/value-proposition/{valuePropId}/shares`
   (`create_value_prop_share_v1_value_proposition__valuePropId__shares_post`) with a `CreateShare`.
   List existing shares first with
   `GET /v1/value-proposition/{valuePropId}/shares`
   (`get_value_prop_shares_v1_value_proposition__valuePropId__shares_get`) so you do not create a
   duplicate — there is no idempotency key to protect you.

2. **Create the collaboration.**
   `POST /v1/value-proposition/{valuePropId}/collaborations`
   (`create_collaboration_share_v1_value_proposition__valuePropId__collaborations_post`) with a
   `CreateCollaboration`. Amend with
   `PUT .../collaborations/{collaborationId}`
   (`update_customer_collaboration_v1_value_proposition__valuePropId__collaborations__collaborationId__put`).

3. **Revoke when the deal moves on.**
   `DELETE .../shares/{shareId}`
   (`delete_value_prop_share_v1_value_proposition__valuePropId__shares__shareId__delete`) and
   `DELETE .../collaborations/{collaborationId}`
   (`delete_customer_collaboration_v1_value_proposition__valuePropId__collaborations__collaborationId__delete`).
   These are destructive and unversioned — there is no soft-delete or restore operation in the spec.

## Buyer side — Collaboration Manager service

4. **Authenticate into the collaboration.**
   `POST /v1/collaboration/login` (`collaboration_login_v1_collaboration_login_post`) with a
   `Login`. `POST /v1/collaboration/` (`create_collaboration_v1_collaboration__post`) creates the
   collaboration record on this side.

5. **Work the value proposition together.**
   - Discovery: `GET /v1/collaboration/valueProps/discovery`
     (`get_discovery_questions_v1_collaboration_valueProps_discovery_get`) and
     `PATCH` the same path (`create_discovery_questions_v1_collaboration_valueProps_discovery_patch`).
   - Benefits: `GET /v1/collaboration/valueProps/benefits`
     (`get_benefits_v1_collaboration_valueProps_benefits_get`) and
     `PATCH /v1/collaboration/valueProps/benefits/active`
     (`update_benefit_active_status_v1_collaboration_valueProps_benefits_active_patch`).
   - Factors: `GET /v1/collaboration/valueProps/factors`
     (`get_factors_v1_collaboration_valueProps_factors_get`) and
     `PATCH` the same path (`update_factors_v1_collaboration_valueProps_factors_patch`).
   - Asset: `GET /v1/collaboration/valueProps/asset`
     (`get_asset_v1_collaboration_valueProps_asset_get`).

## Cautions

- **The buyer can change the model.** `PATCH .../benefits/active` and `PATCH .../factors` exist on
  the buyer-facing service, so a collaboration is a two-way edit surface, not a read-only share.
  Re-read the seller-side value proposition after a collaboration session rather than assuming your
  copy is current.
- **The two services have no shared identifier scheme published.** The Collaboration Manager
  operations take no `valuePropId` path parameter at all — scope comes from the authenticated
  session. Do not assume you can address an arbitrary value proposition from this side.
- **The Collaboration Manager is mounted at `/api/v1/collaboration/`** for its OpenAPI document and
  Swagger UI, while its operations are declared under `/v1/collaboration/`.
