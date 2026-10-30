# Linux WebKit blank Login / Device diagnosis

Snapmaker Orca's **Prepare** and **Preview** use the native slicer. **Login**, **Home**, and **Device** use an embedded WebKitGTK web view to render local Flutter pages. If only these web pages are blank, do not reset printer profiles.

This branch adds native Linux logging for two otherwise hard-to-distinguish cases:

- `[WEBVIEW][linux] WebKit web process terminated, reason=N`: the separate web renderer exited (for example during EGL initialization or WebGL setup).
- `[WEBVIEW][linux] WebKit load failed: ...`: WebKit rejected a page load, with its own error text.

It also logs a failed `wxWebView::Create` or a missing native backend. It does **not** change WebKit settings, user agents, the Flutter bundle, or authentication.

## Reproduction

Build the project using the repository's Linux build instructions. Close all previous Snapmaker Orca processes and launch the newly built executable **from a terminal** so its logs are captured. Switch between **Prepare**, **Home/Login**, and **Device**; record whether each page works.

Capture the full process output to a file (redact tokens, printer identifiers and account details before sharing). Specifically look for `[WEBVIEW][linux]`, `EGL_BAD_PARAMETER`, `[FLUTTER_FLOW]`, and `[OrcaWeb]`.

Do not assume a WebKit upgrade or Mesa rollback is the fix without testing. On CachyOS, the reported regression followed upgrades of `mesa-git`, `webkit2gtk-4.1`, and KDE components in October 2026. The embedded WebKit GTK renderer and Flutter CanvasKit/WebGL path are the primary areas for investigation.

## Follow-up

If the page stays blank **without** a web-process termination or load failure message, instrument the Flutter JavaScript bootstrap or inspect the WebKit JavaScript console: the failure may occur after successful navigation (e.g. WebGL initialization or Dart/Flutter startup).

These changes are diagnostics only. A code or packaging workaround should be driven by the reproduced error and tested on the affected AMD RDNA4/Mesa system before release.
