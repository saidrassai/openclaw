
# OpenClaw: exact setup I ran (step‑by‑step)

Goal: get OpenClaw running from source on Ubuntu with sane defaults and a working Gateway.

## 0) Preflight (context)
- We are on Ubuntu 22.04, repo at /home/ubuntu/openclaw.
- Bun’s node wrapper was on PATH, which broke the build. We fixed this by installing Node 22 via nvm.

## 1) Install Node 22 (fixes bun “node” wrapper issues)
- Command: curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
	- Why: install nvm so we can pin Node 22 reliably.
- Command: export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
	- Why: load nvm in the current shell.
- Command: nvm install 22 && nvm use 22 && node --version
	- Why: install and activate Node 22+ (required by OpenClaw).

## 2) Install repo deps
- Command: pnpm install
	- Why: install workspace dependencies.

## 3) Build the UI (required for Control UI)
- Command: pnpm ui:build
	- Why: builds the web Control UI assets in dist/control-ui.

## 4) Build OpenClaw from source
- Command: pnpm build
	- Why: bundles A2UI + compiles TypeScript to dist/.

## 5) Run onboarding (creates config + installs gateway service)
- Command: pnpm openclaw onboard --install-daemon
	- Why: wizard configures auth, channels, pairing, and installs systemd user service.

## 6) Fix pnpm global bin (needed for some skill installs)
- Command: pnpm setup
	- Why: sets PNPM_HOME and PATH for global installs.
- Command: source ~/.bashrc
	- Why: loads PNPM_HOME into the current shell.

## 7) Repair service PATH + permissions (doctor)
- Command: pnpm openclaw doctor --repair --yes
	- Why: fixes service PATH to include PNPM_HOME and tightens ~/.openclaw permissions.

## 8) Verify gateway status
- Command: pnpm openclaw gateway status
	- Why: confirm systemd service is enabled and the gateway is listening on 127.0.0.1:18789.

## 9) Get Control UI tokenized URL (auth required)
- Command: pnpm openclaw dashboard --no-open
	- Why: prints a tokenized URL for the Control UI.

## 9.1) Fix Control UI auth errors (if you see token missing)
- Action: open the tokenized URL printed above, or paste the token into Control UI settings.
	- Why: the gateway requires auth and will reject unauthenticated UI connections.

## 10) Optional: check health + security
- Command: pnpm openclaw status
	- Why: diagnostic summary.
- Command: pnpm openclaw security audit --deep
	- Why: security posture checks for channels and tools.

## 11) Switch default model to OpenRouter (arcee-ai/trinity-large-preview:free)
- Command: pnpm openclaw models set openrouter/arcee-ai/trinity-large-preview:free
	- Why: sets the default model for the agent.

## 12) Register OpenRouter model in config (avoid “Unknown model”)
- Command:
  pnpm openclaw config set models.providers.openrouter '{"baseUrl":"https://openrouter.ai/api/v1","api":"openai-responses","models":[{"id":"arcee-ai/trinity-large-preview:free","name":"Arcee Trinity Large Preview (free)","reasoning":false,"input":["text"],"cost":{"input":0,"output":0,"cacheRead":0,"cacheWrite":0},"contextWindow":32768,"maxTokens":4096}]}' --json
	- Why: adds the model definition so the runtime recognizes it.

## 13) Restart the gateway to apply model config
- Command: pnpm openclaw gateway restart
	- Why: reloads the new model config.

## 14) Send a WhatsApp test message (manual)
- Command: pnpm openclaw message send --target +212675944495 --message "Test message from OpenClaw"
	- Why: verifies outbound messaging.

## 15) Trigger an agent reply to WhatsApp
- Command: pnpm openclaw agent --to +212675944495 --message "Test agent reply after model update." --channel whatsapp --deliver
	- Why: verifies the agent can respond and deliver via WhatsApp.

---

## Best‑practice notes (brief)
- Use Node 22+; avoid bun for WhatsApp/Telegram in production.
- Keep personal config in ~/.openclaw/openclaw.json and workspace in ~/.openclaw/workspace.
- Use systemd user service on single-user hosts; enable linger if needed.
- Keep the gateway bound to loopback unless you intentionally expose it.
- Use pairing/allowlist by default; don’t open DMs without reviewing security.

