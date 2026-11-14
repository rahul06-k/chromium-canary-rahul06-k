# Chromium Canary Submission — Rahul K (@rahul06-k)

## 1. Loom Walkthrough Video
* **Video URL:** [Watch 5-Minute Walkthrough](https://www.loom.com/share/bb94ed24042d482c97b6ba6729f200c1)

## 2. Build Info
* **Chromium Commit ID (`git rev-parse HEAD`):** `0a35dc167d61d8efa59ac156b65ed5301550623e`
* **Target Built:** `chrome`
* **My Laptop OS & RAM:** `Windows 11, 16GB RAM`

## 3. Architecture Notes
* **Feature A (`X-Candidate-Canary` Header):**
  Outgoing HTTP/HTTPS requests are intercepted by a custom `blink::URLLoaderThrottle` (`CanaryURLLoaderThrottle`), which adds the `X-Candidate-Canary` header inside `WillStartRequest()` whenever the switch is present. Outgoing requests are split across processes: browser-initiated navigations run through `ChromeContentBrowserClient::CreateURLLoaderThrottles` in the **Browser process**, while renderer-initiated fetches and subresources run through `URLLoaderThrottleProviderImpl::CreateThrottles` in the **Renderer process**. The `--inject-candidate-id` flag is forwarded from the Browser process to the child Renderer process by whitelisting `"inject-candidate-id"` in `kSwitchNames` inside `content/browser/renderer_host/render_process_host_impl.cc`, ensuring `CopySwitchesFrom` propagates it safely across process boundaries without requiring `--no-sandbox`.

* **Feature B (`[CANARY_ACK]` Console Hook):**
  In Blink, JavaScript `console` API calls route through `FrameConsole::AddMessage` in `third_party/blink/renderer/core/frame/frame_console.cc` inside the **Blink Renderer process**. The hook checks if the incoming `ConsoleMessage` has a severity of `mojom::blink::ConsoleMessageLevel::kWarning` and message content equal to `"canary-ping"`; if matched, it logs `[CANARY_ACK] Verified build for rahul06-k at <Timestamp>` directly to `stderr` using `LOG(INFO)` and runtime `base::TimeTicks::Now()`, while preserving standard console behavior for all other logs.

## 4. AI Agent Workflow
* **AI Tool & Model Used:** Antigravity / Gemini CLI with Gemini 3.8 Flash (High) and sub-agent task routing.
* **How I Used Sub-Agents:**
  Given Chromium's ~35 million lines of code, work was distributed across specialized sub-agents:
  1. *Sub-Agent 1 (Blink Explorer)* located `third_party/blink/renderer/core/frame/frame_console.cc` and verified header access under Blink's strict `DEPS` rules for `base/time/time.h` and `base/logging.h`.
  2. *Sub-Agent 2 (IPC & Pipeline Explorer)* identified the switch propagation mechanism in `render_process_host_impl.cc` via `kSwitchNames` and determined the lifecycle placement for `blink::URLLoaderThrottle` in `url_loader_throttle_provider_impl.cc` and `chrome_content_browser_client.cc`.
  3. The main orchestrator synthesized surgical edits strictly in `.cc` implementation files, preventing header cache invalidations and keeping incremental rebuilds under 30 seconds.
