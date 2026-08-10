# Codex Pulse

Codex Pulse 是一个基于 Tauri 的轻量 macOS 菜单栏应用，用来集中查看本机与远程服务器上的 Codex CLI 会话：

- 正在进行：本轮任务尚未结束，并且 Codex 进程仍持有会话文件；
- 等待处理：存在等待选择、审批或确认的工具调用；
- 新完成：任务刚刚完成，保留到点击“已查看”；
- 执行失败：收到错误/中止事件，或运行中的会话意外停止。

应用默认显示会话最后一条用户提示词，也可以在右上角菜单切换为原始标题。关注页按“进行中 / 新完成 / 失败”的配置数量控制窗口高度，多余内容自动隐藏并可切换到对应分类查看。

本地会话每 2 秒只读扫描一次，远程服务器每 15 秒通过已有 SSH 配置扫描一次。应用不会上传会话数据，也不会持有 Codex 的 SQLite 数据库或 JSONL 会话文件。

## 运行

要求：macOS 14+、Node.js 22+、Rust stable、已安装 Codex CLI。

```bash
npm install
npm start
```

通过 Homebrew 安装 `rustup` 时，需要确保工具链目录位于 PATH：

```bash
export PATH="/opt/homebrew/opt/rustup/bin:$PATH"
```

## 打包

生成可直接打开的 `.app`：

```bash
npm run package
```

生成 DMG 安装包：

```bash
npm run make
```

产物位于：

```text
src-tauri/target/release/bundle/macos/Codex Pulse.app
src-tauri/target/release/bundle/dmg/Codex Pulse_0.2.0_aarch64.dmg
```

也可以使用 `./scripts/build-app.sh` 将 `.app` 复制到 `dist/Codex Pulse.app`。

首次点击“在终端中继续”时，macOS 可能请求允许 Codex Pulse 控制 Terminal。此权限只用于执行 `codex resume <session-id>`。

## YOLO 模式

右上角菜单可以开启 YOLO 模式。该设置默认关闭并保存在本机；开启后，继续命令会使用：

```bash
codex resume --dangerously-bypass-approvals-and-sandbox <session-id>
```

此参数会跳过确认并关闭 Codex 沙箱，只应在工作目录已经可靠隔离时开启。

## 测试

```bash
npm test
npm run check
```

测试覆盖状态判定、提示词提取、完成确认、命令转义、设置迁移、SSH 配置解析和远程数据规范化。

## 实现边界

独立运行的 Codex CLI 不会把实时 app-server 事件转发给监控进程，因此 Codex Pulse 采用只读检测：结合 JSONL 事件、审批策略、会话文件占用和子进程状态判断任务状态。

本地扫描与状态管理位于 `src-tauri/src/`，远程适配会把 `src/remote/remote_scanner.py` 编译进 Rust 二进制，并通过 SSH 标准输入发送到服务器执行，不在远程服务器写入脚本文件。

## 许可证

Codex Pulse 使用 [MIT License](LICENSE) 开源。
