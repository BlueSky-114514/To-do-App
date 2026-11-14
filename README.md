# TodoApp · 待办事项桌面应用

基于 **C# / .NET 8 / Avalonia UI / FluentAvaloniaUI** 构建的跨平台待办（To-do）桌面应用。采用轻量 MVVM 分层，支持事项的增删改、完成勾选、条件筛选与本地 JSON 持久化，界面遵循 Fluent Design 风格，并可跟随系统明暗主题与强调色。

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4)
![Avalonia](https://img.shields.io/badge/Avalonia-11.3.23-8B3DFF)
![FluentAvaloniaUI](https://img.shields.io/badge/FluentAvaloniaUI-2.4.1-6A5ACD)
![Platform](https://img.shields.io/badge/platform-Windows%20·%20macOS%20·%20Linux-2E7D32)

---

## 功能特性

- **事项管理**
  - 输入标题后回车或点击「添加」创建待办
  - 复选框勾选完成 / 取消完成，完成项自动添加删除线并半透明显示
  - 单条删除
  - 一键清除所有已完成事项
- **筛选视图**：全部 / 进行中 / 已完成，下拉切换即时过滤
- **统计摘要**：底部实时显示总数、待完成数、已完成数
- **本地持久化**：所有事项自动保存到本地 JSON 文件，重启应用后自动加载
- **Fluent 界面**：卡片式列表、圆角、主题画刷；跟随系统明暗主题与系统强调色
- **空状态提示**：列表为空时展示引导文案
- **跨平台**：同一套代码可发布到 Windows、macOS、Linux 桌面端

## 技术栈

| 层级 | 技术 / 包 | 版本 | 用途 |
| --- | --- | --- | --- |
| 运行时 | .NET | 8.0 (net8.0) | 应用运行时与 SDK |
| UI 框架 | Avalonia | 11.3.23 | 跨平台 XAML UI 框架 |
| 桌面宿主 | Avalonia.Desktop | 11.3.23 | 桌面应用生命周期与平台集成 |
| 渲染 | Avalonia.Skia | 11.3.23 | Skia 渲染后端 |
| 字体 | Avalonia.Fonts.Inter | 11.3.23 | Inter 字体（`WithInterFont()` 所在包） |
| 主题控件 | FluentAvaloniaUI | 2.4.1 | WinUI 风格 Fluent 主题与扩展控件库 |
| 序列化 | System.Text.Json | 内置 | 本地 JSON 读写 |
| MVVM | 自研轻量基类 | — | `ObservableObject` / `RelayCommand`，无第三方 MVVM 框架依赖 |

> 说明：旧 NuGet 包名 `FluentAvalonia` 已废弃；Avalonia 11 对应的包为 `FluentAvaloniaUI`。主题只需引入 `FluentAvaloniaTheme`，无需再叠加 `FluentTheme` 或 `SimpleTheme`，否则可能产生样式冲突。

## 环境要求

- **从源码运行 / 编译**：需要安装 [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- **直接运行 exe**：使用发布好的自包含包即可，目标机器无需安装 .NET 运行时

## 快速开始

### 方式一：从源码运行

在项目根目录（包含 `TodoApp.csproj` 的目录）下执行：

```bash
dotnet restore
dotnet run
```

也可以用 Visual Studio、Rider 或 VS Code 打开 `TodoApp.csproj` 后直接调试运行。

### 方式二：使用发布好的 exe

解压 `TodoApp-win-x64.zip`，双击其中的 `TodoApp.exe` 即可启动，无需额外安装运行时。

## 打包发布

发布为 **Windows x64 自包含单文件**（内嵌 .NET 运行时与 SkiaSharp 原生库）：

```bash
dotnet publish -c Release -r win-x64 --self-contained true \
  -p:PublishSingleFile=true \
  -p:IncludeNativeLibrariesForSelfExtract=true \
  -p:CompressionEnabled=true \
  -p:DebugType=none \
  -o publish/win-x64
```

产物位于 `publish/win-x64/TodoApp.exe`。

如需其他平台，替换运行时标识符（RID）即可：

| 目标平台 | RID |
| --- | --- |
| Windows x64 | `win-x64` |
| Windows ARM64 | `win-arm64` |
| macOS Intel | `osx-x64` |
| macOS Apple Silicon | `osx-arm64` |
| Linux x64 | `linux-x64` |

## 项目结构

```
TodoApp/
├── TodoApp.csproj                      # 项目文件与 NuGet 依赖
├── Program.cs                          # 应用入口（Avalonia 构建器配置）
├── App.axaml                           # 应用主题（FluentAvaloniaTheme）
├── App.axaml.cs                        # 应用生命周期，创建主窗口
├── Models/
│   └── TodoItem.cs                     # 可序列化的数据模型
├── ViewModels/
│   ├── ObservableObject.cs             # INotifyPropertyChanged 基类
│   ├── RelayCommand.cs                  # ICommand 轻量实现
│   ├── TodoItemViewModel.cs             # 单条事项的视图模型
│   └── MainViewModel.cs                 # 主窗口视图模型（增删改、筛选、统计、持久化）
├── Views/
│   ├── MainWindow.axaml                 # 主窗口界面（XAML）
│   └── MainWindow.axaml.cs              # 界面交互（回车快捷添加等）
├── Services/
│   └── TodoStorage.cs                  # JSON 本地读写服务
└── Converters/
    ├── BoolToTextDecorationsConverter.cs  # 完成 → 删除线
    └── BoolToOpacityConverter.cs           # 完成 → 半透明
```

### 分层职责

- **Views**：只负责界面展示与转发用户交互，不写业务逻辑。
- **ViewModels**：持有界面绑定状态与命令，完成筛选、统计等业务逻辑。
- **Models**：纯数据结构，可直接序列化到本地文件。
- **Services**：封装文件读写等外部基础设施，对视图模型屏蔽存储细节。

## 数据存储

事项以 JSON 格式保存在系统应用数据目录下的 `TodoApp/todo-items.json`：

| 系统 | 存储路径 |
| --- | --- |
| Windows | `%AppData%\TodoApp\todo-items.json`（即 `C:\Users\<用户>\AppData\Roaming\TodoApp\`） |
| macOS | `~/Library/Application Support/TodoApp/todo-items.json` |
| Linux | `~/.config/TodoApp/todo-items.json` |

单条事项的数据字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `Id` | string | 唯一标识（GUID，无连字符） |
| `Title` | string | 事项标题 |
| `IsCompleted` | bool | 是否已完成 |
| `CreatedAt` | DateTime | 创建时间 |

> 读取本地文件失败时应用会回退为空列表，不会阻断界面使用；写入失败会静默忽略，下次启动再尝试加载。

## 实现要点

- **命令绑定**：`RelayCommand` 将按钮点击、回车操作绑定到 ViewModel 方法；输入框为空时「添加」命令自动不可用。
- **筛选机制**：`AllItems` 为全量数据源，`FilteredItems` 根据当前筛选条件重建；事项完成状态变化时自动刷新列表与统计。
- **属性通知**：`ObservableObject` 提供 `SetProperty` 统一触发 `PropertyChanged`，驱动界面更新。
- **值转换器**：完成状态通过两个转换器分别映射为删除线文本修饰与透明度，避免在 ViewModel 中混入 UI 类型。

## 常见问题

**Q：双击 exe 时 Windows 提示「未知发布者 / SmartScreen」？**
程序未做数字签名，点击「更多信息 → 仍要运行」即可。

**Q：首次启动 exe 较慢？**
自包含单文件首次运行会把内嵌的原生库（如 SkiaSharp）解压到临时目录，之后启动为正常速度。

**Q：我的待办数据保存在哪里？**
见上文「数据存储」一节，删除 `todo-items.json` 会清空本地记录。

**Q：如何手动切换深色 / 浅色主题？**
当前版本跟随系统主题；可在后续版本中加入主题手动切换开关。

## 可扩展方向

- 事项优先级（高 / 中 / 低）与截止日期、到期提醒
- 双击编辑事项标题、拖拽排序
- 分类清单 / 多列表管理
- 深色 / 浅色 / 跟随系统的手动主题切换
- 单元测试（ViewModels 不依赖界面，可直接覆盖测试）
- 打包安装程序（MSIX / 安装向导）

## 说明

本项目为AI生成
