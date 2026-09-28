# SMF Works: integrate Hermes with Microsoft 365

A recipe other Hermes agents can follow to sit in a Microsoft 365 tenant the same way a colleague does: own user, Teams bot, delegated Graph, and Desktop panes for Teams and Outlook on the web.

Written from a working setup. It does not publish that tenant's ids, secrets, or tunnel host. You fill in yours.

## Point your agent here

Install the skill, then tell the agent to follow it:

```bash
hermes skills install smfworks/SMFWorks-Hermes-M365
```

Or, in a Hermes chat:

```text
/skills install https://github.com/smfworks/SMFWorks-Hermes-M365/blob/main/SKILL.md --name hermes-m365-colleague
```

Then:

```text
Follow the hermes-m365-colleague skill. Set me up as a Microsoft 365 colleague. My tenant is <tenant>. My agent user is <agent@domain>. Do not use the admin account for Graph. Do not paste secrets into chat.
```

The agent should load `SKILL.md` and the files under `references/` before it creates anything.

Skill system: https://hermes-agent.nousresearch.com/docs/user-guide/features/skills

## What you get

| Piece | What it is | What it is not |
| --- | --- | --- |
| Licensed M365 user | The colleague identity | The tenant admin |
| Teams-managed bot | DMs and @mentions, no Azure subscription required | Azure Bot Service |
| Delegated Graph | Mail, calendar, files, `/me` as that user | The bot's client id |
| Desktop panes | Teams and Outlook on the web inside Hermes Desktop | A homemade inbox |

## Desktop panes

Install these. Do not rebuild them.

- [SMFWorks Teams Plugin](https://github.com/smfworks/SMFWorks-Teams-Plugin) — Teams web client in the Hermes Desktop pane, with a Graph roster of joined teams
- [SMFWorks Outlook Plugin](https://github.com/smfworks/SMFWorks-Outlook-Plugin) — Outlook on the web in the pane

## Official Hermes docs

- [Microsoft Teams bot](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/teams)
- [Teams meetings](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/teams-meetings) — separate from this recipe

## Layout

- `SKILL.md` — the procedure an agent must follow
- `references/teams-bot.md` — bot, tunnel, env, sideload
- `references/graph-delegated.md` — public client, device code, `/me`
- `references/desktop-panes.md` — install the Teams and Outlook panes
- `references/windows-gateway.md` — start a profile gateway on Windows

## License

MIT. Copyright (c) 2026 SMF Works.
