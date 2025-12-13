![Node-RED](https://img.shields.io/badge/Node--RED-Flows-red)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

# 📟 Raspberry-Pi-20x4-I2C-LCD-Status-Display-with-Node-RED
Turn a **20x4 I2C LCD** (PCF8574 backpack) into a simple **live status display** using a **Raspberry Pi 4 or 5 & Node-RED**.

This repo includes a ready-to-import Node-RED flow that displays:

- **The current date and time** 🕒 on **Line 1**
- A **title** 📟 on **Line 2** (dynamic)
- A **title** 📟 on **Line 3** (dynamic)
- A **scrolling message reel** 📟 on **Line 4** (dynamic)

Use this as a baseline for other projects like:

- 🌍 Earthquake alerts
- 🌫️ Air quality (AQI)
- 🧠 System status (CPU temp, disk, Wi‑Fi)

---

## 📦 Hardware Requirements

- Raspberry Pi 4 or Raspberry Pi 5 💻
- 20x4 character LCD with **I2C backpack** 📟 (PCF8574 or compatible) 
- Jumper wires 🔌 (female–female)

---

## 🔌 Wiring: Raspberry Pi to 20x4 I2C LCD (I2C)

First, connect the 20x4 LCD display to the Raspberry Pi using the I2C interface.

Use the female-female jumper wires as follows:

- `VCC` → **5V** (pin **4**)
- `GND` → **GND** (pin **6**)
- `SDA` → **GPIO 2 / SDA1** (pin **3**)
- `SCL` → **GPIO 3 / SCL1** (pin **5**)

### Wiring Diagram
![Raspberry Pi 4 to 20x4 I2C LCD wiring](screenshots/Raspberry%20Pi%204%20%26%20I2C%2020x4%20LCD%20Wiring%20Diagram.png)

> ⚠️ Double-check your wiring before powering on.

Once the display is connected, you can fine-tune **brightness/contrast** using the potentiometers on the back of the display.

---

## 🧑‍💻 Enable I2C on the Raspberry Pi

1. Open Raspberry Pi configuration:

```bash
sudo raspi-config
```

2. Go to:

- `Interface Options` → `I2C` → `Enable`

3. Reboot:

```bash
sudo reboot
```

### 🧑‍💻 Confirm the LCD shows up on I2C

Install I2C tools:

```bash
sudo apt-get update
sudo apt-get install -y i2c-tools
```

Scan the I2C bus:

```bash
sudo i2cdetect -y 1
```

You should see a hex address such as `0x27` or `0x3F`.

---

## 🧑‍💻 Install Node-RED and the LCD Node

### 🛠 Install Node-RED (if needed)

```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

### 🛠 Install the LCD node

1. Open Node-RED in your browser:

- `http://<raspberrypi-ip>:1880`

2. Install the palette module:

- Menu (☰) → `Manage palette` → `Install`
- Search for: `node-red-contrib-pcf8574-lcd`
- Click `Install`

This node handles the PCF8574 I2C interface to your 20x4 LCD.

---

## 🛠 Import the Flow

1. In Node-RED, go to:

- Menu (☰) → `Import` → `Clipboard`

2. Paste the JSON from this repo:

- [`LCD 20x4 Status Display.json`](./LCD%2020x4%20Status%20Display.json)

3. Click `Import`

### Node-RED Flow
![Node-RED flow for 20x4 I2C LCD](screenshots/I2C%20LCD%2020x4%20Node-Red%20Flow.png)

---

### 📟 Configure the LCD node

Open the `LCD-I2C` node and set:

- Variant: `PCF8574`
- Size: `20x4`
- Address: your detected address from `i2cdetect` (example: `0x27`)

### 🧯 Error Handling (Catch Node + LCD Errors)

This flow includes a **Catch** node wired to a **Debug** node so you can see runtime issues (I2C hiccups, bad payloads, misconfiguration, etc.) without guessing.

- **Catch** captures runtime errors from the flow (especially the `LCD-I2C` node).
- **Debug: `LCD errors`** prints them in the Debug sidebar.

**If `LCD errors` shows nothing:** open the Catch node and set scope to **All nodes** (or select `LCD-I2C`), then **Deploy**.

4. Click `Deploy`.
---

## 📟 What You Should See on the LCD

- **Line 1**: Date/time (updates continuously)
- **Line 2**: Title (default: `Raspberry Pi LCD`)
- **Line 3**: Status (default: `Running on Node-RED`)
- **Line 4**: Scrolling ticker (default: `Made by The Dyslexic Coder!`)

---

## How the `Format for LCD` Function Works

The core of this project is a single Node-RED **Function** node that renders the display.

### 📟 Key behavior

- Keeps a config object in **node context**:
  - `lcdConfig.line2`
  - `lcdConfig.line3`
  - `lcdConfig.ticker`
- Keeps the ticker scroll position in context:
  - `context.get("offset")`
- Sanitizes text so the LCD only receives **ASCII-safe** characters (prevents weird symbols on HD44780 screens).

---

## 🧰 Update the Text Dynamically (from other flows)

To update the LCD without editing the renderer code, send a message into the same `Format for LCD` node with any of:

- `msg.lcdLine2` (Line 2)
- `msg.lcdLine3` (Line 3)
- `msg.lcdTicker` (Line 4 ticker)

Example Function node you can wire into `Format for LCD`:

```js
msg.lcdLine2  = "Earthquake Monitor";
msg.lcdLine3  = "All quiet";
msg.lcdTicker = "No quakes > M4.0 in the last hour.";
return msg;
```

> Tip: Keep ticker text simple (ASCII). Avoid emojis and special bullets like `•`.

---

## 🧰 Optional: Turn the Display On/Off by Time of Day

Inside the `Format for LCD` function:

```js
const USE_DISPLAY_SCHEDULE = false;
const OFF_START_HOUR = 23;
const OFF_END_HOUR   = 7;
```

If you set `USE_DISPLAY_SCHEDULE = true`, the flow will send:

- `msg.action = "off"` during quiet hours
- `msg.action = "on"` outside quiet hours

This controls the display state (it does **not** dim backlight brightness).

---

## 🧰 Notes and Tips

- HD44780 character LCDs work best with **plain ASCII**.
  - Use letters, numbers, and basic punctuation.
  - Avoid emojis, fancy bullets (`•`), and smart quotes.
- If you see random symbols:
  - Confirm your I2C address in `i2cdetect`
  - Re-check wiring (`SDA`/`SCL` swapped is a common issue)
  - Keep text ASCII-only
  - Redeploy Node-Red
- For smoother ticker scrolling:
  - Set your Inject interval to **0.3–0.5 seconds**
  - For less I2C traffic, use **1 second**

---

## 📄 License

This project is open-source.

If using MIT License, create a LICENSE file containing:

```text
MIT License
