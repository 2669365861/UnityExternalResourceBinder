# UnityExternalResourceBinder
# Unity 外部资源动态加载器

## 📖 概述

本工具集提供了一个轻量级、高性能的 Unity 外部资源加载解决方案。它允许应用程序在运行时动态读取外部文件（图片、视频、音频、文本），无需重新打包即可更新内容。

核心包含两部分：
1.  **ResourceManager**：全局单例，负责异步加载与资源缓存。
2.  **ExternalResourceBinder**：组件化工具，挂载在 UI 物体上即可实现“即插即用”。

---

## ✨ 核心特性

-   **🔌 即插即用**：无需编写代码，通过 Inspector 面板直接配置。
-   **📁 智能路径**：支持“绝对路径”与“项目相对路径”两种模式，完美适配团队协作与项目迁移。
-   **🚀 性能优化**：
    -   异步加载：基于 `UnityWebRequest`，避免主线程卡顿。
    -   资源缓存：自动缓存已加载资源，避免重复 IO。
    -   内存安全：自动销毁旧的 Sprite/Texture，防止内存泄漏。
-   **🛠️ 广泛兼容**：
    -   支持 **TextMeshPro** (TMP_Text)。
    -   支持 **AVPro Video** 插件与原生 VideoPlayer 无缝切换。

---

## 📦 安装说明

1.  将 `ExternalResourceLoader` 文件夹复制到 Unity 项目的 `Assets` 目录下。
2.  **AVPro 用户**：确保已导入 AVPro Video 插件，并在 `Player Settings -> Scripting Define Symbols` 中添加 `AVPRO_VIDEO`（通常插件会自动添加）。

---

## 🚀 快速入门

### 1. 挂载脚本
在场景中的 UI 物体（如 Image、RawImage、空物体）上挂载 `ExternalResourceBinder` 脚本。

### 2. 组件自动绑定
脚本在 `Reset` 或首次添加时会自动扫描当前物体并绑定以下组件：
-   `Image` / `RawImage`
-   `VideoPlayer` / `AudioSource`
-   `TMP_Text` (优先) / `Text`

> 💡 提示：如果需要手动指定其他物体上的组件，请直接在 Inspector 中拖拽赋值覆盖。

### 3. 配置参数

| 参数 | 说明 |
| :--- | :--- |
| **Path Mode** | `Absolute`: 使用系统完整路径。<br>`RelativeToProject`: 基于项目根目录的相对路径（推荐）。 |
| **Resource Type** | 选择要加载的资源类型。 |
| **Video Plugin** | 选择视频播放器类型（需先安装对应插件）。 |

### 4. 选择文件
点击 Inspector 面板底部的 **“选择外部文件”** 按钮。
-   系统会弹出文件选择窗口。
-   选择文件后，路径将自动填入，并根据 `Path Mode` 自动转换为相应格式。

---

## 📚 详细功能说明

### 1. 路径模式详解

| 模式 | 适用场景 | 示例 |
| :--- | :--- | :--- |
| **绝对路径** | 适合单机部署、路径固定的展厅程序。 | `D:/Exhibits/Images/bg.png` |
| **相对路径** | **推荐模式**。适合团队协作，项目文件夹位置改变后路径依然有效。 | `StreamingAssets/Images/bg.png` |

### 2. 资源类型支持

| 类型 | 支持组件 | 备注 |
| :--- | :--- | :--- |
| **图片** | `Image`, `RawImage` | 自动创建 `Sprite` 并管理内存释放。 |
| **视频** | `VideoPlayer`, `AVPro MediaPlayer` | 采用 URL 流式播放，不占用内存。 |
| **音频** | `AudioSource` | 支持 mp3, wav, ogg 等格式。 |
| **文本** | `TMP_Text`, `Text` | 优先识别 TextMeshPro 组件。 |

### 3. 视频播放优化

-   **Unity VideoPlayer**: 自动添加 `file:///` 前缀以兼容 Windows 本地文件协议。
-   **AVPro Video**: 调用 `OpenMedia` 接口，支持更高性能的 4K/8K 解码。

---

## 💻 API 接口 (代码调用)

如果您需要在代码中动态切换资源，请使用以下接口：

