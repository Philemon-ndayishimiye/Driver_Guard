# Drive Guard  Driver Drowsiness Detection with Edge Impulse

**Drive Guard** watches the driver's face through a camera and detects in real time whether the driver is **Normal** (awake and alert) or **Drowsy** (eyes closing, falling asleep). When drowsiness is detected, it **sounds an alarm** in the browser and **sends a signal to an Arduino**, which can trigger a buzzer, LED or vibration motor to wake the driver.

The AI model was built and trained on **[Edge Impulse](https://edgeimpulse.com)** and runs **fully on the device** through WebAssembly. No video leaves the computer and no internet is needed while driving.

---

## 1. The Problem

Driver fatigue is one of the major hidden causes of road accidents. A tired driver:

- has slower reaction times,
- can fall into **micro-sleeps** of a few seconds, which at 60 km/h means travelling dozens of metres with no control of the vehicle,
- often does not realise how tired they are until it is too late.

Long-distance bus, truck, taxi and moto drivers are especially at risk because they drive for many hours, at night, and on long routes between cities.

Most vehicles have **no system that watches the driver's alertness**. Commercial driver-monitoring systems exist only in expensive new cars. The drivers who need them most, in older buses, trucks and private cars, cannot afford them.

## 2. Project Aim

Drive Guard aims to provide a **low-cost, offline, real-time drowsiness alert system** that can be added to any vehicle using only a camera, a small computer or laptop, and an Arduino.

Objectives:

1. Collect and label images of drivers in **Normal** and **Drowsy** states.
2. Train a lightweight image classification model on **Edge Impulse** that runs on low-power devices.
3. Deploy the model as **WebAssembly** so it runs in a web browser and in Node.js **without internet**.
4. Build a live monitoring interface that classifies the camera feed **every second**.
5. Warn the driver immediately with an **audible alarm** and a signal to an **Arduino** for a physical alert.

## 3. Methodology

### 3.1 How the system works

```
 Camera ──► Browser (drowsy-monitor.html)
               │
               │ 1. Take a frame every 1 second
               │ 2. Resize to 96 × 96 pixels
               │ 3. Run the Edge Impulse model (WebAssembly, on-device)
               ▼
        Result: Normal / Drowsy + confidence
               │
      ┌────────┴─────────┐
      ▼                  ▼
 Screen + beep     Arduino (USB, Web Serial, 9600 baud)
 alarm if Drowsy   receives "drowsy" or "normal"
                   → buzzer / LED / vibration
```

### 3.2 The model (Edge Impulse)

| Property | Value |
|---|---|
| Edge Impulse project |  |
| Deployed version | 4 (Impulse #4) |
| Model type | Image classification |
| Input | 96 × 96 pixels, **grayscale** (9,216 features) |
| Classes | `Drowsy`, `Normal` |
| Classification threshold | 0.60 |
| Deployment | WebAssembly (browser + Node.js) |

### 3.3 Training pipeline

1. **Data collection:** face images of drivers labelled `Normal` (eyes open, alert) and `Drowsy` (eyes closed or closing, head dropping, yawning).
2. **Pre-processing:** images resized to 96 × 96 and converted to grayscale. This makes the model small and fast, and less sensitive to lighting colour.
3. **Learning block:** a neural network image classifier trained on the extracted features.
4. **Testing:** the model was evaluated on a separate test set in Edge Impulse (Model testing).
5. **Deployment:** the model was exported as WebAssembly, which gives the two folders in this repository.

### 3.4 Results

| Metric | Value |
|---|---|
| Training accuracy | _add from Edge Impulse → Classifier page_ |
| Test accuracy | _add from Edge Impulse → Model testing_ |
| Inference time | _add from Edge Impulse → Classifier page (on-device performance)_ |

## 4. Project Structure

The project has **two folders**, one for each way of running the model:

```
Drive Guard/
├── README.md                         ← this file
├── .gitignore
│
├── browser/                          ← MAIN APP: live camera monitoring in the browser
│   ├── drowsy-monitor.html           # Live drowsiness monitor (camera + alarm + Arduino)
│   ├── index.html                    # Simple test page: paste features → get prediction
│   ├── run-impulse.js                # EdgeImpulseClassifier class (loads and runs the model)
│   ├── edge-impulse-standalone.js    # Edge Impulse WebAssembly loader
│   ├── edge-impulse-standalone.wasm  # The trained model ( must be copied here, see Step 2)
│   ├── server.py                     # Small local web server (port 8082)
│   └── README.md                     # Original Edge Impulse notes
│
└── node/                             ← Command-line testing with Node.js
    ├── run-impulse.js                # Classify raw features from the terminal
    ├── edge-impulse-standalone.js    # Edge Impulse WebAssembly loader
    ├── edge-impulse-standalone.wasm  # The trained model
    └── README.md                     # Original Edge Impulse notes
```

| Folder | What it is for |
|---|---|
| `browser/` | The real application. It opens the camera, detects drowsiness live, beeps and talks to the Arduino. |
| `node/` | Testing and integration. It runs the same model from the terminal on a list of features, which is useful to check that the model works or to plug it into a backend. |

---

## 5. Part A Building the Model on Edge Impulse (Step by Step)

Follow these steps to recreate or improve the model. If you only want to run the project, skip to **Part B**; the trained model is already included.

### Step 1 Create an account and a project
1. Go to **https://studio.edgeimpulse.com** and sign up (free).
2. Click **Create new project** and give it a name, e.g. `driver-fatigue-demo`.
3. Choose **Images** as the project type and **Classify a single object** (image classification).

### Step 2 Collect the data
1. Open **Data acquisition**.
2. Add images in either of two ways:
   - **Upload data:** upload face photos you already have, or a public drowsiness dataset.
   - **Connect a device:** use your phone or laptop camera (scan the QR code from **Devices → Connect a new device**) and take photos directly.
3. Give every image one of two labels:
   - `Normal`  eyes open, looking at the road
   - `Drowsy`  eyes closed or half-closed, yawning, head dropping
4. Keep the two classes **balanced** (similar number of images) and include different people, lighting (day and night), glasses and head angles.
5. Make sure the data is split into **Training** and **Test** sets (about 80% / 20%). Edge Impulse can do this automatically in **Dashboard → Perform train/test split**.

### Step 3 Create the impulse
1. Open **Impulse design  Create impulse**.
2. **Image data** block: set the image size to **96 × 96** and resize mode to *Fit shortest axis*.
3. **Add a processing block:** choose **Image**.
4. **Add a learning block:** choose **Transfer Learning (Images)** or **Classification**.
5. Click **Save Impulse**.

### Step 4 Generate features
1. Open **Image** (the processing block).
2. Set **Color depth** to **Grayscale** and click **Save parameters**.
3. Click **Generate features**. The *Feature explorer* shows how well `Normal` and `Drowsy` are separated.

### Step 5 Train the model
1. Open the learning block (**Transfer learning** / **Classifier**).
2. Set, for example, **Number of training cycles: 20–50**, **Learning rate: 0.0005**, and enable **Data augmentation**.
3. Click **Save & train**.
4. Check the **accuracy**, the **confusion matrix** and the **on-device performance** (inference time and memory).

### Step 6 Test the model
1. Open **Model testing** and click **Classify all**. This checks the model on images it has never seen.
2. Optionally, open **Live classification** to try the model with your own camera.

### Step 7 Deploy as WebAssembly
1. Open **Deployment**.
2. Search for **WebAssembly (browser)** and click **Build**. A `.zip` downloads → this is the `browser/` folder.
3. Search for **WebAssembly (Node.js)** and click **Build**. A `.zip` downloads → this is the `node/` folder.
4. Unzip them into the project. Then copy your custom `drowsy-monitor.html` into the new `browser/` folder.

> Every time you retrain, the deployed version number increases. The app shows the loaded version in the System Log, e.g. `Loaded model: philos/driver-fatigue-demo v4`.

---

## 6. Part B — Running the Project (Step by Step)

### Step 1 — Install the prerequisites

| Tool | Needed for | Download |
|---|---|---|
| **Python 3** | Local web server for the browser app | https://www.python.org |
| **Node.js 16+** | Command-line testing (`node/` folder) | https://nodejs.org |
| **Google Chrome** or **Microsoft Edge** | The live monitor (camera + Web Serial for Arduino) | — |
| Webcam | Live monitoring (built-in, USB, or phone used as a webcam) | — |
| Arduino (optional) | Physical alarm | https://www.arduino.cc/en/software |

Check your installations:

```bash
python --version
node --version
```

### Step 2 Copy the model file into the `browser` folder

The browser app needs **`edge-impulse-standalone.wasm`**, but it is only inside the `node/` folder. Both folders come from the same model version, so copy it:

**Windows (PowerShell):**
```powershell
Copy-Item "node\edge-impulse-standalone.wasm" "browser\"
```

**macOS / Linux:**
```bash
cp node/edge-impulse-standalone.wasm browser/
```

Without this file, the page stays on **"Loading model…"**.

### Step 3 Start the local web server

```bash
cd browser
python server.py
```

(On macOS / Linux use `python3 server.py`.) You should see:

```
Running at http://localhost:8082
```

Keep this terminal open. The page must be opened through this server; double-clicking the HTML file will not work, because browsers block WebAssembly and the camera for local files.

### Step 4 Open the live drowsiness monitor

1. In **Chrome** or **Edge**, go to: **http://localhost:8082/drowsy-monitor.html**
2. Click **Allow** when the browser asks for camera permission.
3. The System Log shows `Loaded model: philos/driver-fatigue-demo v4`.
4. Sit in front of the camera:
   - Eyes open → status **Normal** (green ring)
   - Close your eyes for a few seconds → status **Drowsy** (red, flashing) and a **beep** plays
5. The ring around the camera shows the model's confidence.

### Step 5 Connect the Arduino (optional)

1. Upload a sketch to the Arduino that reads text from the serial port at **9600 baud**. The page sends `drowsy` or `normal` followed by a new line, **only when the state changes**. Example:

   ```cpp
   const int BUZZER = 8;
   const int LED = 13;

   void setup() {
     Serial.begin(9600);
     pinMode(BUZZER, OUTPUT);
     pinMode(LED, OUTPUT);
   }

   void loop() {
     if (Serial.available()) {
       String msg = Serial.readStringUntil('\n');
       msg.trim();
       if (msg == "drowsy") {
         digitalWrite(LED, HIGH);
         tone(BUZZER, 1000);
       } else if (msg == "normal") {
         digitalWrite(LED, LOW);
         noTone(BUZZER);
       }
     }
   }
   ```

2. **Close the Arduino IDE's Serial Monitor.** Only one program can use the port at a time.
3. On the monitor page, click **Connect to Arduino** and choose the Arduino's COM port.
4. The button turns green (**Connected**), and the log shows `Sent to Arduino: Drowsy` / `Normal`.

### Step 6 Test the model with the simple page (optional)

1. Open **http://localhost:8082/index.html**
2. In Edge Impulse Studio, open **Model testing** (or **Live classification**), pick a sample, and copy its **Raw features**.
3. Paste them into the box and click **Run inference**. The result shows the scores for `Drowsy` and `Normal`.

### Step 7 Test the model from the terminal with Node.js (optional)

```bash
cd node
node run-impulse.js "0x5a5a5a, 0x5b5b5b, 0x5c5c5c, ..."
```

Replace the list with the **9,216 raw features** copied from Edge Impulse. Because the list is long, it is easier to save it in a text file and pass the file name:

```bash
node run-impulse.js features.txt
```

Example output:

```
Running inference for philos / driver-fatigue-demo (version 4)
{ anomaly: 0, results: [ { label: 'Drowsy', value: 0.07 }, { label: 'Normal', value: 0.93 } ] }
```

---

## 7. Troubleshooting

| Problem | Solution |
|---|---|
| Stuck on **"Loading model…"** | Copy `edge-impulse-standalone.wasm` into `browser/` (Part B, Step 2), and open the page through `http://localhost:8082`. |
| Camera does not show / no predictions | In `drowsy-monitor.html`, the camera element must be a **video** element: replace `<img id="webcam" alt="phone camera feed">` with `<video id="webcam" autoplay playsinline muted></video>`. An `<img>` cannot display a camera stream. |
| To use a **phone as the camera** | Install a webcam app such as DroidCam or Iriun on the phone and PC. The phone then appears as a normal webcam in the browser. |
| "Connect to Arduino" does nothing | Use **Chrome** or **Edge**; Firefox and Safari do not support Web Serial. |
| Arduino connection fails | Close the Arduino IDE Serial Monitor, check the USB cable, and choose the correct COM port. |
| `Address already in use` on port 8082 | Another server is running. Close it, or change `PORT = 8082` in `server.py`. |
| No beep sound | Click anywhere on the page once; browsers only allow sound after user interaction. |
| Wrong predictions at night | Add more low-light and night images in Edge Impulse and retrain. |

## 8. Future Work

- Run on a small device inside the vehicle, such as a Raspberry Pi or ESP32-CAM, using Edge Impulse for Linux or the Arduino library export.
- Add a **yawning** class and **head-pose** detection.
- Trigger the alarm only after drowsiness lasts **2–3 seconds** in a row, to reduce false alarms from normal blinking.
- Send an SMS or notification to the fleet manager when a driver is repeatedly drowsy.

## 9. Technologies Used

Edge Impulse · TensorFlow Lite · WebAssembly · JavaScript · HTML/CSS · Web Serial API · Web Audio API · Python (local server) · Node.js · Arduino
