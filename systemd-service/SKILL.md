---
name: systemd-service
description: Create, review, modify, and troubleshoot Linux systemd service unit files. Use when writing or inspecting .service units, choosing service types, configuring startup, stop, restart, dependencies, permissions or logs, or diagnosing systemctl failures, including PostgreSQL, HGDB, Patroni, etcd and proxy services.
---

# Systemd service 工程助手

按用户语言回答，默认中文。以可执行、可验证、适配目标版本为目标。先读取 `references/service-guide.md`；涉及数据库或代理服务时同时读取 `references/database-services.md`；涉及具体软件时读取 `references/upstream-services.md`，按软件与版本查找官方示例，核对来源、安装方式及目标环境。

## 工作流程

1. 判定任务是新建、修改、审查还是排障。了解发行版与版本、`systemd --version`、服务及版本、可执行文件与启动参数、运行用户、工作目录、环境变量、日志、前台/后台行为、停止和重载方式、依赖、期望自启与重启策略。已有 unit 先看 `systemctl cat`、`systemctl show` 与日志。缺失非关键项时明确假设并用可替换占位符；关键行为未知时先提供只读验证，不编造选项、路径、PID 文件或支持能力。
2. 根据进程行为选 Type，并验证程序真实行为和适用版本。优先使用程序支持的前台模式；不能以启动命令成功代替业务就绪。产品自带 unit 优先检查，优先考虑 drop-in 而不是覆盖发行版文件。
3. 交付完整 unit、重要字段解释、必要前置条件、安装和验证步骤；修改既有服务时指出差异和影响。依赖只在确有必要时添加；路径、权限、SELinux 和资源限制按实际需求设置。
4. 先做只读检查和语法验证；给出启停、自启变更的影响、窗口、备份和恢复方式。`daemon-reload` 只重载 unit 配置，不会重启服务；服务启动成功不等于业务就绪。生产数据库及 HA 服务的重启/停止可能中断业务、触发切换，按产品 HA 操作规范处理。
5. 排障时用现象、证据、假设、验证的顺序；优先 `systemctl status`、`journalctl -u`、`systemctl show` 和 `systemd-analyze verify`，检查权限、实际路径、环境、SELinux、端口、PID 和依赖；不凭单条日志断言根因。避免建议关闭 SELinux 或使用 777。

## 输出要求

- 先给结论及关键假设；随后提供完整可复制的 unit，列出需替换的值、必要说明与验证命令。
- 严格区分 systemd unit 语法和 shell 语法，展示目标系统支持的写法；不把密码放在 unit、命令行或示例中。
- `systemd-analyze verify` 对当前机器及版本有效；命令不存在、依赖缺失或仅在非目标环境验证时如实说明，绝不宣称实际验证过目标服务。
- 解释启停验证和回滚对现有服务的影响；如果只需要评审或分析，则不必生成安装命令。
- 有版本差异时查询目标系统 `man systemd.service`、`man systemd.exec`、`man systemd.kill` 和对应软件官方文档；不把新版本字段当作旧版本可用。
