# Light LDAP

轻量级 LDAP 认证服务器，自带 Web 管理界面，为自托管服务提供统一的账号认证与单点登录（SSO）。

## 特点

- 单进程、资源占用低，比 OpenLDAP 更容易上手：用户、用户组、密码都在 Web 界面里维护，不需要手写 LDIF
- 标准 LDAP 协议，可直接对接 Nextcloud、Gitea、Jellyfin、Authelia 等支持 LDAP 的应用
- 首次启动根据环境变量自动创建管理员账号与数据目录，无需额外初始化步骤

## 安装

在 1Panel 应用商店中安装 **Light LDAP**，按表单填写：

| 字段 | 说明 |
|------|------|
| LDAP 端口 | LDAP 协议对外端口，默认 `3890` |
| Web 端口 | Web 管理界面对外端口，默认 `17170` |
| LDAP Base DN | 目录树根，默认 `dc=example,dc=com`，建议改成自己的域名，**首次启动后不要再改** |
| 管理员密码 | 内置管理员 `admin` 的初始密码，首次启动时写入数据库 |
| JWT 密钥 | 会话签名密钥，保持表单随机生成的值即可 |

## 首次登录

1. 访问 `http://<服务器IP>:<Web端口>`（默认 `http://<服务器IP>:17170`）
2. 用户名固定为 `admin`，密码为表单里填写的**管理员密码**
3. 登录后在 Web 界面中创建用户与用户组，再按下面的参数把其它应用接进来

## 对接其它应用

| 参数 | 值 |
|------|-----|
| 服务器地址 | `<服务器IP>`（与 1Panel 同网络的应用可用容器名 `lldap`） |
| 端口 | LDAP 端口，默认 `3890` |
| Base DN | 表单里填写的 Base DN，例如 `dc=example,dc=com` |
| Bind DN | `uid=admin,ou=people,<Base DN>` |
| Bind 密码 | 管理员密码 |
| 用户过滤器 | `(uid=%s)`，部分应用需要 `(&(uid=%s)(objectClass=person))` |
| 用户 DN 模板 | `uid={username},ou=people,<Base DN>` |
| 用户组 DN 模板 | `cn={group},ou=groups,<Base DN>` |

> 用户位于 `ou=people`，用户组位于 `ou=groups` 之下，这是 Light LDAP 的固定结构。

## 持久化数据

应用数据持久化在 `./data` 目录（映射到容器内 `/data`），包含用户、用户组、密码等 SQLite 数据库内容与会话密钥。**升级或迁移前请备份该目录**：Base DN 与管理员密码一旦初始化，都存放在这里。

## 环境变量

| 变量 | 说明 | 来源 |
|------|------|------|
| `PANEL_APP_PORT_LDAP` | LDAP 对外端口，映射容器 `3890` | 安装表单 |
| `PANEL_APP_PORT_HTTP` | Web 界面对外端口，映射容器 `17170` | 安装表单 |
| `LLDAP_LDAP_BASE_DN` | 目录树根 DN | 安装表单 |
| `LLDAP_LDAP_USER_PASS` | 内置管理员密码 | 安装表单 |
| `LLDAP_JWT_SECRET` | 会话签名密钥 | 安装表单 |
| `TZ` | 容器时区，默认 `Asia/Shanghai` | compose 默认值 |

上游还支持一批未在表单中暴露的变量（邮件通知、外部数据库、密钥种子等），有需要可在 compose 的 `environment` 中自行追加。

## 注意事项

- **Base DN 初始化后不要修改**：改动会让已对接应用里的用户 DN 全部失配。
- LDAP 端口是明文协议，公网暴露前请自行确认是否需要 TLS；Web 界面建议通过 1Panel 反向代理 + HTTPS 对外提供。
- 修改端口时，表单里的端口和任何防火墙/反向代理规则要同步调整。

## 官方资源

- 官网 / GitHub：https://github.com/lldap/lldap
- 安装与配置文档：https://github.com/lldap/lldap/blob/main/docs/install.md
