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
这里是详细的 **API 接口文档**，供开发人员在代码中动态调用。
---
## 📘 API 接口文档
本框架提供了两层 API：**组件级 API**（快速绑定 UI）和 **核心级 API**（全局加载与缓存）。
### 1. 组件级 API (`ExternalResourceBinder`)
如果您已经将脚本挂载在 UI 物体上，只需通过代码控制切换逻辑。
#### 方法：动态切换资源
```csharp
/// <summary>
/// 运行时动态切换外部资源路径
/// </summary>
/// <param name="newPath">新的文件路径（绝对路径或相对路径，取决于Inspector中的PathMode设置）</param>
public void LoadNewResource(string newPath)
```
**使用示例：**
```csharp
// 场景：点击按钮切换背景图
public class UIManager : MonoBehaviour
{
    // 拖拽获取绑定器引用
    public ExternalResourceBinder backgroundBinder;
    public void OnClickChangeBg()
    {
        // 方式1：使用绝对路径
        backgroundBinder.LoadNewResource("D:/Gallery/Images/bg_02.jpg");
        // 方式2：使用相对路径 (前提：Inspector中 PathMode 设为 RelativeToProject)
        // backgroundBinder.LoadNewResource("StreamingAssets/Images/bg_02.jpg");
    }
}
```
---
### 2. 核心级 API (`ResourceManager`)
如果您不想挂载组件，或者需要在纯逻辑脚本中加载资源，请使用全局管理器。
#### 方法：异步加载资源
```csharp
/// <summary>
/// 异步加载外部资源（自动缓存）
/// </summary>
/// <param name="filePath">文件的绝对路径</param>
/// <param name="type">资源类型</param>
/// <param name="onComplete">完成回调，返回 LoadResult 对象</param>
public void LoadResourceAsync(string filePath, ExternalResourceType type, Action<LoadResult> onComplete)
```
**使用示例：加载图片并手动赋值**
```csharp
using UnityEngine;
using UnityEngine.UI;
public class DynamicLoader : MonoBehaviour
{
    public RawImage targetImage; // 目标显示组件
    public void LoadExternalImage()
    {
        string path = "C:/Data/MyPhoto.jpg";
        // 调用全局管理器加载
        ResourceManager.Instance.LoadResourceAsync(
            path, 
            ExternalResourceType.Image, 
            OnImageLoaded
        );
    }
    // 加载完成回调
    private void OnImageLoaded(LoadResult result)
    {
        // 1. 安全检查
        if (!result.Success)
        {
            Debug.LogError($"加载失败: {result.Error}");
            return;
        }
        // 2. 获取资源对象
        Texture2D tex = result.Data as Texture2D;
        // 3. 业务逻辑处理
        if (tex != null && targetImage != null)
        {
            targetImage.texture = tex;
            Debug.Log("图片加载并赋值成功！");
        }
    }
}
```
**使用示例：加载文本数据**
```csharp
public void LoadConfigData()
{
    string path = Path.Combine(Application.streamingAssetsPath, "config.json");
    
    ResourceManager.Instance.LoadResourceAsync(
        path, 
        ExternalResourceType.Text, 
        (result) => 
        {
            if (result.Success)
            {
                string jsonContent = result.Data as string;
                Debug.Log($"读取到的配置内容: {jsonContent}");
                // TODO: 解析 JSON...
            }
        }
    );
}
```
#### 方法：释放资源
```csharp
/// <summary>
/// 强制释放指定路径的缓存资源（销毁对象）
/// </summary>
public void ReleaseResource(string filePath)
```
**使用示例：**
```csharp
// 场景：场景切换时清理大图，释放内存
public void CleanUp()
{
    string oldPath = "C:/Data/BigTexture.jpg";
    ResourceManager.Instance.ReleaseResource(oldPath);
}
```
---
### 3. 数据结构 (`LoadResult`)
加载完成回调返回的对象结构。
```csharp
public class LoadResult
{
    // 是否加载成功
    public bool Success;
    
    // 错误信息（仅在 Success = false 时有值）
    public string Error;
    
    // 资源对象（需根据类型强转）
    // Image -> Texture2D
    // Audio -> AudioClip
    // Text  -> string
    // Video -> string (视频返回路径，不加载进内存)
    public object Data;
}
```
---
### 4. 枚举定义
在代码中调用时需要用到的类型定义：
```csharp
// 资源类型
public enum ExternalResourceType
{
    Image,  // 图片 -> 返回 Texture2D
    Video,  // 视频 -> 返回路径
    Audio,  // 音频 -> 返回 AudioClip
    Text    // 文本 -> 返回 string
}
// 路径模式
public enum PathMode
{
    Absolute,           // 绝对路径
    RelativeToProject   // 项目相对路径
}
// 视频插件类型
public enum VideoPluginType
{
    UnityVideoPlayer,   // 原生播放器
    AVProVideo          // AVPro 插件
}
```

