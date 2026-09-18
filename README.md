# OpenHands Automations

## ACP/OpenCode baseline

The OpenHands default agent profile is configured for:

- ACP server: `opencode`
- ACP command: `npx -y opencode-ai@1.18.31 acp`
- ACP model: `github-copilot/gpt-5.6-luna`

The OpenCode configuration declares the GitHub Copilot models used by the
profile and its subagents. Authenticate OpenCode as the `openhands` user and
persist its credential file outside the container before recreating the
container:

```sh
mkdir -p "$HOME/.openhands/opencode-data"
docker cp <current-container>:/home/openhands/.local/share/opencode/auth.json \
  "$HOME/.openhands/opencode-data/auth.json"
chmod 600 "$HOME/.openhands/opencode-data/auth.json"
```

The replacement container must mount that directory at
`/home/openhands/.local/share/opencode`. Without this mount, the GitHub
Copilot OAuth credential is lost when the current `--rm` container is
recreated.

The current container was started without this mount. Recreate it with the
existing mounts plus:

```sh
-v "$HOME/.openhands/opencode-data:/home/openhands/.local/share/opencode"
```

Verify the ACP installation after starting the replacement container:

```sh
docker exec <container> sh -lc \
  'npx -y opencode-ai@1.18.31 models github-copilot'
```

The issue processor uses the OpenHands SDK's native `ACPAgentSettings` with
`acp_server: opencode`; it does not call the OpenHands LLM directly. The
automation's `model` value is the default OpenCode model identity, while
`AUTOMATION_MODEL` can override it with another GitHub Copilot model ID.
