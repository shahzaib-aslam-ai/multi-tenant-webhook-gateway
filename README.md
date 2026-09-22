# Multi-Tenant Webhook Gateway

Multi-tenant n8n webhook gateway routing 3 clients to separate validation rules and Google Sheets destinations, with dynamic 401/400/200 responses.

## Workflow

<img width="1740" height="882" alt="n8n workflow screenshot" src="https://github.com/user-attachments/assets/de68b004-4638-4129-a6be-fe10b51d17d2" />

## Overview

One n8n webhook endpoint serves 3 tenants, identified by an `x-tenant-key` header. Tenant config and validation logic live in a single Python dictionary — adding a new tenant requires only a new entry, no code changes.

## Tenants

| Tenant | Required Fields | Destination |
|---|---|---|
| RetailCo | email, order_id, amount | Sheet A |
| SurveyCo | respondent_id, rating | Sheet B |
| BillingCo | invoice_id, amount, due_date | Sheet C |

## Routing Logic

1. Extract tenant key from `x-tenant-key` header.
2. Look up tenant in config dict — invalid/missing key → 401.
3. Validate tenant-specific required fields — missing field → 400 with field name.
4. Valid request → route to tenant's destination sheet, respond 200.

## Response Shapes

**401 Unauthorized**
```json
{ "status": "error", "message": "Invalid or missing tenant API key", "data": {} }
```

**400 Bad Request**
```json
{ "status": "error", "message": "Missing required field for RetailCo: order_id", "data": {} }
```

**200 Success**
```json
{ "status": "success", "message": "Data routed to RetailCo", "data": {} }
```

## Adding a New Tenant

Add one entry to the `tenants` dictionary — no other code changes required.
