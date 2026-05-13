# UlanziDeck Plugin SDK

<p align="start">
   <strong>English</strong> · <a href="./README.zh.md">简体中文</a>
</p>

> Official development kit for **UlanziStudio** programmable macro keypad platform
>
> **SDK Version:** Based on Ulanzi JS Plugin Development Protocol V2.1.2
> **Compatible:** Ulanzi Studio 3.0.11

---

## ⚡ Quick Start

### 1️⃣ Create Plugin Structure

Create a new folder with the naming convention `com.ulanzi.{pluginName}.ulanziPlugin`:

```bash
com.ulanzi.myplugin.ulanziPlugin/
├── manifest.json          # Plugin configuration (required)
├── libs/                  # SDK libraries
├── resources/             # Icons and assets
├── plugin/
│   └── app.js            # Main service entry
└── property-inspector/
    └── inspector.html     # Configuration UI
```

**Naming Convention:**
- Plugin folder: `com.ulanzi.{pluginName}.ulanziPlugin`
- Main service UUID: `com.ulanzi.ulanzistudio.{pluginName}` (4 segments)
- Action UUID: `com.ulanzi.ulanzistudio.{pluginName}.{actionName}` (5+ segments)

---

### 2️⃣ Configure manifest.json

Create `manifest.json` in the plugin root directory:

```json
{
  "Author": "Your Name",
  "Name": "My Plugin",
  "Description": "Plugin description",
  "Icon": "resources/icon.png",
  "Version": "1.0.0",
  "UUID": "com.ulanzi.ulanzistudio.myplugin",
  "Type": "JavaScript",
  "CodePath": "plugin/app.js",
  "Actions": [
    {
      "Name": "My Action",
      "Icon": "resources/action-icon.png",
      "UUID": "com.ulanzi.ulanzistudio.myplugin.action",
      "PropertyInspectorPath": "property-inspector/inspector.html",
      "States": [
        { "Name": "Default", "Image": "resources/action-icon.png" }
      ],
      "Controllers": ["Keypad"]
    }
  ],
  "OS": [
    { "Platform": "windows", "MinimumVersion": "10" },
    { "Platform": "mac", "MinimumVersion": "10.11" }
  ]
}
```

**Required Fields:**
- `UUID` — Plugin unique identifier (exactly 4 segments)
- `CodePath` — Main entry point (`.html` for WebView, `.js` for Node.js)
- `Actions` — Array of action definitions
- `Type` — Fixed value `"JavaScript"`

**Action Fields:**
- `UUID` — Action unique identifier (5+ segments)
- `Controllers` — `["Keypad"]`, `["Encoder"]`, or both
- `Devices` — Target devices (empty array `[]` = all devices)

---

### 3️⃣ Install SDK

Copy the SDK libraries to your plugin project:

```bash
# For HTML Property Inspectors
cp -r common-html/libs ./libs/

# For Node.js Main Services
cp -r common-node ./plugin-common-node/

# Install WebSocket dependency
npm install ws
```

**SDK Structure:**
- `common-html/libs/` — WebSocket client, UI utilities, i18n for Property Inspectors
- `common-node/` — WebSocket server, system APIs for Main Services

---

### 4️⃣ Implement Plugin

Plugins can use either HTML or Node.js as the main service. Choose based on your needs:
- **HTML** — Best for UI-rich plugins, Canvas drawing, simpler logic
- **Node.js** — Best for system integration, file access, complex logic

**HTML Main Service (`plugin/app.html`):**

```html
<!DOCTYPE html>
<html>
<head>
  <script src="../libs/js/constants.js"></script>
  <script src="../libs/js/eventEmitter.js"></script>
  <script src="../libs/js/timers.js"></script>
  <script src="../libs/js/utils.js"></script>
  <script src="../libs/js/ulanziApi.js"></script>
</head>
<body>
  <script>
    $UD.connect('com.ulanzi.ulanzistudio.myplugin');

    $UD.onConnected(() => {
      console.log('Connected to UlanziStudio');
    });

    $UD.onAdd((message) => {
      console.log('Action added:', message.context);
    });

    $UD.onRun((message) => {
      // Main action trigger logic
      console.log('Action triggered:', message.context);
      $UD.toast('Hello from plugin!');
    });
  </script>
</body>
</html>
```

