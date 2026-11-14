# 5-Minute Loom Video Walkthrough Script & Step-by-Step Guide for Rahul K (@rahul06-k)

This guide ensures you score 100% on the interview review. The interviewer specifically enforces:
1. **Unedited screen video** under 5 minutes (Loom or unlisted YouTube).
2. **Microphone ON** (explain out loud while pointing to code).
3. Do **NOT** pause or cut the video.
4. Do **NOT** use `--no-sandbox` or `--single-process`.

---

## Pre-Recording Checklist (Do this BEFORE hitting Record)

1. Make sure your build is already compiled so the incremental build only takes 15–20 seconds:
   ```bash
   autoninja -C out/Default chrome
   ```
2. Have your terminal open at your Chromium `src/` directory.
3. Have your code editor (VS Code / Vim / etc.) open with `third_party/blink/renderer/core/frame/frame_console.cc`.

---

## Video Timeline & Spoken Script

### Part 1: Prove Your Build Works (0:00 – 1:30)

**Action on Screen:**
1. Switch to your terminal in `src/` and run:
   ```bash
   git status
   git diff --stat
   ```
   *Show that 4 .cc files are modified and total lines changed is < 50 lines.*

2. Switch to your code editor with `third_party/blink/renderer/core/frame/frame_console.cc`.
3. Highlight the line:
   ```cpp
   LOG(INFO) << "[CANARY_ACK] Verified build for rahul06-k at "
   ```
4. Live-edit `[CANARY_ACK]` to `[CANARY_ACK_LIVE]`. Save the file.
5. Switch to terminal and run the incremental build live:
   ```bash
   autoninja -C out/Default chrome
   ```
   *(Wait ~15–25 seconds while it links `chrome.dll` / `chrome` binary).*

**What to Say Out Loud:**
> *"Hi Sandeep and the Arakoo team! My name is Rahul. Today I am presenting my Chromium telemetry canary take-home assignment.*
> *Here is my terminal inside `src/`. Running `git status` and `git diff --stat`, you can see clean, surgical changes across just 4 `.cc` files, totaling under 50 lines of code without touching any shared header files.*
> *To prove this is a live, working build, I'm opening `third_party/blink/renderer/core/frame/frame_console.cc`. I'll change the tag from `[CANARY_ACK]` to `[CANARY_ACK_LIVE]`, save it, and run `autoninja -C out/Default chrome`.*
> *Because we enabled component builds and symbol level 0 in `args.gn`, Ninja only recompiles this single compilation unit and re-links in about 20 seconds. And as you can see on screen, the build succeeded!"*

---

### Part 2: Show Your Features Working (1:30 – 3:00)

**Action on Screen:**
1. In your terminal, launch your newly compiled binary:
   ```bash
   ./out/Default/chrome --inject-candidate-id="rahul06-k" --enable-logging=stderr
   ```
   *(Keep the terminal visible alongside the browser window).*

2. In the browser address bar, navigate to:
   ```text
   https://httpbin.org/headers
   ```
3. Zoom in on the webpage JSON output to clearly show:
   ```json
   "X-Candidate-Canary": "rahul06-k"
   ```
4. Open Chrome DevTools (`F12` or `Ctrl+Shift+I`), click the **Network** tab, refresh, click the request, and point to the `X-Candidate-Canary` header in Request Headers.

5. In DevTools, click the **Console** tab.
6. Type:
   ```javascript
   console.warn("canary-ping")
   ```
   and hit Enter.
7. Point to your terminal where Chromium printed:
   ```text
   [CANARY_ACK_LIVE] Verified build for rahul06-k at [timestamp]
   ```
8. Type a regular console warning:
   ```javascript
   console.warn("normal warning")
   ```
   and show that it outputs normally in DevTools but does **NOT** trigger the `[CANARY_ACK_LIVE]` hook in the terminal.

**What to Say Out Loud:**
> *"Now let's verify both features live.*
> *I am launching Chrome from the terminal with `--inject-candidate-id=\"rahul06-k\" --enable-logging=stderr`. Notice I am not passing `--no-sandbox` or `--single-process`; full multi-process isolation is intact.*
> *First, for Feature A, I'll navigate to `https://httpbin.org/headers`. As you can see directly in the JSON response from httpbin, as well as in the DevTools Network tab, the outgoing request has injected `X-Candidate-Canary: rahul06-k`.*
> *Now for Feature B, I'll open DevTools Console and execute `console.warn(\"canary-ping\")`. Immediately, look at our terminal stderr: `[CANARY_ACK_LIVE] Verified build for rahul06-k at` with runtime timestamp from `base::TimeTicks::Now()`.*
> *If I invoke `console.warn(\"test message\")` with any other string, standard logging behavior is preserved with no canary acknowledgment."*

---

### Part 3: Explain Your Code & Answer The Two Questions (3:00 – 5:00)

**Action on Screen:**
Run `git diff` in your terminal or show the diff in VS Code. Point your mouse cursor to the relevant lines as you answer.

**Question 1: Which OS process (Browser, Blink Renderer, or Network Service) runs Feature A, and which runs Feature B?**

**What to Say Out Loud (Point to `frame_console.cc` then throttle files):**
> *"Let's explain the architecture and answer the two questions.*
> *For **Question 1**:*
> * **Feature B** runs inside the **Blink Renderer process**. In `third_party/blink/renderer/core/frame/frame_console.cc`, JavaScript `console.warn()` invokes `FrameConsole::AddMessage`. Blink runs within the sandboxed Renderer process, so this interception and logging happens entirely on the renderer side.*
> * **Feature A** operates across the process boundary. Browser-initiated top-level navigations (like typing a URL into the omnibox) invoke `ChromeContentBrowserClient::CreateURLLoaderThrottles` inside the **Browser process**. Meanwhile, renderer-initiated requests (such as subresources, scripts, fetch, or XHR calls from webpages) invoke `URLLoaderThrottleProviderImpl::CreateThrottles` inside the **Renderer process**. Both hook into Chromium's network pipeline via `blink::URLLoaderThrottle::WillStartRequest()` before the request reaches the Network Service."*

**Question 2: How did the `--inject-candidate-id` flag value get from the main Browser process at startup to the process that adds the HTTP header?**

**What to Say Out Loud (Point to `render_process_host_impl.cc`):**
> *"For **Question 2**:*
> *When the browser executable is launched from the terminal, the main **Browser process** parses `--inject-candidate-id` into `base::CommandLine::ForCurrentProcess()`.*
> *However, child processes (like the Renderer) do not automatically inherit all browser switches for security and memory isolation.*
> *When the Browser process spawns a child renderer, `RenderProcessHostImpl` constructs the child process command line. It uses an explicit whitelist array called `kSwitchNames` inside `content/browser/renderer_host/render_process_host_impl.cc`.*
> *By adding `\"inject-candidate-id\"` to `kSwitchNames`, `renderer_cmd->CopySwitchesFrom()` copies the switch and its value across process boundaries into the child renderer's command line.*
> *Inside the Renderer process, `URLLoaderThrottleProviderImpl` inspects `base::CommandLine::ForCurrentProcess()->GetSwitchValueASCII(\"inject-candidate-id\")` and attaches our custom `CanaryURLLoaderThrottle`.*
> *All changes were done with zero header modifications, keeping rebuild times lightning fast and respecting Chromium's process isolation completely. Thank you!"*
