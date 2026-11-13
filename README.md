# Systemd Service Skill

帮助 AI 助手编写、审查和排查 Linux systemd service，兼顾通用服务与数据库、集群场景。

## 功能

- 根据进程运行方式选择服务类型，编写或审查 unit。
- 检查依赖、运行用户、环境变量、重启、停止方式和日志。
- 提供语法检查、服务验证、变更影响及回滚方法。
- 关注 PostgreSQL、HGDB、Patroni、etcd、pgpool-II 和 hgproxy 的管理边界与版本差异。
- 按需查阅 Patroni、etcd、vip-manager、PostgreSQL 的上游示例，以及 Pgpool-II、Tomcat 的官方文档和发行版打包线索。

## 使用

将 `systemd-service/` 放入支持 SKILL.md 格式的工具的技能目录，然后按工具要求刷新或重启。以 Codex 用户级技能目录为例：

```bash
git clone https://github.com/blatwo/systemd-service-skill.git
mkdir -p ~/.agents/skills
cp -R systemd-service-skill/systemd-service ~/.agents/skills/
```

各产品的技能目录和加载方式可能不同，请以相应产品文档为准。也可以直接把 `systemd-service/` 作为项目内的 Skill 使用。

示例请求：

- 帮我为 Rocky Linux 上的前台服务写一个 systemd unit。
- 审查这个 hgproxy service 的停止、重启和日志配置。
- Patroni 服务启动失败，帮我根据状态和日志排查。
- 解释 `Type=forking`、`PIDFile=` 和 `KillMode=` 的适用条件。

## 目录

```text
systemd-service/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── service-guide.md
    ├── database-services.md
    └── upstream-services.md
```

## 官方参考

[上游与发行版 Service 参考索引](systemd-service/references/upstream-services.md) 标明示例来源、适用边界和核对要点。链接保留上游来源，实际使用时应查阅与目标软件版本一致的文件；Pgpool-II 和 Tomcat 条目提供官方文档或发行版打包线索，并不冒称已核实上游完整 unit。

## 注意事项

生成的 unit 应结合目标系统版本和实际软件文档核对，在测试环境验证。生产环境的重启、停止和自启变更可能中断服务或触发 HA 切换。Skill 不会自动验证目标主机，也不应假设应用支持某个启动参数。

## 参考

- [systemd.service](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html)
- [systemd.kill](https://www.freedesktop.org/software/systemd/man/latest/systemd.kill.html)
- [systemd.unit](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html)
