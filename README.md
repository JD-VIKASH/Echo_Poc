# ECHO — Android APK Proof of Concept (v1.0.0-poc)

ECHO is a personalized hearing-health platform designed for preliminary hearing screening, real-world sound exposure monitoring, and personalized listening protection.

This repository contains the complete native Android Proof of Concept (POC) implementation, built with **Kotlin**, **Jetpack Compose**, and low-level **Android AudioTrack / AudioRecord** subsystems.

---

## 📦 Deliverables Summary

| Artifact | Location | Size / Status |
| :--- | :--- | :--- |
| **Debug Android APK** | `echo-screening-v1.0.0-debug.apk`<br>`app/build/outputs/apk/debug/app-debug.apk` | **16.5 MB (16,566,077 bytes)**<br>Compiled & Verified |
| **Architecture Specification** | [ARCHITECTURE.md](file:///d:/ECHO/echo-poc/ARCHITECTURE.md) | Complete module & state machine specs |
| **Device Capability Matrix** | [DEVICE_CAPABILITY_MATRIX.md](file:///d:/ECHO/echo-poc/DEVICE_CAPABILITY_MATRIX.md) | Output routes, microphones, lifecycle & unknowns |
| **Validation Plan** | [VALIDATION_PLAN.md](file:///d:/ECHO/echo-poc/VALIDATION_PLAN.md) | Artificial ear calibration & clinical study protocol |
| **Automated Unit Tests** | `app/src/test/java/com/echo/hearing/` | **26 Tests Executed — 100% Passed** |

---

## 🚀 Quick Install & Run on Android Device

### Prerequisites
- A compatible Android phone running **Android 8.0 (Oreo, API 26)** or higher (compiled for **Android 15, API 35**).
- Wired 3.5mm headphones, USB-C headset, or Bluetooth earphones.
- (Optional) `adb` installed if side-loading from a PC.

### Installation via ADB
```bash
adb install -r echo-screening-v1.0.0-debug.apk
```

### Direct Download & Sideload
1. Transfer `echo-screening-v1.0.0-debug.apk` to your Android device (via USB cable, Google Drive, or local storage).
2. Open the file on your device and tap **Install** (enable "Install unknown apps" if prompted).
3. Launch **ECHO** from your app drawer.

---

## 🛠️ Building & Testing from Source

### Prerequisites
- JDK 17 or JDK 21 (Microsoft OpenJDK 21 or Eclipse Temurin).
- Android SDK with platform `android-35` and build-tools `35.0.1+`.

### Build Debug APK
```bash
# On Windows PowerShell:
$env:JAVA_HOME = "C:\Program Files\Microsoft\jdk-21.0.12.101-hotspot"
.\gradlew.bat assembleDebug
```
The output APK is generated at:
`app/build/outputs/apk/debug/app-debug.apk`

### Run Automated Tests
```bash
# Run all native Android unit test suites:
.\gradlew.bat testDebugUnitTest

# Run Node.js domain regression tests:
npm test
```

---

## ✨ Implemented Features

### Module A — Accessible Mobile Foundation & User Flow
- **Welcome & Informed Consent**: Clear clinical disclaimers distinguishing screening from diagnosis; explicit user consent gate.
- **Headphone Check**: Dynamic detection of wired headphones, USB headsets, Bluetooth A2DP, and speaker.
- **Test Instructions**: Visual guide for pure tone response protocol.
- **Progressive Test Flow**: Left/right ear indicator pills, frequency targets (500Hz, 1000Hz, 2000Hz, 4000Hz), trial counters.
- **Accessible Response Button**: High-contrast, large touch-target "I HEAR IT" button.
- **Results Screen**: Frequency-by-frequency hearing threshold summary, audio quality audit flags, audiological guidance.

### Module B — Typed Session State Machine
- Strict Finite State Machine enforcing valid transitions:
  `NOT_STARTED` ➔ `ONBOARDING_CONSENT` ➔ `DEVICE_READINESS` ➔ `CALIBRATION_GATE` ➔ `ENVIRONMENTAL_NOISE_CHECK` ➔ `TEST_PREP` ➔ `STIMULUS_PRESENTATION` ➔ `WAITING_FOR_RESPONSE` ➔ `TRIAL_VALIDATION` ➔ `AUTOMATIC_PAUSE_NOISE` ➔ `RECOVERY_RECHECK` ➔ `RESULTS`.
- Rejection of illegal state transitions with typed error tracking.

### Module C — Audio Route & Calibration Gate
- Inspects `AudioManager.getDevices(GET_DEVICES_OUTPUTS)` to identify active route.
- Blocks built-in phone speaker from test execution (ear isolation requirement).
- Explicit provenance registry marking hardware as `UNCALIBRATED`, `FIXTURE_DEFAULT`, or `SIMULATED_PROFILE`.
- Prevents uncalibrated hardware from presenting misleading clinical claims.

### Module D — Audio Stimulus Engine
- Pure-tone synthesis using Android `AudioTrack` (16-bit PCM at 44.1 kHz).
- **Hann / Raised-Cosine Envelope**: 20ms onset and offset smoothing to eliminate switching clicks and spectral splatter.
- **Pure Channel Isolation**: Pure left-channel output for `Ear.LEFT` and pure right-channel output for `Ear.RIGHT`.
- Safe digital attenuation range (-80 dBFS to 0 dBFS).
- Immediate audio halt on user pause, app backgrounding, or disturbance.

### Module E — Modified Hughson-Westlake Threshold Engine
- Decoupled, pure-Kotlin staircase algorithm:
  - Down 10 dB on response (`HEARD`).
  - Up 5 dB on non-response (`NOT_HEARD` or timeout).
  - Threshold acceptance upon **2 out of 3 ascending responses** at the same stimulus level.
- Maximum trial limit safeguards (18 trials) to prevent user fatigue.
- Independent tracking and state isolation per ear and test frequency.

### Module F — Environmental Noise & Automatic Disturbance Handling
- Real-time microphone monitoring using Android `AudioRecord`.
- Computes RMS power every 100ms and converts to dBFS.
- **Automatic Pause**: Immediately halts the test if room noise exceeds acceptable threshold during a stimulus.
- **Trial Contamination**: Disturbed trials are marked `CONTAMINATED_BY_NOISE` and retried; never treated as non-response.
- **Quiet Recovery Window**: Requires 3 consecutive seconds of continuous quiet before allowing test resumption.

### Module G — Local Persistence & Privacy
- Offline-first local storage (`context.filesDir/echo_sessions/`) storing structured session JSON records.
- **Strict Privacy**: Zero microphone audio recording or retention; only RMS energy is analyzed transiently in memory.
- Zero server communication or cloud dependency.

### Developer Simulation Panel
- Collapsible dev panel accessible via the **DEV SIM** chip in the app bar.
- Allows live testing of:
  - Injecting ambient noise spikes (`INVALID` / `SUSPECT`).
  - Clearing noise to trigger the 3-second quiet recovery countdown.
  - Simulating headphone unplug events (`AUTOMATIC_PAUSE_DEVICE`).
  - Simulating user response events.

---

## 🧪 Automated Test Results Summary

### Android Unit Tests (`./gradlew.bat testDebugUnitTest`):
- `HughsonWestlakeProtocolTest`: **7 of 7 PASSED**
  - Contaminated trial by noise does not advance level or count as non-response
  - Hughson-Westlake threshold accepted after 2 ascending responses
  - Ear and frequency isolation verified across full protocol
  - Moves down 10 dB on heard and up 5 dB on not heard
  - Max trials guard prevents infinite search
  - Starts at configured starting level and first target
  - Invalid interrupted trial does not alter threshold search
- `NoiseAndDisturbanceTest`: **3 of 3 PASSED**
  - Fake noise monitor injects states accurately
  - Recovery check fails when noise is invalid or quiet window is insufficient
  - Unavailable sensor reported when microphone permission missing
- `SessionRepositoryAndPrivacyTest`: **3 of 3 PASSED**
  - In-memory repository stores and retrieves sessions without loss
  - Privacy guarantee: zero raw audio stored in session model
  - Simulated sessions carry simulation flag explicitly
- `SessionStateMachineTest`: **9 of 9 PASSED**
  - Starts in `NOT_STARTED` state
  - Valid onboarding and consent transitions
  - Declined consent aborts session
  - Unsupported audio route aborts session
  - Supported route proceeds through calibration and noise precheck
  - Noisy environment causes automatic pause
  - Illegal transitions are rejected with `IllegalStateException`
  - Cancellation allowed from active states
  - Restart resets state to `NOT_STARTED`
- `StimulusEngineTest`: **4 of 4 PASSED**
  - Left ear PCM has signal only in left channel
  - Right ear PCM has signal only in right channel
  - Raised-cosine envelope ramps up smoothly from zero
  - Fake stimulus engine tracks tone and stop behavior

### Pre-existing Node.js Tests (`npm test`):
- **8 of 8 PASSED**

---

## ⚠️ Known Limitations & Next Steps

1. **Acoustic Calibration**:
   - The current POC output is calibrated in digital attenuation (**dBFS**), not physical sound pressure level (**dB SPL**) or clinical hearing level (**dB HL**).
   - *Next Step*: Follow [VALIDATION_PLAN.md](file:///d:/ECHO/echo-poc/VALIDATION_PLAN.md) to measure output voltages and SPL in an IEC 60318-1/4 artificial ear coupler for targeted headphone models.
2. **Microphone Directivity & OEM Variance**:
   - Android microphone frequency response varies between device manufacturers.
   - *Next Step*: Benchmark microphone sensitivity using a calibrated acoustic pink noise source against a Type-1 sound level meter.
3. **Cross-Platform iOS Parity**:
   - The core state machine and staircase protocols are written in standard Kotlin and ready for Kotlin Multiplatform (KMP) sharing or direct Swift porting.
