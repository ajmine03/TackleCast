# TackleCast

<p align="center">
  <img src="assets/icon.png" alt="TackleCast" width="128">
</p>

**A lightweight, GPU-accelerated low-latency capture card viewer for Windows and Linux.** No recording bloat, no complex OBS configuration—just your console gaming on your laptop screen with synchronized high-fidelity audio.

Built specifically for console-to-laptop setups (such as **PS4 / PS5 / Nintendo Switch / Xbox → HDMI USB Capture Card → Laptop**) using devices like Genki ShadowCast, Elgato Cam Link, AVerMedia, and generic UVC/UAC USB capture cards. Written in Rust for minimal latency, zero-copy GPU video rendering, and a robust lockless audio pipeline with automatic sample rate and channel conversion.

---

## Key Features

- **Robust Audio Passthrough**:
  - Independent capture and playback stream management (never fails due to mismatched hardware rates).
  - High-precision 4-point Catmull-Rom cubic Hermite resampling (seamlessly matches PS4 48 kHz PCM, capture cards at 44.1/48/96 kHz, and host audio hardware).
  - Dynamic channel adaptation (mono capture card → stereo laptop speakers/headphones, stereo pass-through, or multi-channel downmixing).
  - Lockless single-producer single-consumer (SPSC) ring buffer minimizing latency without blocking the audio thread.
  - Full volume control (0–100%) and instant mute toggle in UI.
  - Native Windows WASAPI and Linux PipeWire / ALSA backends via `cpal`.
  - Comprehensive startup diagnostics and rate-limited underrun/overrun reporting.
- **GPU-Accelerated Video Pipeline**:
  - DirectShow (Windows) and V4L2 (Linux) device capture via FFmpeg.
  - Custom YUV-to-RGB WGSL shader rendering via `wgpu`.
  - Zero-copy NVIDIA GPU MJPEG decoding via nvJPEG/CUDA (on supported Windows configurations).
  - Automatic fallback to multi-threaded CPU decode for maximum hardware compatibility.
- **Console-Optimized Display**:
  - Resolution modes: 720p, 1080p, 1440p, 4K with aspect-ratio-preserving letterboxing.
  - Flexible FPS targets: 30, 60, 120, or Custom (30–240 FPS).
  - Nearest Neighbor and Bilinear texture filtering.
  - Fullscreen borderless mode (press `F11`).
  - Screen sleep suppression during active gameplay.
- **Clean, Minimal UI**:
  - Compact in-game settings overlay (press `Escape`).
  - Real-time Audio & Video status indicators (e.g. `Audio: Connected`, `Audio: No Input`, `FPS: 60.0`).
  - Persistent JSON configuration across restarts.

---

## Releases & Downloads

