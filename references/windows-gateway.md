# Windows: start a profile gateway

Use this when Git Bash cannot launch `hermes.exe` cleanly, or when the Teams webhook must survive logon.

Do not start the gateway from a short-lived agent shell and call that persistence. The child dies with the runner.

## One-shot start

Set `HERMES_HOME` to the colleague's profile directory, not the default home, unless this colleague is the default profile.

```python
import os, subprocess
env = os.environ.copy()
env["HERMES_HOME"] = r"C:\path\to\hermes\profiles\<profile>"
exe = r"C:\path\to\hermes-agent\venv\Scripts\hermes.exe"
log_path = os.path.join(env["HERMES_HOME"], "logs", "teams-gateway-stdout.log")
os.makedirs(os.path.dirname(log_path), exist_ok=True)
log = open(log_path, "ab")
subprocess.Popen(
    [exe, "--profile", "<profile>", "gateway", "run"],
    env=env,
    stdout=log,
    stderr=subprocess.STDOUT,
    cwd=env["HERMES_HOME"],
)
```

Completion: `curl http://127.0.0.1:<TEAMS_PORT>/health` returns `ok`, and the port is listening.

A Windows `asyncio.start_unix_server` warning in a watchdog is non-blocking if health is ok.

## Persist across logon

Register a Scheduled Task per colleague:

- Trigger: at logon of the user who owns the machine
- Restart on failure
- Unlimited execution time
- The task runs the same gateway command, with that profile's `HERMES_HOME` and port

Register a separate task for the tunnel. Start the tunnel task first, then each gateway task.

Do not merge two colleagues into one task. Do not point a colleague task at a different profile's Telegram bot. If the default profile already runs a Telegram gateway, the Teams profile must not reuse that token.

## Verify from outside

Local health is not enough. From another network path, the public `/api/messages` URL must answer. An empty POST that returns 401, 415, or 500 means the webhook was hit. A timeout means the tunnel or the listener is down.

Gold-standard check: reboot, wait two minutes, both `/health` endpoints and the public webhook still answer. Do not reboot a machine from an agent session unless the owner asked.
