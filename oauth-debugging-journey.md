## Debugging Journey: Basic Auth Failure → OAuth 2.0 Success

**Goal:** Test a Scripted REST API (inbound integration) endpoint using Postman.

**Problem:** Basic Auth consistently failed with 401 Unauthorized, even with 
correct credentials, across multiple accounts (admin and a freshly created 
integration_user) and multiple passwords.

**Debugging steps taken:**
1. Verified the REST endpoint and Default ACL configuration were correct.
2. Ruled out account lockout (checked "Locked out" field — unchecked).
3. Ruled out network/proxy interference using curl -v — confirmed the 
   Authorization header WAS reaching the server correctly.
4. Server response showed "Basic authentication problem, ignoring" — a 
   genuine credential rejection at the platform level, not a transport issue.
5. Tested with a brand-new user (integration_user) with a freshly generated 
   password — same failure. This ruled out a single-account issue and pointed 
   to a platform-wide block (likely MFA enforcement on this Zurich-release PDI).

**Resolution:** Pivoted to OAuth 2.0 (Password Credentials grant):
1. Created an OAuth Application Registry in ServiceNow → got Client ID + Secret.
2. Configured Postman: Auth Type = OAuth 2.0, Grant Type = Password Credentials, 
   Access Token URL = https://<instance>.service-now.com/oauth_token.do
3. Generated an access token successfully, used it as a Bearer token.
4. Request succeeded — 200 OK, correct JSON response from the Scripted REST API.

**Key takeaway:** Basic Auth failing consistently across multiple valid 
accounts is a strong signal of a platform-level block (e.g., MFA), not a 
credentials problem. OAuth 2.0 is also the more production-realistic 
approach anyway.
