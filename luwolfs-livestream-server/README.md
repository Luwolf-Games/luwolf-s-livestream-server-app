# Luwolf Livestream Server

Windows desktop application that streams your display, a window, or OBS Studio output to viewers watching in their browser, on TV, or on mobile, across the local network or over the internet, without Twitch or YouTube. The video is sent directly from your PC: the app encodes with ffmpeg and delivers via MediaMTX, and viewers simply open a link. Designed for small audiences (around 5 viewers), as each viewer receives a copy through your upload bandwidth.

## Using the Portable Package

### Download and Launch

1. Download `luwolf-livestream-server-portable-X.Y.Z.zip` from the repository's [Releases](../../releases) page.
2. Extract it to any folder. No installation is required: the zip includes ffmpeg and MediaMTX in `vendor/`.
3. Run `luwolf-livestream-server\luwolf-livestream-server.exe`.

If the app warns that ffmpeg or MediaMTX is missing, the `vendor\` folder next to the `.exe` was deleted or moved: extract the zip again. To verify the package without opening a window, run `luwolf-livestream-server.exe --self-check` in a terminal.

### Streaming

1. Click **Choose source** and select a monitor, a window, or **OBS Studio (WHIP)**. When capturing a window, the app captures only audio from that program.
   Minimized windows appear in the list labeled as "minimized", without a thumbnail: when starting, the app restores the window and streams it. If you minimize the window during the stream, viewers stay on the last frame until you restore it (in HDR capture, the stream drops if it remains minimized for more than ~10 s).
2. Enable at least one output:
   - **Real time**: WebRTC, for web browsers. Aiming for under 1 s latency; not yet measured across every hardware setup.
   - **Compatible**: HLS, for TVs, VLC, and legacy devices, with a few seconds of latency. Shown on the web page as "Stable".
3. Click **Start stream**. The app only reports "Live" once the video feed reaches the server.

Default video codec is H.264 (NVENC on NVIDIA GPUs, otherwise CPU). AV1 is only used if explicitly selected, and iPhones as well as older TVs do not support it. With AV1 on CPU (SVT-AV1), the stream is delivered via **Real-time** only: **Compatible** is disabled for that stream because MediaMTX 1.21.2 does not build HLS with SVT-AV1 video (ADR-0015). NVENC AV1 supports both.

If the computer cannot keep up with the selected quality, the log warns ("FFmpeg is encoding at 0.8x real-time speed"): lower the quality, frame rate, or switch to NVENC.

### Streaming via OBS Studio (WHIP)

Requires OBS Studio 30 or newer.

1. In the app, select **OBS Studio (WHIP)** and click **Start stream**. The OBS card shows the **WHIP Server** (`http://127.0.0.1:8889/live/whip`) and the key.
2. In OBS, go to **Settings > Stream**, choose the **WHIP** service, paste the server URL, and paste the key into **Bearer Token** (**Copy** button). The key changes with every stream session.
3. In **Settings > Output**, use an H.264 encoder.
4. Click **Start Streaming** in OBS.

For an OBS version without WHIP, turn on **Settings > Advanced > RTMP input for OBS** (port 1935) and use the **RTMP Server** and **RTMP Key** displayed by the app. Latency will be slightly higher.

### Watching

- On this PC: `http://localhost:8787`.
- On the local network (phone, TV, another PC): `http://<this PC's IP>:8787`. The app displays the ready-to-use link.

The web page port is 8787 by default and can be changed in the app. Everything the viewer accesses goes through it: the page, **Real-time** (`/live/whep`), and **Stable** (`/live/index.m3u8`, the URL for VLC and TVs). The app proxies video to MediaMTX, which only listens locally on the machine (ADR-0014).

### Watching over the Internet (Public Mode)

1. Turn on **Public mode**. When turned off, no one outside the local network can watch.
2. Choose whether internet viewers require an invitation:
   - **Require an invite** enabled (default): the public link includes a token, and only users with that link can watch. Toggling Public Mode off and on regenerates the token and invalidates the previous link, including viewers already watching via Stable.
   - Disabled: anyone with the address can watch. Publishing still requires the session key.
3. In your router/modem, forward to this PC's IP:

   | Port | Protocol | Purpose |
   |---|---|---|
   | 8787 (the web page port) | TCP | Web page, Real-time and Stable |
   | 8189 | UDP | Real-time video media (WebRTC) |
   | 8892 (TCP and UDP) and 8893 (UDP) | | Only with MoQ (experimental) enabled in **Advanced** |

   If the router forwards from another external port (for example 80 → 8787), links will follow the port the viewer connected on.