Pre-compiled, ready-to-use binaries are automatically built and published on the [GitHub Releases](https://github.com/ajmine03/TackleCast/releases) page:

- **Windows 10/11 (`.exe` bundle)**:
  1. Download `TackleCast-Windows-x86_64.zip` from Releases.
  2. Extract the archive to any folder.
  3. Double-click `TackleCast.exe` (all required FFmpeg runtime DLLs and icons are bundled).
- **Linux (`.sh` launcher)**:
  1. Download `TackleCast-Linux-x86_64.tar.gz` from Releases.
  2. Extract: `tar -xzvf TackleCast-Linux-x86_64.tar.gz && cd TackleCast-Linux-x86_64`
  3. Run: `./run.sh` (handles device node permissions and launches the binary).

---

## Hardware Setup Guide (PS4 → Capture Card → Laptop)

### 1. Physical Connections
```text
  [ PlayStation 4 / 5 ]
            │  HDMI Out
            ▼
┌────────────────────────┐
│ HDMI USB Capture Card  │
└────────────────────────┘
            │  USB 3.0 / USB-C
            ▼
     [ Laptop / PC ]  ───► [ Speakers / Headphones ]
```

1. Connect an HDMI cable from the **PS4 HDMI OUT** port into the **Capture Card HDMI IN** port.
2. Plug the USB capture card into a high-speed **USB 3.0 / 3.1 / USB-C port** on your laptop (avoid unpowered USB hubs to prevent frame drops or audio stutter).
3. Connect your headphones or use your laptop's built-in speakers.

### 2. PS4 Audio & Video Settings
For optimal compatibility:
- **Audio Output Settings**:
  - Navigate to PS4 **Settings → Sound and Screen → Audio Output Settings → Primary Output Port**: Select **HDMI OUT**.
  - **Audio Format (Priority)**: Select **Linear PCM** (2-channel stereo 48 kHz). *Avoid Dolby Digital or DTS bitstream formats, as USB capture cards only accept uncompressed PCM.*
- **HDCP Settings**:
  - Navigate to PS4 **Settings → System**: Ensure **Enable HDCP** is **UNCHECKED**. *(Capture cards cannot display video or audio if HDCP encryption is active).*

---

## Platform Setup & Running

### Windows (10 / 11)

1. **Launch TackleCast**:
   - Run `TackleCast.exe` (or `cargo run --release`).
2. **Configure Devices**:
   - Press **Escape** to toggle the settings menu.
   - **Video Device**: Select your capture card (e.g. `USB Video`, `ShadowCast`, `Cam Link 4K`).
   - **Audio Input**: Select your capture card's audio endpoint (e.g. `Digital Audio Interface (USB Digital Audio)`, `Capture Card Audio`).
   - **Audio Output**: Select your laptop's playback device (e.g. `Speakers (Realtek Audio)`, `Headphones`).
   - **Volume**: Adjust slider (0–100%) or uncheck **Mute**.
   - Check the **Audio Status** label at the bottom of the menu; it will show `Connected` in green once streams are running.
3. **Play**:
   - Press **Escape** to hide the menu.
   - Press **F11** for immersive borderless fullscreen.

### Linux (Ubuntu, Debian, Fedora, Arch, etc.)

TackleCast natively supports modern Linux desktop environments with PipeWire, PulseAudio, and ALSA, as well as V4L2 for video capture.

1. **Verify Device Recognition**:
   ```bash
   # Check video capture device node
   v4l2-ctl --list-devices
   # Check audio input devices
   wpctl status   # or: arecord -l
   ```
2. **Run TackleCast**:
   ```bash
   cargo run --release
   ```
3. **Configure Devices**:
   - Press **Escape** to open the menu.
   - Select your `/dev/video*` capture device under **Video Device**.
   - Select your capture card under **Audio Input** and your default sink/headphones under **Audio Output**.

---

## Troubleshooting: Video Works but Audio Doesn't

If video capture is displaying smoothly but no sound comes out of your speakers or headphones, follow these diagnostic steps:

### 1. Verify TackleCast Audio Status
Press **Escape** to open the settings menu and look at the **Audio Status**:
- **`Audio: Connected`**: Streams are open and active. Check your laptop master volume, output device selection, and verify TackleCast is not muted.
- **`Audio: No Input Device`**: TackleCast could not open the capture card's microphone endpoint. Verify the correct input device is selected from the dropdown.
- **`Audio: No Output Device`**: Your selected playback device is disconnected or unavailable. Choose your default laptop speakers or headphones.
- **`Audio: Error (...)`**: Format negotiation failed or the device was unplugged. Check the detailed message.

### 2. Check PS4 Audio Settings
- Go to PS4 **Settings → Sound and Screen → Audio Output Settings → Audio Format (Priority)**.
- Ensure **Linear PCM** is selected. Encoded streams (Bitstream Dolby or DTS) cannot be decoded by basic UVC/UAC capture dongles and will output pure silence.

### 3. Windows Sound & Privacy Settings
- **Microphone Privacy Permission**: Windows treats capture card audio inputs as microphones.
  - Open Windows **Settings → Privacy & Security → Microphone**.
  - Ensure **Microphone access** and **Let desktop apps access your microphone** are turned **ON**.
- **Windows Sound Control Panel**:
  - Press `Win + R`, type `mmsys.cpl`, and hit Enter.
  - Go to the **Recording** tab, find your capture card (often labeled *Digital Audio Interface* or *USB Audio*), right-click → **Properties → Advanced**.
  - Test setting default format to **2 channel, 16 bit, 48000 Hz (DVD Quality)**.
  - Ensure the device is not muted or disabled.

### 4. Linux PipeWire / PulseAudio Settings
- Open `pavucontrol` (PulseAudio Volume Control).
- In the **Configuration** tab, ensure your capture card profile is set to **Pro Audio** or **Digital Stereo (IEC958) Input**.
- In the **Recording** tab, verify `tacklecast` is capturing from the correct source and that the volume meter is moving.

### 5. Inspect Application Logs
TackleCast writes detailed rotating diagnostic logs to `logs/tacklecast_*.log`.
At startup, it logs every detected video, audio input, and audio output device, along with sample rates, formats, and channels. If an audio error or buffer underrun occurs, it will be logged with a timestamp:
```text
=== Audio & Video Diagnostics at Startup ===
detected video capture devices (1):
  [0] USB Video
detected audio input devices (2):
  [0] Digital Audio Interface (USB Digital Audio)
  [1] Microphone Array (Realtek Audio)
detected audio output devices (2):
  [0] Speakers (Realtek Audio)
  [1] Headphones (Realtek Audio)
Audio input stream config: sample_rate=48000Hz, channels=2, format=I16
Audio output stream config: sample_rate=48000Hz, channels=2, format=F32
Audio input started successfully
Audio output started successfully
```

---

## Controls

| Action | Key / Gesture |
|---|---|
| Open / Close Settings Menu | `Escape` |
| Toggle Fullscreen | `F11` |
| Show Cursor | Move mouse (auto-hides after 3 seconds of inactivity) |

---

## Architecture

```text
src/
├── main.rs          # Event loop (winit), application state, hardware diagnostics
├── capture.rs       # Platform video capture (DirectShow on Windows, V4L2 on Linux via FFmpeg)
├── render.rs        # wgpu rendering pipeline, WGSL color conversion shaders, letterboxing
├── devices.rs       # Cross-platform device enumeration (DirectShow/V4L2, WASAPI/PipeWire/ALSA)
├── settings.rs      # JSON configuration persistence with backward-compatible defaults
├── ui.rs            # egui in-game menu, audio status indicators, volume/mute controls
├── logger.rs        # Rotating daily file logging via tracing
├── audio/
│   ├── mod.rs       # AudioPassthrough coordinator with independent in/out streams
│   ├── common.rs    # SPSC lockless ring buffer, Catmull-Rom cubic resampler, channel conversion
│   ├── windows.rs   # Windows WASAPI format negotiation and device resolution
│   └── linux.rs     # Linux PipeWire / ALSA format negotiation and device resolution
├── dx12_interop.rs  # Windows DX12 shared buffers for CUDA zero-copy (optional feature)
└── gpu_decode.rs    # NVIDIA nvJPEG hardware MJPEG decoding (optional feature)
```

---

## Building from Source

Detailed build and dependency instructions for Windows and Linux can be found in [BUILD.md](BUILD.md).

```bash
# Debug build & test suite
cargo test
cargo build

# Optimized release binary
cargo build --release
```

---

## Pushing to Your GitHub Fork & Publishing Releases

To push your work to your GitHub repository ([`ajmine03/TackleCast`](https://github.com/ajmine03/TackleCast)):

### 1. Push Code to `main`
```bash
# Push with personal access token or SSH
git push origin main
```
*Note: If prompted for credentials, use your GitHub username and a Personal Access Token (PAT) with `repo` scope.*

If you use SSH:
```bash
git remote set-url origin git@github.com:ajmine03/TackleCast.git
git push origin main
```

### 2. Publish an Automated GitHub Release
When you are ready to cut a new release, tag your commit and push the tag. The GitHub Actions release workflow will automatically build both Windows `.exe` (`.zip`) and Linux `.tar.gz` bundles and attach them to a new GitHub Release:
```bash
# Create release tag (e.g. v2.2.0)
git tag v2.2.0

# Push tag to trigger automated build & release
git push origin v2.2.0
```

---

## License

MIT License. See [LICENSE](LICENSE) for details.
