# UlanziDeck Plugin SDK

<p align="start">
   <a href="./README.md">English</a> · <strong>简体中文</strong>
</p>

> **UlanziStudio**可编程宏按键平台官方开发套件
>
> **协议版本：** 基于 Ulanzi JS 插件开发协议 V2.1.2
> **兼容版本：** Ulanzi Studio 3.0.11

---

## ⚡ 快速开始

### 1️⃣ 创建插件结构

按照命名规范创建插件文件夹 `com.ulanzi.{插件名}.ulanziPlugin`：

```bash
com.ulanzi.myplugin.ulanziPlugin/
├── manifest.json          # 插件配置（必需）
├── libs/                  # SDK 库
├── resources/             # 图标和资源
├── plugin/
│   └── app.js            # 主服务入口
└── property-inspector/
    └── inspector.html     # 配置页面
```

**命名规范：**
- 插件文件夹：`com.ulanzi.{插件名}.ulanziPlugin`
- 主服务 UUID：`com.ulanzi.ulanzistudio.{插件名}`（4 段）
- Action UUID：`com.ulanzi.ulanzistudio.{插件名}.{功能名}`（5 段以上）

---

### 2️⃣ 配置 manifest.json

在插件根目录创建 `manifest.json`：

```json
{
  "Author": "您的名称",
  "Name": "我的插件",
  "Description": "插件描述",
  "Icon": "resources/icon.png",
  "Version": "1.0.0",
  "UUID": "com.ulanzi.ulanzistudio.myplugin",
  "Type": "JavaScript",
  "CodePath": "plugin/app.js",
  "Actions": [
    {
      "Name": "我的功能",
      "Icon": "resources/action-icon.png",
      "UUID": "com.ulanzi.ulanzistudio.myplugin.action",
      "PropertyInspectorPath": "property-inspector/inspector.html",
      "States": [
        { "Name": "默认", "Image": "resources/action-icon.png" }
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

**必填字段：**
- `UUID` — 插件唯一标识（恰好 4 段）
- `CodePath` — 主入口（`.html` 用于 WebView，`.js` 用于 Node.js）
- `Actions` — Action 功能列表数组
- `Type` — 固定值 `"JavaScript"`

**Action 字段：**
- `UUID` — Action 唯一标识（5 段以上）
- `Controllers` — `["Keypad"]`、`["Encoder"]` 或两者
- `Devices` — 目标设备（空数组 `[]` = 所有设备）

---

### 3️⃣ 安装 SDK

复制 SDK 库到您的插件项目：

```bash
# HTML 配置页面 SDK
cp -r common-html/libs ./libs/

# Node.js 主服务 SDK
cp -r common-node ./plugin-common-node/

# 安装 WebSocket 依赖
npm install ws
```

**SDK 结构：**
- `common-html/libs/` — 用于配置页面的 WebSocket 客户端、UI 工具、国际化
- `common-node/` — 用于主服务的 WebSocket 服务端、系统 API

---

### 4️⃣ 实现插件

插件可以使用 HTML 或 Node.js 作为主服务。根据需求选择：
- **HTML** — 适合 UI 丰富的插件、Canvas 绘制、逻辑较简单
- **Node.js** — 适合系统集成、文件访问、逻辑复杂

**HTML 主服务（`plugin/app.html`）：**

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
      console.log('已连接到 UlanziStudio');
    });

    $UD.onAdd((message) => {
      console.log('功能已添加:', message.context);
    });

    $UD.onRun((message) => {
      // 主要的按键触发逻辑
      console.log('功能触发:', message.context);
      $UD.toast('插件运行成功！');
    });
  </script>
</body>
</html>
```

**Node.js 主服务（`plugin/app.js`）：**

```js
import UlanziApi, { Utils, RandomPort } from './plugin-common-node/index.js';

const $UD = new UlanziApi();

// 生成随机端口供配置页面连接（仅 Node.js 需要）
const port = new RandomPort().getPort();

// 连接到 UlanziStudio
$UD.connect('com.ulanzi.ulanzistudio.myplugin');

$UD.onConnected(() => {
  console.log('已连接到 UlanziStudio');
});

$UD.onAdd((message) => {
  console.log('功能已添加:', message.context);
});

$UD.onRun((message) => {
  // 主要的按键触发逻辑
  console.log('功能触发:', message.context);
  $UD.toast('插件运行成功！');
});
```

**主要区别：**
- HTML 使用 `<script>` 标签加载 SDK；Node.js 使用 ES 模块（`import`）
- Node.js 需要 `RandomPort` 生成 WebSocket 端口供配置页面连接
- 两者使用相同的 `$UD` API 处理事件和命令

---

### 5️⃣ 实现配置页面