4. If your public IP is dynamic, create a free hostname on [DuckDNS](https://www.duckdns.org) (or similar) and enter it under **Settings > Access > Public address** (hostname only, such as `my-name.duckdns.org`, without `http://` or port). Without it, the app uses whatever public IP it detects.

### HDR Displays

With Windows HDR enabled, standard capture appears washed out or blown out: Windows stores the desktop in high dynamic range, and the app used to capture in 8-bit, clipping everything above standard white. When the selected display is in HDR, the app captures in floating-point and tone-maps to SDR before encoding: the UI retains its normal brightness (the "SDR content brightness" set in Windows Display Settings) and highlights in HDR games and videos are compressed rather than clipped. Viewers receive a standard SDR stream (ADR-0016).

- Tone-mapping runs on the GPU via Vulkan. Without Vulkan, the stream falls back to standard capture and logs "HDR active on monitor: image may appear blown out". Turn off Windows HDR during the stream, or run the diagnostic below and share the results.
- A window on an HDR display is only tone-mapped if the bundled ffmpeg has `gfxcapture`; otherwise it falls back as before with a warning. Capturing the entire monitor resolves this.
- The toggle is located under **Settings > Advanced > Convert HDR to SDR**, enabled by default, and conversion only runs with an **NVENC** codec. With CPU encoding (libx264, SVT-AV1), floating-point capture combined with CPU encoding slows down the stream significantly, so the app keeps standard capture and logs an explanation.

To run diagnostics, close the app and execute in a terminal inside the `.exe` folder:

```bat
luwolf-livestream-server.exe --hdr-probe > hdr.txt
```

The app reports, per display, whether HDR is enabled and the SDR white level, whether conversion works on this PC and how fast, and saves into `hdr-probe\` an image of each capture method (`monitor1-a-bitblt.png`, `…-d-ddagrab-fp16-libplacebo.png`, which is the one used live, and others) along with `relatorio.txt`.

### Measuring Latency

1. Under **Settings > Advanced**, enable **Diagnostic clock in the video**. The top-right corner of the video will display the local PC clock with milliseconds, recorded when the frame is encoded.
2. Open the page on this PC (or on a device with synchronized clock) and compare the video clock with the Windows clock using a screen photo or screenshot. The difference is the latency from encoder to viewer display; capture adds up to one additional frame.
3. On the web page, **Player delay** shows the WebRTC jitter buffer (Real-time) or distance from the live edge in HLS (Stable). This represents player buffering only.
4. Turn off the clock after testing.

Measured on Linux using bundled ffmpeg and MediaMTX (640x360, 30 fps, libx264), from captured frame to decoded frame: ~0.2 s on Real-time (Chromium), 0.3 s reading local RTSP, and 1.7 s on Stable. On Windows with actual screen capture, measurements are still pending.

## Development

The app only runs on Windows. Code and tests that do not depend on Windows also run on Linux and WSL.

The app is written in Rust with a GPUI interface ([ADR-0019](docs/decisions/0019-reescrever-em-rust-com-gpui.md)); repository tools live in the `xtask` crate ([ADR-0026](docs/decisions/0026-so-rust-no-repositorio.md)).

```sh
mise install          # Rust (with the Windows target) and cargo-about, as pinned in mise.toml
mise run check        # fmt, clippy (Linux and Windows targets), tests, crate boundaries: same as CI check job
mise run vendor       # downloads MediaMTX and ffmpeg from vendor.toml to vendor/ (cargo xtask vendor)
cargo run -p luwolf-app --bin luwolf-livestream-server   # launches the app (Windows)
mise run ui:linux     # opens the interface in WSLg, without capture
mise run build        # portable zip (Windows; cargo xtask dist)
mise run fetch:linux        # Linux builds of the same versions in vendor-linux/ (WSL, CI)
mise run test:integration   # real MediaMTX, ffmpeg and web page on 127.0.0.1
```

Without `vendor/`, the app offers to download the pinned ffmpeg and MediaMTX, or looks for them in a chosen folder and in PATH.

Integration tests (`#[ignore]` tests in `crates/runtime/tests/`) run the full stack on Linux: web page proxy, MediaMTX with app-generated configuration, and the stream session with a fake synthetic frame-numbered display. They cover each encoder (NVENC only when an NVIDIA GPU is present), single port routing, Public Mode with and without token, latency, and HDR-to-SDR conversion (with Mesa software Vulkan when no GPU is present). They run in a dedicated CI job outside `mise run check`.

### Releasing a Version

Releases are automated by [release-please](https://github.com/googleapis/release-please) from the [Conventional Commits](https://www.conventionalcommits.org/) on `master` ([ADR-0024](docs/decisions/0024-versao-e-changelog-pelo-release-please.md)):

1. On every push to `master`, the **Release** workflow keeps a release PR open. It bumps the version in `[workspace.package]` of `Cargo.toml` (and `Cargo.lock`) and adds the new section to `CHANGELOG.md`. Before 1.0, `feat` and `fix` bump the patch and a breaking change bumps the minor.
2. Merge the release PR when you want to release. The workflow tags `vX.Y.Z`, verifies the tag matches that version (`cargo xtask check-tag`), builds `luwolf-livestream-server-portable-X.Y.Z.zip`, runs `--self-check`, attaches the zip and publishes the prerelease, with the changelog section as notes.

Never bump the version or edit `CHANGELOG.md` by hand. Builds never edit repository files: version information comes solely from `Cargo.toml`.

Before making changes, please read:

- [`AGENTS.md`](AGENTS.md): commands, pitfalls, and conventions.
- [`ARCHITECTURE.md`](ARCHITECTURE.md): project structure and component boundaries.
- [`docs/decisions/`](docs/decisions/README.md): Architecture Decision Records (ADRs) and rationale.
- [`docs/plano-de-migracao-rust.md`](docs/plano-de-migracao-rust.md): the Rust migration plan, its `migrate/NN-*` phases and Windows test scripts.

## Licenses

The portable zip includes in `LICENSES/`: MediaMTX license (MIT), ffmpeg license (GPLv3), instructions on obtaining ffmpeg source code, the licenses of the Rust crates compiled into the executable (`crates.html`, generated by cargo-about) and the Geist font license (OFL-1.1).
