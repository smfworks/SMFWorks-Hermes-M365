# Delegated Graph as the colleague

The Teams bot cannot read the colleague's mailbox, calendar, or `/me`. That is a second app: a public client, device-code as the agent user.

## App registration

In the same tenant, register a public client (mobile/desktop). Name it something you will recognize, for example `Hermes Graph Delegated`.

Delegated permissions to request, then admin-consent:

- `User.Read`
- `offline_access`
- `Mail.Read`
- `Calendars.Read`
- `Files.Read`

Add `Mail.ReadWrite`, `Calendars.ReadWrite`, or `Files.ReadWrite` only if the colleague must send mail or edit files. Do not grant application permissions on this app for `/me` work. Application permissions do not have a `/me`.

Do not add these permissions to the Teams bot app. Do not put the bot's client id in the Graph auth script.

Record, in the profile, not in git:

- Application (client) id
- Tenant id
- Redirect: device code, so no secret is required for a public client

## Sign in as the agent

The human completes device code in a browser while signed in as `agent@yourdomain.com`, not as the admin.

One code at a time. Codes expire in about 15 minutes. If a poll is already running, do not start another.

Store the resulting token in that profile, for example `microsoft_token.json` under the profile home. Never print the access token or refresh token. Never commit the file.

A check that proves the door is the right one:

```bash
# Call Microsoft Graph /me with the stored token.
# Completion: userPrincipalName is the agent UPN, not the admin.
```

If `/me` returns the admin, you signed in as the wrong account. Delete the token file and device-code again as the agent user.

## What a normal token cannot do

`Mail.Read` on this delegated app can list the colleague's own mail. It still may not be what `az login` has. Azure CLI's token is often a different app, signed in as whoever ran `az login` — frequently the admin.

Do not point the Desktop panes at `az` and call that the colleague. The panes in `desktop-panes.md` host the web client so mail and channel conversation do not depend on a missing scope.

If a Graph call returns missing scope, say so. Do not fabricate an empty inbox.

## Admin consent

If the agent user cannot consent, an admin consents the delegated permissions for the tenant once. After that, each colleague still device-codes as themselves. Consent is not a substitute for signing in as the right user.
