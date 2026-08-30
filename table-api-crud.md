## Table API — Full CRUD + OAuth Scope Fix

**Goal:** Perform Create, Read, Update, Delete operations on the `incident` 
table using ServiceNow's built-in Table API, authenticated via OAuth 2.0.

### HTTP Method → URL Pattern

| Method | URL                                          | Purpose                        |
|--------|----------------------------------------------|---------------------------------|
| GET    | /api/now/table/incident                      | Read (list or filtered)         |
| POST   | /api/now/table/incident                      | Create — no ID, doesn't exist yet |
| PUT    | /api/now/table/incident/{sys_id}              | Update — targets one record     |
| DELETE | /api/now/table/incident/{sys_id}              | Delete — targets one record     |

Key idea: the resource (what you're acting on) lives in the URL; the action 
(what you're doing to it) lives in the HTTP method. This is the core 
principle behind REST design.

### Steps performed
1. POST with a JSON body (short_description, urgency, impact) → 201 Created, 
   returned the new record including its sys_id.
2. PUT to /incident/{sys_id} with updated fields → 200 OK, record updated.
3. DELETE to /incident/{sys_id}, no body → 204 No Content (successful delete 
   returns no body, by REST convention).

### Bug hit along the way: OAuth "Access to unscoped api is not allowed"

**Symptom:** A previously-working OAuth-authenticated GET request to a 
custom Scripted REST API (in Global scope) suddenly returned 403 Forbidden 
with: 
`{"error":{"message":"User Not Authorized","detail":"Access to unscoped 
api is not allowed"}}`

**Root cause:** By default, an OAuth Application Registry's 'Scope 
Restriction' field limits the OAuth client to its own application scope. 
Since the Scripted REST API lives in the Global scope, the OAuth client 
was blocked from reaching it.

**Fix:**
1. Navigate to System OAuth > Application Registry.
2. Open the relevant OAuth application record.
3. Change 'Scope Restriction' to 'Broadly Scoped' (allows cross-scope 
   access — fine for testing; in production, scope this down intentionally 
   instead of broadening it).
4. Get a fresh access token (old tokens were issued under the old 
   restriction and won't reflect the change).

**Why this exists:** it's a security boundary preventing a scoped app's 
OAuth client from silently accessing unrelated parts of the instance. 
'Broadly Scoped' is an explicit override for cases (like ours) where that's 
genuinely needed.