---
description: Set up a live RTMP-to-HLS streaming stack on OSC. Use this skill when the user wants to stream from OBS or any RTMP encoder and serve HLS to viewers.
---

# Live Streaming Stack (RTMP to HLS)

This skill deploys the `eyevinn-live-encoding` service on Open Source Cloud to receive an RTMP stream and serve it as HLS.

## Before you begin

**TLS certificate delay.** A new OSC instance reports ready before its TLS certificate is issued. If the first
request to the RTMP or HLS URL returns a connection or certificate error, wait approximately 60 seconds and retry
the same request unchanged. This is normal for any new instance on the platform.

**Token cost and plan limits.** `eyevinn-live-encoding` costs 50 tokens/day per running instance.

- FREE plan: 100-token one-time grant, no daily refill. A single instance runs for approximately 2 days before
  exhausting the grant.
- PERSONAL plan: refills 40 tokens/day. This does not cover a 50/day instance.
- PROFESSIONAL plan: refills 300/day. This is the least expensive plan for continuous use.

If a tool call fails on plan or token grounds, tell the user which plan covers the cost (PROFESSIONAL for continuous
use) and stop. Do not retry the call.

## What you are setting up

- Service: `eyevinn-live-encoding`
- Input: RTMP from OBS Studio or any RTMP encoder
- Output: HLS adaptive bitrate

## Steps

1. Call `osc_search_tools` to confirm `eyevinn-live-encoding` is available and retrieve its required fields.
   Only `name` is required.
2. Call `osc_call_tool` with `create-service-instance`, providing `serviceId: "eyevinn-live-encoding"` and a
   `name` of the user's choice (lowercase, alphanumeric, hyphens allowed).
3. Poll the instance status using `osc_call_tool` with `describe-service-instance` until the instance reports
   `running`. This typically takes 60-120 seconds.
4. Retrieve the RTMP ingest URL and HLS playback URL from the instance details.
5. Present the RTMP URL to the user for their OBS Stream settings (Server field) and the HLS URL for their player.

## Expected outcome

- RTMP ingest URL: ready to receive a stream from OBS
- HLS playback URL: ready to serve viewers
- Wall-clock setup time: approximately 390 seconds from create to first stream