**Node.js Main Service (`plugin/app.js`):**

```js
import UlanziApi, { Utils, RandomPort } from './plugin-common-node/index.js';

const $UD = new UlanziApi();

// Generate random port for Property Inspector connection (Node.js only)
const port = new RandomPort().getPort();

// Connect to UlanziStudio
$UD.connect('com.ulanzi.ulanzistudio.myplugin');

$UD.onConnected(() => {
  console.log('Connected to UlanziStudio');
});

$UD.onAdd((message) => {
  console.log('Action added:', message.context);
});

$UD.onRun((message) => {
  // Main action trigger logic
  console.log('Action triggered:', message.context);
  $UD.toast('Hello from plugin!');
});
```

**Key Differences:**
- HTML uses `<script>` tags to load SDK; Node.js uses ES modules (`import`)
- Node.js requires `RandomPort` to generate WebSocket port for Property Inspector connection
- Both use the same `$UD` API for events and commands

---

### 5️⃣ Implement Property Inspector

Create `property-inspector/inspector.html`:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" href="../../libs/css/uspi.css">
  <script src="../../libs/js/constants.js"></script>
  <script src="../../libs/js/eventEmitter.js"></script>
  <script src="../../libs/js/timers.js"></script>
  <script src="../../libs/js/utils.js"></script>
  <script src="../../libs/js/ulanziApi.js"></script>
</head>
<body>
  <div class="uspi-wrapper">
    <form id="property-inspector">
      <div class="uspi-item">
        <div class="uspi-item-label" data-localize>Message</div>
        <input type="text" class="uspi-item-value" name="message" value="">
      </div>
    </form>
  </div>

  <script>
    $UD.connect('com.ulanzi.ulanzistudio.myplugin.action');

    // Load saved settings when action is added
    $UD.onAdd((message) => {
      Utils.setFormValue(message.param, '#property-inspector');
    });

    // Send settings to main service when form changes
    document.getElementById('property-inspector').addEventListener('change', () => {
      const params = Utils.getFormValue('#property-inspector');
      $UD.sendParamFromPlugin(params);
    });
  </script>
</body>
</html>
```

**Key Points:**
- Wrap content in `.uspi-wrapper` for automatic i18n and styling
- Use `data-localize` attribute for automatic translation
- Use `Utils.getFormValue()` and `Utils.setFormValue()` for form handling

---

### 6️⃣ Add Localization (Optional)

Create localization JSON files in the plugin root directory:

Localization JSON files can include two kinds of entries:

- Top-level manifest fields: use `Name`, `Description`, and an `Actions` array aligned with `manifest.json` to localize how the plugin and its actions appear in the app.
- `Localization` object: use it for Property Inspector or page text. Each key matches the text or attribute value referenced by `data-localize`.

**zh_CN.json:**
```json
{
  "Name": "我的插件",
  "Description": "我的插件描述",
  "Actions": [
    {
      "Name": "插件操作",
      "Tooltip": "执行插件操作"
    }
  ],
  "Localization": {
    "Message": "消息",
    "Save": "保存",
    "Hello": "你好"
  }
}
```

**en.json:**
```json
{
  "Name": "My Plugin",
  "Description": "My plugin description",
  "Actions": [
    {
      "Name": "Plugin Action",
      "Tooltip": "Run plugin action"
    }
  ],
  "Localization": {
    "Message": "Message",
    "Save": "Save",
    "Hello": "Hello"
  }
}
```

**Supported Languages:**
- `zh_CN` — Simplified Chinese
- `zh_HK` — Traditional Chinese (HK)
- `en` — English
- `ja_JP` — Japanese
- `de_DE` — German
- `ko_KR` — Korean
- `pt_PT` — Portuguese
- `es_ES` — Spanish

**Usage in HTML:**
```html
<!-- Automatic translation by element text content -->
<div class="uspi-item-label" data-localize>Message</div>

