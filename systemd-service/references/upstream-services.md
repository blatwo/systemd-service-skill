# 上游与发行版 Service 参考索引

用于按需查阅和比较，不是直接可部署的模板。检索日期：2026-11-13。链接中的 main、master、current、latest 会变化，不能代表用户已安装的版本；优先切换到匹配的 tag、版本文档或用户实际安装包。软件未指定版本时，明确这一限制。网页不可访问时说明未核实，不编造 unit。

## Patroni

- 上游示例：[patroni.service](https://github.com/patroni/patroni/blob/master/extras/startup-scripts/patroni.service)；[上游说明](https://github.com/patroni/patroni/blob/master/extras/startup-scripts/README.md)。来源是上游 master 示例，不是通用生产基线，上游说明曾在 CentOS 7.1 + pip 安装环境测试。
- 重点核对 `Type=notify`、环境文件、路径、`ExecReload`、`TimeoutSec` 和目标版本进程行为。上游使用 `KillMode=process`，目的是让 Patroni 自己有序关闭 PostgreSQL；这是有产品上下文的例外，异常情况下可能残留子进程，必须依据目标版本和集群运行方式评估，不能推广到其他服务。

## etcd

- 上游示例：[etcd.service](https://github.com/etcd-io/etcd/blob/main/contrib/systemd/etcd.service)。来源为上游 main，针对目标 etcd 版本检查同路径文件和通知支持。
- 示例使用 `Type=notify`、专用用户和 `Restart=always`；路径、节点名、数据目录、集群配置、网络依赖、文件句柄限制均须按部署核实。不要把示例默认值当成三节点集群配置。

## vip-manager

- 上游示例：[vip-manager.service](https://github.com/cybertec-postgresql/vip-manager/blob/master/vip-manager.service)；[项目文档](https://github.com/cybertec-postgresql/vip-manager)。来源是上游 master，需匹配实际版本和安装包。
- 核对程序与配置路径、DCS、网卡、VIP、所需网络权限及启动顺序；示例的 `Before=patroni.service` 是排序关系，并不创建依赖或确保 DCS 与 VIP 就绪，须按实际架构判断。

## PostgreSQL

- 官方手册示例：[Starting the Database Server](https://www.postgresql.org/docs/current/server-start.html)。该链接为 current，可切换到目标主版本。它是官方文档中的示例，不一定是发行版软件包的 unit。
- 官方示例使用 `Type=notify` 时要求服务端构建支持 systemd；其 `KillMode=mixed`、信号和无限超时应结合实际关闭/恢复要求审查。受 Patroni 等管理的数据库不另行启用独立 unit，除非集群方案明确要求。

## Pgpool-II

- [官方 RPM 安装与服务管理文档](https://www.pgpool.net/docs/latest/en/html/install-rpm.html)说明软件包及 `pgpool.service`，可切换对应版本；它不等同于已经核实了完整的上游 unit 文件。
- 先在目标机检查 `systemctl cat pgpool.service`、`systemctl show pgpool.service -p FragmentPath -p DropInPaths` 与 `rpm -ql <实际包名>`。确认安装包、版本、前台选项、配置、PCP、VIP 和停止方式；不要把 hgproxy 的参数视为 Pgpool-II 的通用参数。

## Apache Tomcat

- [Apache Tomcat 启停说明](https://github.com/apache/tomcat/blob/main/RUNNING.txt)是上游程序运行说明，不是官方 systemd unit。针对目标版本查看对应 release/tag。
- [Debian bookworm tomcat10 包文件列表](https://packages.debian.org/bookworm/all/tomcat10/filelist)列出发行版的 `tomcat10.service`；属于 Debian 打包，不是 Apache 上游通用 unit，不能套用到 Rocky 或手工安装。
- 根据实际安装方式查 `systemctl cat tomcat10.service` 或对应 unit、包版本和 Java 环境；确认前台启动、`CATALINA_HOME`/`CATALINA_BASE`、账户及停止方式。

## 使用顺序

1. 优先读取目标机原有 unit 和 drop-in；记录软件版本、系统版本、安装来源、管理者及生效路径。
2. 查匹配版本的上游或发行版文件，标记样例的假设和许可；对不明确的版本或文件说明未验证。引用路径而非把变动的 main/master 内容当成固定版本。
3. 对照已有配置逐项判断，保留产品必要的特殊生命周期行为；解释为何采用或调整。新生成的内容仅为适配方案，不称为原版文件。
4. 执行目标环境的语法和产品健康检查。特别关注 Patroni/etcd/VIP 的启动顺序、仲裁、故障切换和维护窗口。
