# enroll vs install — Why the Agent Showed Offline

## Symptom
Elastic Agent enrollment on `linuxt` reported "Successfully enrolled the Elastic Agent",
but the Fleet UI showed the agent as **offline**, and:

```bash
sudo systemctl status elastic-agent
# Unit elastic-agent.service could not be found.
```

## Root cause
The command used was:

```bash
sudo ./elastic-agent enroll --url=https://<fleet-server-ip>:8220 \
    --enrollment-token=<TOKEN> --insecure
```

`enroll` performs a one-time registration handshake only. It does **not** create a
persistent systemd service — so there was nothing left running to send ongoing
heartbeats/check-ins after the initial handshake completed.

## Fix
Re-ran using `install` instead of `enroll`, from the clean top-level extracted binary
(not a leftover nested `data/elastic-agent-<hash>/` path from an earlier failed
attempt):

```bash
cd /home/linuxt/elastic-agent-8.11.0-linux-x86_64
sudo ./elastic-agent install --url=https://<fleet-server-ip>:8220 \
    --enrollment-token=<TOKEN> --insecure --force
```

`install` performs the enrollment handshake **and** registers `elastic-agent` as a
systemd service that persists across reboots and keeps checking in.

## Verification

```bash
sudo systemctl status elastic-agent
```

Result: `Loaded: loaded (...enabled...)`, `Active: active (running)` — matching the
Fleet Server host's own agent status. Fleet UI flipped from offline to healthy shortly
after.
