# Desktop panes: Teams and Outlook

Host the real web clients inside Hermes Desktop. Do not build a second inbox.

| Pane | Repo | Plugin id |
| --- | --- | --- |
| Teams | https://github.com/smfworks/SMFWorks-Teams-Plugin | `hermes-teams-inbox` |
| Outlook on the web | https://github.com/smfworks/SMFWorks-Outlook-Plugin | `hermes-outlook` |

Each repo has an `install.sh` and a README. The folder name must equal the plugin id.

## Where files go

Desktop UI is app-global:

```text
~/.hermes/desktop-plugins/<plugin-id>/plugin.js
```

On Windows that is `%LOCALAPPDATA%\hermes\desktop-plugins\<plugin-id>\plugin.js`.

The Python API lives with plugins:

```text
~/.hermes/plugins/<plugin-id>/
```

A named profile's backend only answers `/api/plugins/<id>/...` if that plugin is in **that profile's** `plugins.enabled`. A copy enabled only on the default home returns:

```text
404 {"detail":"Plugin not found"}
```

That is not a Graph sign-in error. Enable it on the profile that owns the window:

```bash
HERMES_HOME=<profile home> hermes plugins enable <plugin-id>
```

Routes mount when that backend starts. Restart it after the first enable. Then, in the Desktop command palette: **Reload desktop plugins**.

## What the panes actually do

Teams lists joined teams from Graph when the token can, and hosts the Teams web client for the conversation. Channel message reads need `ChannelMessage.Read.All`, which a normal Azure CLI token does not have.

Outlook hosts Outlook on the web. Graph, when present, only names the signed-in account. Mail stays in Outlook.

Both keep **Pop out** for a separate window. The pane is the product.

## Webview rules, already baked into those repos

If you fork them, keep these. They are why the first versions looked installed and did not work.

- Do not load `/l/` or `/dl/` launcher URLs into the webview. Those pages offer "open the desktop app". The buttons do nothing inside the pane.
- Do not cancel every navigation that is not the home URL. Sign-in and the app's own boot redirects must proceed. Cancelling them leaves a logo on screen forever.
- Follow `window.open` in the same webview when it is an https URL. An auth popup that you bounce back to the home page never finishes.
- Do not cover the webview with an absolutely positioned sibling. The guest paints on top, so the page looks clickable, and the sibling takes the clicks.
- Do not call `getURL()` until `dom-ready`. Electron throws, and the plugin page dies.
- Spoof a normal Chrome user agent. A raw Electron agent is more likely to get the "get the desktop app" wall.

## Completion

- Sidebar shows **Teams** and **Outlook**
- Each page renders the web client inside Hermes, not only an Open button
- A `Plugin not found` response, if you still call the API, is fixed by enabling the plugin on the active profile and restarting that backend