创建 `property-inspector/inspector.html`：

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

    // 功能添加时加载已保存的设置
    $UD.onAdd((message) => {
      Utils.setFormValue(message.param, '#property-inspector');
    });

    // 表单变化时发送设置到主服务
    document.getElementById('property-inspector').addEventListener('change', () => {
      const params = Utils.getFormValue('#property-inspector');
      $UD.sendParamFromPlugin(params);
    });
  </script>
</body>
</html>
```

**关键点：**
- 使用 `.uspi-wrapper` 包裹内容以启用自动国际化与样式
- 使用 `data-localize` 属性实现自动翻译
- 使用 `Utils.getFormValue()` 和 `Utils.setFormValue()` 处理表单

---

### 6️⃣ 添加国际化（可选）

在插件根目录创建国际化 JSON 文件：

**zh_CN.json：**
```json
{
  "Localization": {
    "Message": "消息",
    "Save": "保存",
    "Hello": "你好"
  }
}
```

**en.json：**
```json
{
  "Localization": {
    "Message": "Message",
    "Save": "Save",
    "Hello": "Hello"
  }
}
```

**支持的语言：**
- `zh_CN` — 简体中文
- `zh_HK` — 繁体中文（香港）
- `en` — 英语
- `ja_JP` — 日语
- `de_DE` — 德语
- `ko_KR` — 韩语
- `pt_PT` — 葡萄牙语
- `es_ES` — 西班牙语

**HTML 中使用：**
```html
<!-- 自动翻译元素文本内容 -->
<div class="uspi-item-label" data-localize>Message</div>

<!-- 自动翻译属性值 -->
<option value="save" data-localize="Save">Save</option>

<!-- JavaScript 中手动翻译 -->
<script>
  const label = $UD.t('Message');  // 返回翻译后的字符串
</script>
```

---

### 7️⃣ 使用模拟器测试

UlanziDeck Simulator 允许在不使用桌面应用的情况下测试插件：

```bash
cd UlanziDeckSimulator
npm install
npm start
```

**测试步骤：**

1. **复制插件** — 将插件文件夹复制到 `UlanziDeckSimulator/plugins/`

2. **打开模拟器** — 在浏览器中打开 http://127.0.0.1:39069

3. **刷新插件列表** — 点击 "Refresh Plugin List" 按钮加载插件

4. **启动主服务** — 对于 Node.js 插件，手动启动主服务：
   ```bash
   node path/to/your/plugin/app.js
   ```

5. **添加功能到按键** — 从插件列表拖拽功能到按键

6. **打开配置页面** — 点击按键打开配置 UI

7. **测试事件** — 使用右键菜单手动触发事件进行测试

**模拟器限制：**
- `openview` 和 `openurl` 无法打开本地文件（浏览器限制）
- Node.js 插件的主服务需手动启动
- 功能默认不自动加载

---

### 8️⃣ 在桌面应用中调试

启动 UlanziStudio 时附加调试参数：

**可用参数：**

| 参数 | 说明 |
|------|------|
| `--log` | 写入日志到文件 |
| `--logLevel` | 设置日志详细程度 |
| `--webRemoteDebug` | 启用 HTML 插件调试（端口 9292） |
| `--nodeRemoteDebug` | 启用 Node.js 插件调试 |

**Windows：**

右键 UlanziStudio 快捷方式 → 属性 → 在 **目标** 栏末尾追加：

```
"C:\...\Ulanzi Studio.exe" --log --webRemoteDebug --nodeRemoteDebug
```

**macOS：**

```bash
open /Applications/Ulanzi\ Studio.app --args --log --webRemoteDebug
```

**访问调试工具：**

- **HTML 插件：** 在浏览器中打开 `localhost:9292` 调试所有已加载的 HTML 插件
- **Node.js 插件：** 在 Chrome 中打开 `chrome://inspect`，找到插件后点击 "inspect"

**重要提示：**
- macOS 使用 `open` 命令可能导致应用无法获得辅助功能权限，影响快捷键功能
- 如快捷键不工作，请使用 `./UlanziStudio` 直接启动

---

## 📦 SDK 概览

| 包名 | 用途 |
|------|------|
| [`common-html`](./common-html/README.zh.md) | HTML 插件（WebView）— WebSocket 客户端、UI 工具、国际化、Canvas 辅助 |
| [`common-node`](./common-node/README.zh.md) | Node.js 插件 — WebSocket 服务端、系统 API、文件系统访问 |
| [`UlanziDeckSimulator`](./UlanziDeckSimulator/README.zh.md) | 浏览器模拟器 |
| [`demo`](./demo/) | 示例插件 |

---

## 🏗️ 架构设计

### 主服务类型

插件可以使用 HTML 或 Node.js 作为主服务：

