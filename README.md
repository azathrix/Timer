<h1 align="center">Timer</h1>

<p align="center">
  Unity 定时器系统，提供延迟执行、重复执行、帧计数、条件等待等功能
</p>

<p align="center">
  <a href="https://github.com/Azathrix/Timer"><img src="https://img.shields.io/badge/GitHub-Timer-black.svg" alt="GitHub"></a>
  <a href="https://www.npmjs.com/package/com.azathrix.timer"><img src="https://img.shields.io/npm/v/com.azathrix.timer.svg" alt="npm"></a>
  <a href="https://github.com/Azathrix/Timer/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"></a>
  <a href="https://unity.com/"><img src="https://img.shields.io/badge/Unity-6000.3+-black.svg" alt="Unity"></a>
</p>

---

## 特性

- 延迟执行、重复执行
- 帧计数（下一帧、延迟帧数、帧重复）
- 条件等待（WaitUntil、WaitWhile）
- 进度回调，配合 Lerp 使用
- Builder 模式，复杂配置
- 暂停/恢复/取消/重置

## 安装

### 方式一：Package Manager（推荐）

1. 打开 `Edit > Project Settings > Package Manager`
2. 在 `Scoped Registries` 中添加：
   - Name: `Azathrix`
   - URL: `https://registry.npmjs.org`
   - Scope(s): `com.azathrix`
3. 点击 `Save`
4. 打开 `Window > Package Manager`
5. 切换到 `My Registries`
6. 找到 `Timer` 并安装

### 方式二：修改 manifest.json

在 `Packages/manifest.json` 中添加：

```json
{
  "scopedRegistries": [
    {
      "name": "Azathrix",
      "url": "https://registry.npmjs.org",
      "scopes": ["com.azathrix"]
    }
  ],
  "dependencies": {
    "com.azathrix.timer": "*"
  }
}
```

### 方式三：Git URL

1. 打开 `Window > Package Manager`
2. 点击 `+` > `Add package from git URL...`
3. 输入：`https://github.com/Azathrix/Timer.git`

> ⚠️ Git 方式无法自动解析依赖，需要先手动安装：
> - [Azathrix Framework](https://github.com/Azathrix/AzathrixFramework.git)

## 快速开始

```csharp
var timerManager = AzathrixFramework.GetSystem<TimerManager>();

// 延迟执行
timerManager.Delay(2f, () => Debug.Log("2秒后执行"));

// 重复执行
timerManager.Repeat(0.5f, () => Debug.Log("每0.5秒执行"), repeatCount: 10);

// 下一帧执行
timerManager.NextFrame(() => Debug.Log("下一帧"));

// 帧延迟
timerManager.DelayFrames(60, () => Debug.Log("60帧后"));
```

## 功能

### 延迟执行

```csharp
// 基础延迟
timerManager.Delay(2f, () => Debug.Log("完成"));

// 使用真实时间（不受 TimeScale 影响）
timerManager.Delay(2f, () => Debug.Log("完成"), useRealTime: true);
```

### 重复执行

```csharp
// 重复10次
timerManager.Repeat(0.5f, () => Debug.Log("执行"), repeatCount: 10);

// 无限重复
var timer = timerManager.Repeat(1f, () => Debug.Log("每秒执行"));
timer.Cancel(); // 手动取消
```

### 帧计数

```csharp
// 下一帧
timerManager.NextFrame(() => Debug.Log("下一帧"));

// 延迟帧数
timerManager.DelayFrames(60, () => Debug.Log("60帧后"));

// 帧重复
timerManager.RepeatFrames(30, () => Debug.Log("每30帧执行"), repeatCount: 5);
```

### 条件等待

```csharp
// 等待条件满足
timerManager.WaitUntil(() => player.IsReady, () => StartGame());

// 带超时
timerManager.WaitUntil(() => isLoaded, () => OnLoaded(), timeout: 10f);

// 等待条件不满足
timerManager.WaitWhile(() => animator.IsPlaying, () => OnAnimationEnd());
```

### 进度回调

```csharp
// 每帧回调进度 (0-1)
timerManager.Create(3f,
    progress => slider.value = progress,
    () => Debug.Log("完成"));

// 配合 Lerp 使用
var startPos = transform.position;
var endPos = targetPos;
timerManager.Create(2f, p => transform.position = Vector3.Lerp(startPos, endPos, p));
```

### Builder 模式

复杂配置使用 Builder：

```csharp
timerManager.GetBuilder()
    .SetDuration(3f)
    .SetOnUpdate(p => slider.value = p)
    .SetOnComplete(() => Debug.Log("完成"))
    .UseRealTime()              // 不受 TimeScale 影响
    .BindTo(gameObject)         // 绑定到 GameObject，销毁时自动取消
    .Build();

// 帧重复
timerManager.GetBuilder()
    .SetFrameRepeat(30, 5)      // 每30帧执行，共5次
    .SetOnComplete(() => Debug.Log("执行"))
    .Build();

// 保存配置复用
var config = timerManager.GetBuilder()
    .SetDuration(1f)
    .UseRealTime()
    .BuildContext();

timerManager.GetBuilder(config)
    .SetOnComplete(() => Debug.Log("使用配置"))
    .Build();
```

### 定时器控制

```csharp
var timer = timerManager.Delay(5f, () => Debug.Log("完成"));

// 暂停/恢复
timer.Pause();
timer.Resume();

// 查看状态
Debug.Log($"进度: {timer.Progress}");      // 0-1
Debug.Log($"已过时间: {timer.Elapsed}");
Debug.Log($"剩余时间: {timer.Remaining}");
Debug.Log($"是否运行: {timer.IsRunning}");
Debug.Log($"是否暂停: {timer.IsPaused}");

// 取消
timer.Cancel();

// 立即完成（触发回调）
timer.Complete();

// 重置
timer.Reset();
```

### 全局控制

```csharp
timerManager.PauseAll();    // 暂停所有
timerManager.ResumeAll();   // 恢复所有
timerManager.CancelAll();   // 取消所有
timerManager.CancelAll(gameObject);  // 取消指定 GameObject 的定时器
```

## 依赖

- [Azathrix Framework](https://www.npmjs.com/package/com.azathrix.framework)

## License

MIT