<!-- Automatic translation by attribute value -->
<option value="save" data-localize="Save">Save</option>

<!-- Manual translation in JavaScript -->
<script>
  const label = $UD.t('Message');  // Returns localized string
</script>
```

---

### 7️⃣ Test with Simulator

The UlanziDeck Simulator allows testing plugins without the desktop application:

```bash
cd UlanziDeckSimulator
npm install
npm start
```

**Testing Steps:**

1. **Copy plugin** — Copy your plugin folder to `UlanziDeckSimulator/plugins/`

2. **Open simulator** — Open http://127.0.0.1:39069 in your browser

3. **Refresh plugin list** — Click "Refresh Plugin List" button to load your plugin

4. **Start main service** — For Node.js plugins, start the main service manually:
   ```bash
   node path/to/your/plugin/app.js
   ```

5. **Add action to key** — Drag your action from the plugin list to a key

6. **Open property inspector** — Click the key to open the configuration UI

7. **Test events** — Use right-click menu to manually trigger events for testing

**Simulator Limitations:**
- `openview` and `openurl` cannot open local files (browser restriction)
- Main service must be started separately for Node.js plugins
- Actions are not auto-loaded by default

---

### 8️⃣ Debug in Desktop App

Launch UlanziStudio with debugging flags enabled:

**Available Flags:**

| Flag | Description |
|------|-------------|
| `--log` | Write logs to file |
| `--logLevel` | Set log verbosity |
| `--webRemoteDebug` | Enable HTML plugin debugging at port 9292 |
| `--nodeRemoteDebug` | Enable Node.js plugin debugging |

**Windows:**

Right-click UlanziStudio shortcut → Properties → append to **Target**:

```
"C:\...\Ulanzi Studio.exe" --log --webRemoteDebug --nodeRemoteDebug
```

**macOS:**

```bash
open /Applications/Ulanzi\ Studio.app --args --log --webRemoteDebug
```

**Access Debug Tools:**

- **HTML Plugins:** Open `localhost:9292` in browser to debug all loaded HTML plugins
- **Node.js Plugins:** Open `chrome://inspect` in Chrome, find your plugin and click "inspect"

**Important Notes:**
- Using `open` command on macOS may prevent Accessibility permissions, affecting hotkey functionality
- Use `./UlanziStudio` directly if hotkeys are not working

---

## 📦 SDK Overview

| Package | Purpose |
|---------|---------|
| [`common-html`](./common-html/README.md) | HTML plugins (WebView) — WebSocket client, UI utilities, i18n, Canvas helpers |
| [`common-node`](./common-node/README.md) | Node.js plugins — WebSocket server, system APIs, file system access |
| [`UlanziDeckSimulator`](./UlanziDeckSimulator/README.md) | Browser simulator for testing |
| [`demo`](./demo/) | Example plugins |

---

## 🏗️ Architecture

### Main Service Types

Plugins can use either HTML or Node.js as the main service:

| Type | Entry | Loading | SDK | Use Case |
|------|-------|---------|-----|----------|
| **HTML Plugin** | `.html` | WebView | `common-html` | UI-rich plugins, Canvas drawing, simple logic |
| **Node.js Plugin** | `.js` | Node.js v20 | `common-node` | System integration, file access, complex logic |

**Property Inspector** (optional) is always HTML using `common-html`, opened when user configures an action.

