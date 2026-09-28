---
name: hermes-m365-colleague
description: Integrate Hermes into Microsoft 365 as a colleague.
version: 1.0.0
author: Paula Rossi (SMF Works)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [microsoft-365, teams, outlook, graph, hermes, colleague]
---

# Hermes as a Microsoft 365 colleague

Stand up Hermes the way a person sits in a tenant: its own licensed user, a Teams bot that people can message, delegated Graph for that user's mailbox and files, and optional Desktop panes for Teams and Outlook on the web.

This is not Azure Bot Service. It is not a shared admin login. It is not a homemade mail client.

Official Hermes Teams walkthrough: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/teams

Meeting transcripts are a different setup: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/teams-meetings

## When to Use

- Add a Hermes profile to Microsoft 365 as its own user
- Teams DMs and @mentions without an Azure subscription
- Mail, calendar, and files as that user via delegated Graph
- Embed Teams and Outlook on the web in Hermes Desktop

Don't use for: redeeming product keys, consumer Microsoft accounts, or posting meeting summaries (that pipeline is separate).

## Hard rules

1. **Two doors. Never merge them.** The Teams bot registration and the Graph delegated public client are different apps. The bot client cannot read `/me`, mailbox, or OneDrive. The Graph client is not the bot Teams will JWT.
2. **The agent gets its own licensed user.** Device-code and Graph calls run as that UPN, never as the tenant admin.
3. **No Azure subscription means `--teams-managed`.** Do not run `az bot create` or `teams app create --azure` unless `az account list` shows a real subscription. A tenant login that reports no subscription cannot create Bot Service resources.
4. **Align tenants before you create anything.** `teams status`, `az account show`, and the target org must be the same tenant. A leftover login registers the bot in the wrong directory.
5. **After `teams app create`, use that app's client id and secret.** An earlier Entra registration is not the bot. Write `TEAMS_CLIENT_ID` and `TEAMS_CLIENT_SECRET` into that profile's `.env`. Never paste secrets into chat, commits, issues, or this repo.
6. **Never paste M365 passwords or product keys.** The human redeems keys and signs in. The agent writes files and checks health.
7. **One bot per profile, one port.** The first colleague uses `3978`. The next uses `3979`. Do not point two profiles at one webhook.
8. **Desktop UI is app-global. The plugin API is not.** A plugin enabled only on the default home 404s when the window's backend is a named profile. Enable it on that profile and restart that backend. Routes mount at process start.
9. **Do not invent an inbox.** If the Graph token lacks `ChannelMessage.Read.All` or `Mail.Read`, host the web client. An empty homemade list is a lie.
10. **Never print or commit tokens.** `microsoft_token.json` and `.env` stay on the machine.

## Prerequisites

- A Microsoft 365 tenant and an admin who can create users and assign a seat
- One licensed user per Hermes colleague (example: `agent@yourdomain.com`)
- Node.js for `@microsoft/teams.cli`
- A public HTTPS tunnel (devtunnel, ngrok, or cloudflared) or a production domain
- Azure CLI only if you want Graph token checks. It is not required to create a Teams-managed bot
- Hermes Desktop, if you want the Teams and Outlook panes

Confirm the tenant before any create:

```bash
teams status
az account show
```

Completion: both report the same tenant id as the org you mean to join.

## Procedure

Work one colleague at a time. Finish verification for that profile before starting the next.

1. **Create the user and assign a seat.** Admin does this in Microsoft 365 admin. The agent does not redeem product keys. Completion: the user can sign in at https://www.office.com.
2. **Read `references/teams-bot.md` and create the Teams-managed bot.** Completion: `curl http://127.0.0.1:<TEAMS_PORT>/health` returns `ok`, and a DM to the bot gets a reply.
3. **Read `references/graph-delegated.md` and sign that user into Graph.** Completion: a `/me` call as that user returns their own UPN, not the admin's.
4. **Optional: install the Desktop panes in `references/desktop-panes.md`.** Completion: Teams and Outlook open inside Hermes Desktop, not only in an external window.
5. **Second colleague:** new profile, new port, new bot registration, new Graph device-code as that second UPN. Do not copy the first profile's `.env`.

## Desktop panes shipped with this pattern

These are the reference panes. Install them. Do not reimplement a mail or channel list on top of a token that cannot read messages.

- Teams in the pane: https://github.com/smfworks/SMFWorks-Teams-Plugin
- Outlook on the web in the pane: https://github.com/smfworks/SMFWorks-Outlook-Plugin

## Pitfalls

- `teams app create` without `--teams-managed` on a tenant with no Azure subscription fails, or it targets the wrong directory.
- Tunnel up and gateway down looks like a Teams outage. Check local `/health` before blaming the bot.
- Device codes expire in about 15 minutes. One live code at a time. Do not stack a second code while the first is pending.
- Sideload blocked: Users need **Upload custom apps** on the Global app setup policy, or org-wide custom apps enabled. See `references/teams-bot.md`.
- A `404 {"detail":"Plugin not found"}` on `/api/plugins/<id>/status` is an enable-gate miss, not a Graph sign-in failure.
- Launcher URLs (`/l/`, `/dl/`) open the desktop app. Inside a Desktop webview those buttons do nothing. The panes above skip them.
- Do not cover a `<webview>` with an absolutely positioned sibling. The guest paints on top, and the sibling eats the clicks.

## Verification

Declare the colleague live only when all of these are true:

- [ ] User exists, has a seat, and can open office.com
- [ ] `teams status` tenant matches the org
- [ ] Local `/health` on that profile's port returns `ok`
- [ ] A Teams DM to the bot gets a reply
- [ ] Graph `/me` as the agent UPN returns that user, not the admin
- [ ] Secrets are in the profile `.env` and token file, not in chat or git
- [ ] If Desktop panes are in scope, Teams and Outlook render inside Hermes Desktop

Load the matching reference before each phase. Do not skip a check because a command exited 0.
