---
name: deploy-my-repo
description: Deploy the user's own git repository to Open Source Cloud (OSC) as a private My App and return its live URL. Use this skill when the user says "deploy this repo", "put my app live", "host this on OSC", or has code an agent wrote and wants it running. Not for adding an open-source project to the public OSC catalog.
---

# Deploy My Repo

This skill puts a repository live on Open Source Cloud as a My App: private to the user's account, at a stable URL that survives rebuilds and restarts.

## Calling the tools

Call each tool named below directly. If the server only exposes the dispatcher tools, use `osc_search_tools` to read a tool's fields, `osc_call_tool` for read-only tools (`get-my-plan`, `wait-for-app-ready`, `diagnose-my-app`, `get-my-app-logs`, `list-git-credentials`), and `osc_write_tool` for tools that create or change something (`create-my-app`, `restart-my-app`, `update-my-app-source-ref`, `update-my-app-github-token`). `osc_call_tool` refuses write tools.

## Ask two things first

1. **Which repository and branch?** You need an HTTPS git URL. To deploy a branch, append `#branch` (for example `https://github.com/user/repo#develop`). Without it the default branch deploys.
2. **Is the repository private?** If yes, the app needs a registered git credential. A private repository created without one fails its build with a 403.

**Stop and tell the user** if there is no git remote (local-only code must be pushed first), or if the runtime is not one of: Node.js, Python, WebAssembly, Go, .NET, PHP.

## Check the repo before deploying

- The app must listen on the port in the `PORT` environment variable, which OSC sets to 8080. Use `process.env.PORT` or the runtime's equivalent. OSC also injects `APP_URL` and `AUTH_URL`. Do not set those three yourself.
- `.osc.yaml` is not supported. For a monorepo, deploy a sub-directory with `subPath`. Call `get-myapp-deploy-guide` before recommending any repository restructuring.
- Secrets go in a parameter store bound at creation (`configService`), never in the repository or the git URL.

## Steps

1. Call `get-my-plan` to see the user's plan before creating anything.
2. Call `create-my-app` with:
      - `name`: letters and digits only. No hyphens, underscores or spaces, so `my-app` becomes `myapp`.
      - `type`: one of `nodejs`, `python`, `wasm`, `golang`, `dotnet`, `php`.
      - `gitHubUrl` for GitHub, or `gitUrl` for any other HTTPS git host, with `#branch` appended if needed.
      - For a private repository, `gitCredential` with the name of a registered credential. `list-git-credentials` shows the existing ones, and `start-git-credential-registration` registers a new one without the token ever appearing in the chat. Do not use the deprecated `gitToken` or `gitHubToken` fields.
      - Optional: `subPath`, `configService`.
3. A `buildStatus: "building"` response means the request was **accepted**, not that the app runs. Call `wait-for-app-ready` with the `appId`. Each call waits up to about 50 seconds. Call it again only while it returns `checkAgain: true`, and tell the user it is still building before each repeat. Do not poll `get-my-app` or `list-my-apps` instead.
      - Stop waiting and go to step 5 if it returns `consecutiveTimeouts` of 3 or more, `checkAgain: false` without `ready: true`, or a `diagnostic.healthStatus` of `"not running"`.
4. When it returns `ready: true`, show the `URL` from the `create-my-app` response as the **Live URL**. Never substitute a runner URL.
5. If `buildStatus` is `"failed"` or the app is stuck, call `diagnose-my-app` with **`appName`** (this tool takes `appName`, not `appId`), then `get-my-app-logs` with the `appId` and no `level` filter. Report the literal error. A single "not found" during the first 10 minutes can be transient: call `wait-for-app-ready` once more after 10 to 30 seconds before recreating anything.

## Updating later

- New commits: call `restart-my-app` with the `appId` (it pulls the latest code and restarts), then `wait-for-app-ready` with `awaitNewGeneration: true`.
- A different branch or tag: call `update-my-app-source-ref`.
- A rebuild fails with "Source repository is not reachable (404)", or the logs show a git clone 403: the git token has expired. Check `list-git-credentials`, then call `update-my-app-github-token` with a `gitCredential`.

## Cost and plan

A My App costs about 10 tokens a day, and the `create-my-app` response states the user's new daily burn. If a call fails on plan or token grounds, tell the user which plan covers it and stop. Do not retry the call.

## Expected outcome

- A Live URL on `apps.osaas.io` that answers within a minute or two for a small Node.js app
- The same URL after every restart or rebuild
