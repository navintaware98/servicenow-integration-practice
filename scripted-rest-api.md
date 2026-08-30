## Scripted REST API — Custom POST and GET (beyond the Table API)

**Why build custom endpoints instead of just using the Table API?**
The Table API gives raw, generic access to every field on a table — fine 
for internal tools, but risky and clumsy to hand to an external system. A 
Scripted REST API lets me define a purpose-built contract: exactly what the 
caller must send, what's optional (with sensible defaults), what gets 
validated before touching the database, and exactly what shape the 
response takes. It's the difference between exposing my whole database and 
exposing a designed interface.

### POST /create-incident
- Reads the caller's JSON body via `request.body.data`.
- Validates `short_description` is present — rejects early with 400 if not, 
  rather than creating a broken record.
- Applies defaults for optional fields (`urgency`, `impact` default to '3' 
  using the `||` pattern) — the caller doesn't need to know ServiceNow's 
  internal conventions.
- Creates the record with `GlideRecord.initialize()` + field assignment + 
  `.insert()` — insert() is the actual write to the database; everything 
  before it is just building the record in memory.
- Returns 201 with a clean, minimal response (sys_id + number), not the 
  full raw record.

### GET /incident-status?number=...
- First custom endpoint using a query parameter (`request.queryParams.x`), 
  not a request body — appropriate for GET, which conventionally has no body.
- Three distinct response paths depending on input:
  - No `number` param → 400 (bad request — caller's fault, missing input)
  - `number` param given but no match found → 404 (not found — valid 
    request, but nothing there)
  - Match found → 200 with a clean subset of fields, using 
    `.getDisplayValue()` for fields like `state`/`urgency` so the caller 
    gets human-readable labels ("New", "High") instead of raw internal 
    codes (1, 2).
- This distinction between 400 and 404 is worth remembering precisely — 
  they're both "it didn't work" but for very different reasons, and mixing 
  them up is a common mistake.

**Bigger picture takeaway:** across today's work (Table API CRUD, custom 
POST with validation, custom GET with query params), the throughline is 
that REST design is about intentional contracts — the URL says what 
resource you're touching, the method says what action, and a well-built 
API adds validation and shaping on top so the interface is safe and clear 
for whoever's calling it.
