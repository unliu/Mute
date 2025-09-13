# Mute.shared 单例实例

<cite>
**本文档引用的文件**  
- [Mute.swift](file://Mute/Classes/Mute.swift#L19)
- [ViewController.swift](file://Example/Mute/ViewController.swift#L23-L47)
- [README.md](file://README.md#L45-L75)
</cite>

## 目录
1. [简介](#简介)
2. [核心属性分析](#核心属性分析)
3. [单例初始化与线程安全性](#单例初始化与线程安全性)
4. [生命周期与资源管理](#生命周期与资源管理)
5. [实际使用示例](#实际使用示例)
6. [与其他组件的协同工作](#与其他组件的协同工作)

## 简介
`Mute` 是一个用于检测 iOS 设备静音开关状态的 Swift 库。由于 iOS 系统本身未提供原生 API 来检测静音开关，该库通过播放一个短暂的无声音频并测量其播放时长来判断设备是否处于静音状态。`Mute.shared` 是该库的核心单例实例，作为全局访问点，开发者可通过它配置检测频率、设置回调通知，并触发手动检测。

**Section sources**
- [README.md](file://README.md#L1-L10)

## 核心属性分析
`Mute.shared` 是 `Mute` 类的静态常量属性，采用 Swift 的 `static let` 语法实现单例模式。该属性在类加载时由系统保证线程安全地初始化一次，此后所有对 `Mute.shared` 的访问都返回同一个实例。

```swift
public static let shared = Mute()
```

此设计确保了：
- **全局唯一性**：整个应用生命周期内仅存在一个 `Mute` 实例。
- **全局可访问性**：任何需要检测静音状态的模块都可以通过 `Mute.shared` 直接访问。
- **惰性初始化**：实例在第一次被访问时才创建，优化了启动性能。

**Section sources**
- [Mute.swift](file://Mute/Classes/Mute.swift#L19)

## 单例初始化与线程安全性
`Mute` 类的单例模式通过以下机制保障：
1. **私有化构造函数**：使用 `private override init()` 确保外部无法通过 `Mute()` 创建新实例。
2. **静态常量**：`static let shared` 在 Swift 中由运行时保证线程安全的初始化。当多个线程同时首次访问 `shared` 时，Swift 会确保 `init()` 方法仅被调用一次。

```swift
private override init() {
    super.init()
    // 初始化音频资源并开始调度检测
    self.schedulePlaySound()
    // 注册应用前后台切换通知
    NotificationCenter.default.addObserver(self, selector: #selector(Mute.didEnterBackground(_:)), name: UIApplication.didEnterBackgroundNotification, object: nil)
    NotificationCenter.default.addObserver(self, selector: #selector(Mute.willEnterForeground(_:)), name: UIApplication.willEnterForegroundNotification, object: nil)
}
```

这种实现方式符合 Swift 中推荐的线程安全单例模式，无需额外的同步机制。

**Section sources**
- [Mute.swift](file://Mute/Classes/Mute.swift#L106-L120)

## 生命周期与资源管理
`Mute` 单例的生命周期与应用进程一致。为避免内存泄漏和资源浪费，其在析构时会执行必要的清理工作：

```swift
deinit {
    if self.soundId != 0 {
        AudioServicesRemoveSystemSoundCompletion(self.soundId)
        AudioServicesDisposeSystemSoundID(self.soundId)
    }
    NotificationCenter.default.removeObserver(self)
}
```

清理内容包括：
- **释放音频资源**：调用 `AudioServicesDisposeSystemSoundID` 释放由 `AudioToolbox` 框架分配的 `SystemSoundID`。
- **移除通知观察者**：防止在实例销毁后仍接收通知导致的崩溃。

此外，`Mute` 通过监听 `UIApplication.didEnterBackgroundNotification` 和 `UIApplication.willEnterForegroundNotification` 通知，在应用进入后台时暂停检测（`isPaused = true`），在返回前台时恢复检测，从而优化后台资源消耗。

**Section sources**
- [Mute.swift](file://Mute/Classes/Mute.swift#L137-L145)

## 实际使用示例
以下示例展示了如何在 `ViewController` 中使用 `Mute.shared`：

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    // 设置检测间隔为2秒
    Mute.shared.checkInterval = 2.0

    // 设置为总是通知（即使状态未改变）
    Mute.shared.alwaysNotify = true

    // 设置状态变化时的回调
    Mute.shared.notify = { [weak self] isMuted in
        self?.label.text = isMuted ? "已静音" : "未静音"
    }

    // 5秒后暂停检测
    DispatchQueue.main.asyncAfter(deadline: .now() + 5.0) {
        Mute.shared.isPaused = true
    }

    // 10秒后恢复检测
    DispatchQueue.main.asyncAfter(deadline: .now() + 10.0) {
        Mute.shared.isPaused = false
    }
}

@IBAction func checkPressed(_ sender: UIButton) {
    // 手动触发一次检测
    Mute.shared.check()
}
```

**Section sources**
- [ViewController.swift](file://Example/Mute/ViewController.swift#L23-L47)

## 与其他组件的协同工作
`Mute.shared` 通过以下属性与应用其他部分协同工作：
- **`notify` 回调**：当静音状态改变或达到检测间隔时，调用此闭包，通常用于更新 UI。
- **`checkInterval`**：控制自动检测的频率，默认为1秒，最小值为0.5秒。
- **`isPaused`**：控制检测的启停，常与应用生命周期结合使用。
- **`check()` 方法**：允许开发者在需要时立即执行一次检测，无需等待下一个时间间隔。

这些组件共同构成了一个灵活、低侵入性的静音状态监控系统。

**Section sources**
- [Mute.swift](file://Mute/Classes/Mute.swift#L25-L55)
- [README.md](file://README.md#L45-L75)