| 类型 | 入口文件 | 加载方式 | SDK | 适用场景 |
|------|---------|---------|-----|---------|
| **HTML 插件** | `.html` | WebView | `common-html` | 丰富 UI、Canvas 绘制、逻辑简单 |
| **Node.js 插件** | `.js` | Node.js v20 | `common-node` | 系统集成、文件访问、逻辑复杂 |

**配置页面**（可选）始终使用 HTML 和 `common-html`，在用户配置功能时打开。

### 通信流程

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│   配置页面       │  WebSocket│   UlanziStudio │  WebSocket│   主服务         │
│   (HTML)        │ ←──────→│     (桥接)       │ ←──────→│   (Node.js/HTML)│
└─────────────────┘         └──────────────────┘         └─────────────────┘
```

**插件生命周期：**
- **主服务** — 长时间运行的后台进程（Node.js 或 HTML）
- **配置页面** — 用户配置按键时创建，切换页面时销毁

---

## 🔑 核心概念

### UUID 规则

```
主服务：     com.ulanzi.ulanzistudio.{plugin}           → 4 段
功能：       com.ulanzi.ulanzistudio.{plugin}.{action}  → 5 段以上
插件包：     com.ulanzi.{plugin}.ulanziPlugin
```

### Context 参数

每个按键实例有唯一的 `context` 字符串：

```js
// 格式：uuid + '___' + key + '___' + actionid
const context = $UD.encodeContext(msg)     // 编码为 context 字符串
const decoded = $UD.decodeContext(context) // 解码 → { uuid, key, actionid }
```

**重要：** 同一个功能可以被分配到多个按键上。`context` 参数唯一标识每个实例。

---

## 🎮 设备与控制器

### 支持的设备

- **D200**
- **D200H**
- **Dial**
- **D200X**

### 控制器类型

功能指定支持的控制器类型：

| 控制器 | 描述 | 事件 |
|--------|------|------|
| `Keypad` | 标准按键 | `onRun`, `onKeyDown`, `onKeyUp` |
| `Encoder` | 旋钮 | `onDialDown`, `onDialUp`, `onDialRotate*` |

**示例配置：**
```json
{
  "Controllers": ["Keypad", "Encoder"],
  "Devices": ["D200X"]
}
```

此功能支持 D200X 设备上的按键和旋钮两种操作方式。

### 旋钮布局（D200X）

D200X 设备的旋钮区域有独立的布局画布用于自定义显示内容：

```json
"Encoder": {
  "layout": "$UA1"        // 预置布局：图标 + 文字
  // 或 "layout": "layout.json"  // 自定义布局文件
}
```

**预置布局：**
- `$UA1` — 图标 + 文字布局
- `$UA2` — 文字 + 文字布局

---

## 🚀 核心 API

### 按键图标

```js
// 使用 manifest.json States 数组中定义的状态编号
$UD.setStateIcon(context, stateIndex, text)

// 使用本地图片文件路径（相对于插件根目录）
$UD.setPathIcon(context, 'resources/icon.png', text)

// 使用 base64 编码的图片数据
$UD.setBaseDataIcon(context, base64Data, text)

// 使用本地 GIF 文件实现动画
$UD.setGifPathIcon(context, 'anim.gif', text)

// 使用 base64 编码的 GIF 数据
$UD.setGifDataIcon(context, base64GifData, text)
```

### 设置持久化

```js
// 保存功能级别的设置（仅在功能激活时保存）
$UD.setSettings(data, context)

// 请求已保存的功能设置（响应通过 onDidReceiveSettings）
$UD.getSettings(context)

// 保存插件全局设置
$UD.setGlobalSettings(data)

// 请求全局设置（响应通过 onDidReceiveGlobalSettings）
$UD.getGlobalSettings()
```

**重要：** 功能处于非激活状态时设置不会被保存。

### 系统功能

```js
// 在 UlanziStudio 上显示 Toast 提示
$UD.toast('你好世界')

// 触发系统级快捷键
$UD.hotkey('Ctrl+C')           // Windows
$UD.hotkey('⌘C')               // macOS

// 在系统浏览器中打开 URL
$UD.openUrl('https://example.com')

// 以弹窗形式打开本地 HTML 文件
$UD.openView('popup.html', 400, 300)

// 打开文件选择对话框
$UD.selectFileDialog('image(*.png *.jpg)')

// 打开文件夹选择对话框
$UD.selectFolderDialog()

// 写入插件日志文件
$UD.logMessage('调试信息', 'debug')

// 在按键上显示错误指示
$UD.showAlert(context)
```

### 事件处理

```js
// 功能被添加到按键（拖拽）
$UD.onAdd((message) => {
  console.log('功能已添加:', message.context);
})

// 按键被触发执行（单击确认）— 插件逻辑主入口
$UD.onRun((message) => {
  console.log('功能触发:', message.context);
})