### Communication Flow

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│   Property      │  WebSocket│   UlanziStudio │  WebSocket│   Main          │
│   Inspector     │ ←──────→│     (Bridge)     │ ←──────→│   Service       │
│   (HTML)        │         │                  │         │   (Node.js/HTML)│
└─────────────────┘         └──────────────────┘         └─────────────────┘
```

**Plugin Lifecycle:**
- **Main Service** — Long-running background process (Node.js or HTML)
- **Property Inspector** — HTML page created when user configures a key, destroyed on navigate away

---

## 🔑 Key Concepts

### UUID Rules

```
Main Service:  com.ulanzi.ulanzistudio.{plugin}           → 4 segments
Action:        com.ulanzi.ulanzistudio.{plugin}.{action}  → 5+ segments
Package:       com.ulanzi.{plugin}.ulanziPlugin
```

### Context Parameter

Each key instance has a unique `context` string:

```js
// Format: uuid + '___' + key + '___' + actionid
const context = $UD.encodeContext(msg)     // Encode to context string
const decoded = $UD.decodeContext(context) // Decode → { uuid, key, actionid }
```

**Important:** The same action can be assigned to multiple keys. The `context` parameter uniquely identifies each instance.

---

## 🎮 Devices & Controllers

### Supported Devices

- **D200**
- **D200H**
- **Dial**
- **D200X**

### Controller Types

Actions specify which controller types they support:

| Controller | Description | Events |
|------------|-------------|--------|
| `Keypad` | Standard button press | `onRun`, `onKeyDown`, `onKeyUp` |
| `Encoder` | Rotary dial | `onDialDown`, `onDialUp`, `onDialRotate*` |

**Example Configuration:**
```json
{
  "Controllers": ["Keypad", "Encoder"],
  "Devices": ["D200X"]
}
```

This action supports both keypad buttons and rotary dial on the D200X device.

### Encoder Layout (D200X)

The D200X dial area has an independent layout canvas for customizing display content:

```json
"Encoder": {
  "layout": "$UA1"        // Built-in layout: icon + text
  // or "layout": "layout.json"  // Custom layout file
}
```

**Built-in Layouts:**
- `$UA1` — Icon + Text layout
- `$UA2` — Text + Text layout

---

## 🚀 Core APIs

### Button Icons

```js
// Use state index from manifest.json States array
$UD.setStateIcon(context, stateIndex, text)

// Use local image file path (relative to plugin root)
$UD.setPathIcon(context, 'resources/icon.png', text)

// Use base64-encoded image data
$UD.setBaseDataIcon(context, base64Data, text)

// Use local GIF file for animation
$UD.setGifPathIcon(context, 'anim.gif', text)

// Use base64-encoded GIF data
$UD.setGifDataIcon(context, base64GifData, text)
```

### Settings Persistence

```js
// Save action-specific settings (only saves when action is active)
$UD.setSettings(data, context)

// Request saved action settings (response via onDidReceiveSettings)
$UD.getSettings(context)

// Save plugin-wide global settings
$UD.setGlobalSettings(data)

// Request global settings (response via onDidReceiveGlobalSettings)
$UD.getGlobalSettings()
```

**Important:** Settings are NOT saved when the action is inactive.

### System Functions

```js
// Show toast notification on UlanziStudio
$UD.toast('Hello World')

// Trigger system-level hotkey
$UD.hotkey('Ctrl+C')           // Windows
$UD.hotkey('⌘C')               // macOS

// Open URL in system browser
$UD.openUrl('https://example.com')

// Open local HTML file in popup window
$UD.openView('popup.html', 400, 300)

// Open file picker dialog
$UD.selectFileDialog('image(*.png *.jpg)')

// Open folder picker dialog
$UD.selectFolderDialog()

// Write to plugin log file
$UD.logMessage('Debug info', 'debug')

// Show error indicator on button
$UD.showAlert(context)
```

### Event Handlers

```js
// Action added to key (drag-and-drop)
$UD.onAdd((message) => {
  console.log('Action added:', message.context);
})

// Key triggered (single click confirmed) — main entry point
$UD.onRun((message) => {
  console.log('Action triggered:', message.context);
})

// Key press started (fires before run, useful for long-press detection)
$UD.onKeyDown((message) => {})

// Key released
$UD.onKeyUp((message) => {})

// Action active state changed
$UD.onSetActive((message) => {
  console.log('Active:', message.active);  // true/false
})

