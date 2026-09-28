# Teams bot (no Azure subscription)

Create a Teams-managed bot for one Hermes profile. Official walkthrough: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/teams

## Before you create

1. `teams login` as an admin of the **target** tenant. Device code is fine. One live code at a time.
2. `teams status` and `az account show` (if Azure CLI is present) must show that same tenant.
3. Pick a port. First colleague: `3978`. Second: `3979`. Set `TEAMS_PORT` to match.
4. Expose that port with a public HTTPS tunnel. Teams cannot call localhost.

```bash
npm install -g @microsoft/teams.cli@preview
teams login
teams status --verbose
```

devtunnel example (replace the port if this is not the first colleague):

```bash
devtunnel create hermes-bot --allow-anonymous
devtunnel port create hermes-bot -p 3978 --protocol http
devtunnel host hermes-bot
```

The public URL is HTTPS. The local listener is plain HTTP. Do not mark the local port as HTTPS.

## Create

No Azure subscription:

```bash
teams app create --name "Hermes" --teams-managed --endpoint "https://<tunnel-host>/api/messages"
```

If `az account list` shows a real subscription and you intend to use Azure Bot Service, follow the official doc instead of `--teams-managed`. Do not mix the two.

The CLI prints a client id, a client secret, a tenant id, and an install link. The secret is shown once.

Write them into **that profile's** `.env`, not the default home, unless this colleague is the default profile:

```bash
TEAMS_CLIENT_ID=<client id from teams app create>
TEAMS_CLIENT_SECRET=<secret from that same create>
TEAMS_TENANT_ID=<tenant id>
TEAMS_PORT=3978
TEAMS_ALLOWED_USERS=<aad object ids, comma-separated>
```

`TEAMS_ALLOWED_USERS` comes from `teams status --verbose`. Without it, anyone who can install the bot can talk to it.

Also set `platforms.teams.enabled: true` in that profile's `config.yaml`.

Do not point `TEAMS_CLIENT_ID` at an older Entra app you created by hand. Teams JWTs are for the app `teams app create` just made.

chmod `600` on `.env` where the filesystem supports it.

## Start and prove it

```bash
hermes gateway run
curl http://127.0.0.1:3978/health
```

Completion: the body is `ok`, and the log contains `Webhook server listening` on that port.

Then:

```bash
teams app get <teamsAppId> --install-link
```

Open the link, install, and send a DM. Completion: the bot replies.

On Windows, if `hermes.exe` is awkward to launch from Git Bash, use `references/windows-gateway.md`.

## Sideload blocked

If custom apps cannot be uploaded:

- https://admin.teams.microsoft.com/policies/app-setup — Global — **Upload custom apps** On
- or https://admin.teams.microsoft.com/manageapps — org-wide custom apps

## Two colleagues

Do not reuse the first bot. Second profile, second `teams app create`, second port, second tunnel port, second `.env`. A shared tunnel host with two port mappings is fine. A shared client secret is not.

## Failure looks like

| Symptom | Cause |
| --- | --- |
| Teams 502 / bot not responding | Tunnel is up, local `/health` is not |
| Bot created in the wrong org | `teams login` tenant did not match the target org |
| JWT failures after you reused an old app id | `TEAMS_CLIENT_ID` is not the id from this `teams app create` |
| Works in DM, silent in a channel | Channel messages need an @mention unless you granted RSC and set `TEAMS_REQUIRE_MENTION` |