// 按键按下（在 run 之前触发，可用于长按检测）
$UD.onKeyDown((message) => {})

// 按键释放
$UD.onKeyUp((message) => {})

// 功能激活状态变化
$UD.onSetActive((message) => {
  console.log('激活状态:', message.active);  // true/false
})

// 功能从一个或多个按键上移除
$UD.onClear((message) => {
  // message.param 是数组，每项包含 .context
})
```

### 旋钮事件

```js
// 旋钮按下
$UD.onDialDown((message) => {})

// 旋钮释放
$UD.onDialUp((message) => {})

// 旋钮旋转（任意方向）
$UD.onDialRotate((message) => {
  // message.rotateEvent: 'left' | 'right' | 'hold-left' | 'hold-right'
})

// 向左旋转（未按住）
$UD.onDialRotateLeft((message) => {})

// 向右旋转（未按住）
$UD.onDialRotateRight((message) => {})

// 按住旋钮并向左旋转
$UD.onDialRotateHoldLeft((message) => {})

// 按住旋钮并向右旋转
$UD.onDialRotateHoldRight((message) => {})
```

### 跨页面通信

```js
// 主服务 → 配置页面（透传数据，上位机不保存）
$UD.sendToPropertyInspector(data, context)

// 配置页面 → 主服务（透传数据，上位机不保存）
$UD.sendToPlugin(data)
```

**事件处理：**
```js
// 主服务中：接收来自配置页面的数据
$UD.onSendToPlugin((message) => {})

// 配置页面中：接收来自主服务的数据
$UD.onSendToPropertyInspector((message) => {})
```

---

## ⚠️ 重要提示

1. **非激活状态不保存设置**
   功能处于非激活状态时设置不会被保存。在依赖 `setSettings` 之前请检查 `onSetActive`。

2. **模拟器限制**
   - `openview` 和 `openurl` 无法打开本地文件（浏览器限制）
   - Node.js 插件的主服务需手动启动
   - 功能默认不自动加载

3. **路径处理**
   使用 `Utils.getPluginPath()` 处理跨平台路径：
   ```js
   const pluginPath = Utils.getPluginPath();
   const configPath = `${pluginPath}/config.json`;
   ```

4. **端口管理**
   Node.js 主服务使用 `RandomPort` 避免端口冲突：
   ```js
   const port = new RandomPort().getPort();  // 写入 ws-port.js
   ```

5. **clear 事件中的 Context**
   对于 `clear` 事件，`context` 在 `param` 数组的每个元素中：
   ```js
   $UD.onClear((message) => {
     if (message.param) {
       for (const item of message.param) {
         console.log('已移除:', item.context);
       }
     }
   });
   ```

---

## 📚 参考文档

| 主题 | 链接 |
|------|------|
| manifest.json 完整参考 | [manifest.md](./manifest.md) |
| common-html SDK（HTML 插件） | [common-html/README.zh.md](./common-html/README.zh.md) |
| common-node SDK（Node.js 插件） | [common-node/README.zh.md](./common-node/README.zh.md) |
| 模拟器使用指南 | [UlanziDeckSimulator/README.zh.md](./UlanziDeckSimulator/README.zh.md) |
| 国际化 (i18n) | 见 [common-html/README.zh.md » 国际化](./common-html/README.zh.md) |
| 调试指南 | 见步骤 8 或 [common-html/README.zh.md » 调试](./common-html/README.zh.md) |

---

## 📄 manifest.json 参考

| 字段 | 必填 | 说明 |
|------|------|------|
| `UUID` | ✅ | 插件标识（恰好 4 段） |
| `CodePath` | ✅ | 入口文件（`.html` 或 `.js`） |
| `Actions` | ✅ | Action 定义列表 |
| `Type` | ✅ | 固定值 `"JavaScript"` |
| `Author` | ✅ | 开发者名称 |
| `Name` | ✅ | 插件名称 |
| `Icon` | ✅ | 插件图标路径 |
| `Version` | ✅ | 插件版本 |
| `Controllers` | | `["Keypad"]`、`["Encoder"]` 或两者 |
| `Devices` | | `[]` = 所有设备，`["D200X"]` = 指定设备 |
| `OS` | | 支持的操作系统 |

完整参考：**[manifest.md](./manifest.md)**

---

## 🔗 相关链接

| 资源 | 链接 |
|------|------|
| 下载 | https://www.ulanzi.cn/software/ulanzi_studio |
| 官方论坛 | https://bbs.ulanzistudio.com |

---

## 📜 许可证

**AGPL 3.0** — 使用该 SDK 的服务提供商必须公开其修改后的源代码。

---

## 📬 联系方式

插件开发合作请联系：**ustudioservice@ulanzi.com**