// Action removed from one or more keys
$UD.onClear((message) => {
  // message.param is array, each item has .context
})
```

### Encoder Events

```js
// Dial pressed
$UD.onDialDown((message) => {})

// Dial released
$UD.onDialUp((message) => {})

// Dial rotated (any direction)
$UD.onDialRotate((message) => {
  // message.rotateEvent: 'left' | 'right' | 'hold-left' | 'hold-right'
})

// Rotated left (not held)
$UD.onDialRotateLeft((message) => {})

// Rotated right (not held)
$UD.onDialRotateRight((message) => {})

// Rotated left while pressed
$UD.onDialRotateHoldLeft((message) => {})

// Rotated right while pressed
$UD.onDialRotateHoldRight((message) => {})
```

### Cross-Page Communication

```js
// Main Service → Property Inspector (pass-through, not saved by host)
$UD.sendToPropertyInspector(data, context)

// Property Inspector → Main Service (pass-through, not saved by host)
$UD.sendToPlugin(data)
```

**Event Handlers:**
```js
// In Main Service: receive from Property Inspector
$UD.onSendToPlugin((message) => {})

// In Property Inspector: receive from Main Service
$UD.onSendToPropertyInspector((message) => {})
```

---

## ⚠️ Important Notes

1. **Settings Not Saved When Inactive**
   Settings are NOT saved when the action is inactive. Check `onSetActive` before relying on `setSettings`.

2. **Simulator Limitations**
   - `openview` and `openurl` cannot open local files (browser restriction)
   - Main service must be started separately for Node.js plugins
   - Actions are not auto-loaded by default

3. **Path Handling**
   Use `Utils.getPluginPath()` for cross-platform path handling:
   ```js
   const pluginPath = Utils.getPluginPath();
   const configPath = `${pluginPath}/config.json`;
   ```

4. **Port Management**
   For Node.js main services, use `RandomPort` to avoid port conflicts:
   ```js
   const port = new RandomPort().getPort();  // Writes ws-port.js
   ```

5. **Context in clear Event**
   For the `clear` event, `context` is in each item of the `param` array:
   ```js
   $UD.onClear((message) => {
     if (message.param) {
       for (const item of message.param) {
         console.log('Removed:', item.context);
       }
     }
   });
   ```

---

## 📚 Reference Documentation

| Topic | Link |
|-------|------|
| manifest.json Full Reference | [manifest.md](./manifest.md) |
| common-html SDK (Property Inspector) | [common-html/README.md](./common-html/README.md) |
| common-node SDK (Main Service) | [common-node/README.md](./common-node/README.md) |
| Simulator Guide | [UlanziDeckSimulator/README.md](./UlanziDeckSimulator/README.md) |
| Localization (i18n) | See [common-html/README.md » Localization](./common-html/README.md) |
| Debugging Guide | See Step 8 above or [common-html/README.md » Debugging](./common-html/README.md) |

---

## 📄 manifest.json Reference

| Field | Required | Description |
|-------|----------|-------------|
| `UUID` | ✅ | Plugin ID (exactly 4 segments) |
| `CodePath` | ✅ | Entry point (`.html` or `.js`) |
| `Actions` | ✅ | Action definitions array |
| `Type` | ✅ | Fixed value `"JavaScript"` |
| `Author` | ✅ | Developer name |
| `Name` | ✅ | Plugin name |
| `Icon` | ✅ | Plugin icon path |
| `Version` | ✅ | Plugin version |
| `Controllers` | | `["Keypad"]`, `["Encoder"]`, or both |
| `Devices` | | `[]` = all devices, `["D200X"]` = specific |
| `OS` | | Supported operating systems |

Full reference: **[manifest.md](./manifest.md)**

---

## 🔗 Related Links

| Resource | Link |
|----------|------|
| Download | https://www.ulanzi.com/pages/ulanzi-app |
| Community Forum | https://bbs.ulanzistudio.com |

---

## 📜 License

**AGPL 3.0** — Services using this SDK must disclose their modified source code.

---

## 📬 Contact

Plugin development inquiries: **ustudioservice@ulanzi.com**
