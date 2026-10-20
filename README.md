# Nyxuri Scripts

用于 Nyxuri、Niri 和 Noctalia 环境的实用脚本集合。

## Steam 游戏图标同步

`steam-game-icon-sync` 会扫描 Steam 游戏库，根据 Steam 缓存的艺术图生成统一的圆角图标和桌面条目，使 Niri Dock 中的 Steam 游戏拥有正确的名称、图标和窗口标识。

仓库包含两个脚本：

- `steam-game-icon-sync`：扫描游戏库并生成图标与桌面条目。
- `steam-game-icon-round`：为主脚本提供圆角处理能力，也可以单独使用。

### 依赖

必需：

- Bash 4 或更高版本。
- ImageMagick 7，提供 `magick` 命令。
- Steam 及其游戏库缓存。

可选：

- `niri` 和 `jq`：自动识别 Linux 原生游戏的实际窗口类别。
- `noctalia`：同步后自动刷新 Noctalia 配置和 Dock。
- `update-desktop-database`：刷新桌面条目数据库。

Arch Linux 可安装：

```bash
sudo pacman -S imagemagick jq
```

### 使用

在仓库目录直接运行：

```bash
chmod +x steam-game-icon-sync steam-game-icon-round
./steam-game-icon-sync --dry-run
```

确认输出无误后执行实际同步：

```bash
./steam-game-icon-sync
```

也可以把两个脚本安装到 `~/.local/bin`，之后在任何目录直接调用：

```bash
install -Dm755 steam-game-icon-sync ~/.local/bin/steam-game-icon-sync
install -Dm755 steam-game-icon-round ~/.local/bin/steam-game-icon-round
steam-game-icon-sync --dry-run
```

如果 `~/.local/bin` 已在 `PATH` 中，安装后可以直接运行 `steam-game-icon-sync`。

### 同步内容

每次同步会：

- 扫描主 Steam 库以及 `libraryfolders.vdf` 中的其他游戏库。
- 排除 Proton、Steam Linux Runtime 和公共组件。
- 从 Steam `librarycache` 获取游戏艺术图。
- 生成 256×256 图标并添加透明圆角。
- 创建 `steam_app_<AppID>.desktop`。
- 写入正确的 `StartupWMClass` 和图标路径。
- 保留已有桌面条目的中文名称。
- 更新桌面数据库，并请求 Noctalia 重新加载配置和 Dock。
- 自动检测正在运行的 Niri 游戏窗口标识。

### 新游戏流程

Windows/Proton 游戏通常安装后直接同步即可：

```bash
steam-game-icon-sync
```

Linux 原生游戏如果同步后仍显示默认图标，按以下步骤处理：

1. 启动一次游戏。
2. 游戏保持运行，执行 `steam-game-icon-sync`。
3. 脚本会缓存实际窗口标识，例如：

   ```text
   322330	dontstarve_steam_x64
   ```

缓存后关闭游戏，再次进行同步也会继续使用该窗口标识。

### 调整圆角

默认圆角半径是 256×256 图片上的 56 像素。需要更小或更大的圆角时：

```bash
steam-game-icon-sync --radius 40
```

也可以只处理一个或多个独立图标：

```bash
steam-game-icon-round --radius 40 图标.png
```

### 生成文件位置

- 桌面条目：`~/.local/share/applications/`
- 圆角图标：`~/.local/share/icons/hicolor/256x256/apps/`
- 未圆角的原始封面：`~/.local/share/steam-game-icons/sources/`
- 自动检测的窗口类别缓存：`~/.local/share/steam-game-icons/wmclass-auto.tsv`
- 可选的手工覆盖表：`~/.config/steam-game-icon-sync/wmclass-overrides.tsv`

手工覆盖表使用 AppID 和实际窗口类别，以 Tab 分隔：

```text
AppID	实际窗口类别
```

只有自动检测失败或窗口标题与游戏名称严重不一致时才需要手工添加。

### 常见情况

- `no cached artwork`：先在 Steam 库中打开游戏页面，让 Steam 缓存艺术图，再运行脚本。
- 游戏图标没有立即刷新：再次运行同步脚本；必要时关闭并重新打开游戏。
- 已有非脚本生成的同名 `.desktop`：脚本会跳过，避免覆盖手工配置。
- 游戏更新后窗口类别变化：启动游戏并重新同步一次。

脚本不会修改游戏文件、存档或 Steam 游戏安装内容。
