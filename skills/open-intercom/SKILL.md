---
description: Set up Open Intercom on OSC for real-time team communication. Use this skill when the user wants a self-hosted open intercom system with WebRTC audio/video.
---

# Open Intercom Setup

This skill deploys the Open Intercom stack on Open Source Cloud. The stack requires three services created in a
specific order and wired together. Skipping the wiring steps or creating them out of order produces crash-looping
pods.

## Before you begin

**TLS certificate delay.** Each new OSC instance reports ready before its TLS certificate is issued. If an
initial request to any instance URL returns a connection or certificate error, wait approximately 60 seconds and
retry the same request unchanged. This applies to all three instances in this stack.

**Token cost and plan limits.** The Open Intercom stack requires three running instances. The combined daily cost
depends on the token rates of each service; use `osc_search_tools` to check current rates before confirming with
the user. As a guide:

- FREE plan: 100-token one-time grant, no daily refill. Likely exhausted within hours for a three-instance stack.
- PERSONAL plan: refills 40 tokens/day. May not cover the combined cost.
- PROFESSIONAL plan: refills 300/day. Recommended for any multi-instance stack.

If a tool call fails on plan or token grounds, tell the user which plan covers the combined cost and stop. Do not
retry the call.

## What you are setting up

Three services, created in this exact order:

1. `eyevinn-docker-wrtc-sfu` -- the WebRTC SFU (selective forwarding unit)
2. A database: `apache-couchdb` (recommended) or `birme-osc-mongodb`
3. `eyevinn-intercom-manager` -- the intercom manager, wired to both of the above

## Steps

1. Call `osc_search_tools` with the query `"wrtc sfu"` to confirm `eyevinn-docker-wrtc-sfu` is available and
   retrieve its required fields.

2. Call `osc_call_tool` with `create-service-instance` to create the SFU:
   - `serviceId`: `"eyevinn-docker-wrtc-sfu"`
   - `name`: a name of the user's choice

3. Call `osc_call_tool` with `describe-service-instance` on the SFU instance and record:
   - The instance URL
   - The `apiKey` (or equivalent SFU API key field)

   Note the SFU URL. The intercom manager connects to the SFU on port 8080. Construct the `smbUrl` as
   `http://<sfu-host>:8080` -- plain HTTP with the explicit port, **no credentials in this URL**. The API key
   goes in a separate field.

4. Call `osc_search_tools` with `"couchdb"` and then call `osc_call_tool` with `create-service-instance` to
   create the database:
   - `serviceId`: `"apache-couchdb"` (preferred) or `"birme-osc-mongodb"`
   - `name`: a name of the user's choice

5. Call `osc_call_tool` with `describe-service-instance` on the database instance and record the connection
   string. **Critical:** if the connection string ends with a `/DBNAME` placeholder (as CouchDB connection
   strings from `describe-service-instance` do), replace the literal text `DBNAME` with the actual database
   name you will use, for example `intercom-manager`. The intercom manager fails DB init with no error message
   if the database name is missing from the path.

6. Call `osc_search_tools` with `"intercom manager"` to confirm `eyevinn-intercom-manager` is available and
   retrieve its required fields.

7. Call `osc_call_tool` with `create-service-instance` to create the intercom manager:
   - `serviceId`: `"eyevinn-intercom-manager"`
   - `name`: a name of the user's choice
   - `smbUrl`: the SFU URL with port 8080, for example `http://<sfu-host>:8080` -- plain URL only, no API key
   - `smbApiKey`: the SFU API key recorded in step 3
   - `dbUrl`: the full database connection string with the real database name in the path, for example
     `http://admin:password@<db-host>:5984/intercom-manager`

8. Poll `describe-service-instance` on the intercom manager until it reports `running`.

9. Return the intercom manager URL to the user.

## Wiring rules (critical)

- **Creation order is not optional.** The intercom manager fails to start if the SFU or database is not already
  running when it is created.
- **`smbUrl` must not contain credentials.** The SFU API key belongs exclusively in `smbApiKey`. Putting it in
  the URL causes authentication failures.
- **`dbUrl` must include the database name as a path suffix.** For example:
  `http://admin:secret@couchdb-host:5984/intercom-manager`. The segment after the last `/` is the database
  name. Omitting it leaves the intercom manager unable to initialise its schema.
- **Replace `/DBNAME` in CouchDB connection strings.** The `describe-service-instance` response for
  `apache-couchdb` returns a connection string ending in the literal text `/DBNAME` as a placeholder. Substitute
  the real database name before passing the value to `dbUrl`.
