# Service 设计与验证参考

## 类型和进程生命周期

- 前台常驻：`Type=simple` 通用；目标版本支持时 `Type=exec` 可以在 exec 成功后报告启动，但不证明业务就绪。进程不应自行后台化；wrapper 应以 `exec` 替换自身。
- 自行 daemonize：`Type=forking`；确认父进程退出后的主进程追踪，程序确实生成且维护 PID 文件时才配置 `PIDFile=`。
- `Type=notify` 仅用于程序确实发送 `READY=1`；`oneshot` 适合执行后退出的任务，`RemainAfterExit=yes` 表示执行完成后保持 active，并不代表进程仍运行。
- `ExecStart=` 不是 shell：管道、重定向、`&&`、通配符和 shell 展开不能按普通 shell 假定有效；需要 shell 时显式调用并审慎引用，优先脚本或直接执行程序。不要假设读取用户的交互式 profile。使用绝对路径并配置所需环境。
- `ExecStop=` 要同步等待进程停止；未配置时 systemd 按 KillMode 和信号策略停止。`KillMode=process` 或 `none` 不应作为通用方案，可能残留进程；有产品证据才使用并解释后果。数据库和多进程服务须核对产品推荐的停止语义与超时，不能机械替换默认值。

## 依赖、重启和权限

- `After=` 仅排序，不会主动拉起依赖；`Requires=` 或 `Wants=` 表示依赖，分别评估失败影响。`network-online.target` 不保证远端数据库可用，只有实际需要时使用并验证 wait-online。
- 长期运行服务可按需求选择 `Restart=on-failure`、`RestartSec=` 和启动速率限制，不能一概用于所有任务；`oneshot` 的重启选项受版本和类型限制。启动频率限制字段的名称及位置核对目标版本。
- 采用专用非 root 用户、适当目录权限和明确的 `WorkingDirectory=`；按产品需要设置 `LimitNOFILE=` 等。不要盲目应用加固选项：保护路径、网络和 namespace 限制可能阻断服务。
- unit 环境变量和 `EnvironmentFile=` 不适合作为秘密管理机制，优先产品支持的凭据机制并核对目标版本；不要在示例中写入密码。

## 最小验证与回滚

以 `example.service` 和实际 unit 路径替换下列占位名称。变更前记录 `systemctl cat example.service`、`systemctl is-enabled example.service`、`systemctl is-active example.service`，备份待修改文件；运行中服务可能因后续重启中断。安装到 `/etc/systemd/system/` 需适当权限；drop-in 使用 `systemctl edit` 或等价方式，记录原有覆盖配置。

```bash
systemd --version
systemd-analyze verify /etc/systemd/system/example.service
sudo systemctl daemon-reload
systemctl cat example.service
systemctl show example.service -p Type -p MainPID -p ExecStart -p ExecStop -p KillMode -p Restart -p FragmentPath -p DropInPaths
```

依据用户授权及维护窗口执行 `systemctl start` 或 `restart`；`enable` 改变开机自启，不等同于启动。之后检查 `systemctl status example.service --no-pager -l`、`journalctl -u example.service -b --no-pager -n 100` 和产品级健康检查；systemd active 与应用可用分开确认。

回滚：恢复原文件或撤销本次新增 drop-in，执行 `daemon-reload`；要让已运行进程应用旧配置需按维护窗口重启，恢复原自启状态。区分原文件不存在时删除本次新增文件与恢复已有文件；避免覆盖其他并发变更。

官方文档：https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html 、https://www.freedesktop.org/software/systemd/man/latest/systemd.kill.html 、https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html
