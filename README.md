# 🎯 Live Queue Repeat Visitor Monitor

A standalone, privacy-first, client-side web application designed to identify and flag repeat visitors in queues using real-time facial recognition. The system runs directly in the browser using WebAssembly and TensorFlow.js, requiring no backend servers or database setup.

---

## 🚀 Features

- **Edge Facial Recognition**: Computes 128-dimensional facial descriptors entirely inside the client browser via `@vladmandic/face-api`.
- **Biometric Matching**: Uses Euclidean vector distance matching with customizable thresholding to identify returning individuals.
- **Debounce & Cooldown Logic**: Distinguishes between someone standing continuously in front of the lens versus re-entering the line after leaving.
- **Industrial Siren Audio Alert**: High-urgency, dual-tone alternating sound generated via the native Web Audio API (no external MP3 downloads required).
- **Mobile Back-Camera Default**: Automatically requests the device's rear/environment camera (`facingMode: "environment"`).
- **Hardware Flash / Torch Control**: On supported mobile browsers (Chrome / Android), directly control the physical camera LED flash.
- **Zero Cloud Storage**: Operates entirely in RAM; no facial images or biometrics leave the local device.

---

## 🛠️ Prerequisites

To run this application on desktop or mobile, you need:
1. A modern web browser supporting WebRTC (Google Chrome, Safari, Firefox, Edge).
2. A secure origin (**`localhost`** or **HTTPS**). Mobile operating systems (iOS and Android) block access to camera hardware over unencrypted `http://` connections.

---

## 📦 Quick Start

### 1. Local Desktop Testing

You can serve the `index.html` file using any lightweight HTTP server:

#### Using Python:
```bash
# Navigate to the folder containing index.html
python3 -m http.server 8000
```
Open [http://localhost:8000](http://localhost:8000) in Google Chrome or Edge.

#### Using Node.js:
```bash
npx serve .
```

---

### 2. Running on Mobile (Rear Camera & Torch)

Because mobile browsers require an **HTTPS** context to grant access to the camera and physical torch hardware, choose one of the following methods:

#### Method A: Instant HTTPS Tunnel (Recommended)
1. Start your local server:
   ```bash
   python3 -m http.server 8000
   ```
2. In a second terminal window, run a secure tunnel:
   ```bash
   npx localtunnel --port 8000
   # or
   ngrok http 8000
   ```
3. Open the resulting `https://...` link on your phone's browser.

#### Method B: Chrome Remote USB Debugging (Android)
1. Connect your Android phone to your PC via USB and enable USB Debugging.
2. Open `chrome://inspect/#devices` on desktop Chrome.
3. Add port forwarding: `8000` $\rightarrow$ `localhost:8000`.
4. Open `http://localhost:8000` directly on Chrome on your phone.

---

## ⚙️ Configuration Parameters

You can fine-tune detection sensitivity and queue timing directly inside `index.html`:

```javascript
// Sensitivity threshold for face descriptor matching.
// Lower = stricter match (fewer false positives).
// Higher = looser match (handles angles/lighting variations better).
const MATCH_THRESHOLD = 0.55; 

// The time window (in seconds) during which a return is flagged as a violation.
const REPEAT_WINDOW_SEC = 60;  

// Debounce interval (in seconds).
// If the same face stays in frame continuously under this duration,
// it is marked as 'In Queue' rather than a repeated entry.
const DEBOUNCE_SEC = 5;        
```

---

## 🔍 How the Algorithm Works

```
                     [Live Camera Frame]
                              │
                     Detect Face Landmarking
                              │
               Extract 128-D Vector Embedding
                              │
             Compare against Active Memory Cache
                              │
                 ┌────────────┴────────────┐
          Match Found?              No Match (New Face)
                 │                         │
      ┌──────────┴──────────┐       Register ID & Timestamp
   Within                Exceeds           │
 Debounce?           Cooldown Window?   Display Green Box
   ┌──┴──┐               ┌──┴──┐
 YES     NO            YES     NO
  │      │              │      │
Ignore  Trigger     Permit   Trigger
(Same   Alert       Entry    Alert
 Pass)  (Siren)              (Siren)
```

1. **Detection**: Captures frames every 300ms using the `TinyFaceDetector`.
2. **Feature Extraction**: Generates a normalized 128-float biometric vector describing the spatial relationships of facial landmarks.
3. **Distance Calculation**:
   $$\text{distance} = \sqrt{\sum_{i=1}^{128} (A_i - B_i)^2}$$
   If $\text{distance} < \text{MATCH\_THRESHOLD}$, the face matches a known profile.
4. **Time Validation**: Compares `Date.now()` with the last recorded sighting to determine whether to ignore (debounced), alert (repeat violator), or log as a normal pass.

---

## 📈 Scalability & Capacity

- **Memory Consumption**: Each registered visitor takes roughly **512 bytes** of memory. Registering 1,000 visitors requires under **1 MB** of RAM.
- **Matching Speed**: Searches are linear ($O(N)$). Up to ~1,500 visitors will run with zero perceptible latency on modern smartphone processors.
- **Data Lifecycle**: Stored in volatile client memory (`visitors = []`). Refreshing the browser resets the counter and history.

---

## 🛡️ Privacy & Compliance Notice

- **No Images Stored**: The software does not capture, store, or transmit photographic stills or video recordings of visitors.
- **Biometric Vectors**: Only mathematical floating-point representations (vectors) are retained in RAM for the active session.
- Please verify compliance with your local biometric data privacy laws (e.g., GDPR, CCPA, BIPA) before deploying in public surveillance environments.