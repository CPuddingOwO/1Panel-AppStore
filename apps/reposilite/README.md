# Reposilite

轻量级自托管 Maven 仓库，支持代理远程仓库、私有仓库和 Web 管理界面。

## 首次启动配置

Reposilite 的访问令牌只能通过 CLI/Console 创建，首次启动需要特殊处理：

### 1. 创建临时管理员

在 1Panel 应用安装表单中，**Reposilite 启动参数** 字段填入：
```
--token admin:secret
```

### 2. 启动应用

点击安装，等待容器启动完成。

### 3. 登录 Web 界面

访问 `http://<你的服务器IP>:<配置的端口>`，使用以下凭据登录：
- 用户名：`admin`
- 密码：`secret`

### 4. 生成真实管理令牌

1. 登录后进入 **Console** 标签页
2. 执行命令：
   ```
   token-generate admin m
   ```
   > `m` 表示 management（管理权限）
3. 系统会生成一个强密码，**请务必复制保存**

### 5. 移除临时管理员

1. 在 1Panel 中停止 Reposilite 应用
2. 编辑应用，将 **Reposilite 启动参数** 清空（删除 `--token admin:secret`）
3. 重新启动应用

### 6. 使用真实令牌登录

用第 4 步生成的用户名和密码登录，后续所有配置（仓库、用户、前端等）都在 Web 界面的 **Settings** 里操作。

## 持久化数据

应用数据持久化在 `./data` 目录下，包含：

| 路径 | 说明 |
|------|------|
| `configuration.cdn` | 本地配置文件（端口、SSL、数据库等） |
| `repositories/` | Maven 仓库数据 |
| 数据库文件 | Token、统计、共享配置等 |
| `logs/` | 日志文件 |

## 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `JAVA_OPTS` | JVM 启动参数 | `-Xmx128M` |
| `REPOSILITE_OPTS` | Reposilite 启动参数 | 空 |

## 常用 REPOSILITE_OPTS 参数

| 参数 | 说明 |
|------|------|
| `--token name:secret` | 创建临时管理员（仅首次使用） |
| `--local-configuration=/app/data/custom.cdn` | 指定自定义配置文件 |
| `--working-directory=/app/data` | 指定工作目录 |

## 官方文档

- 官网：https://reposilite.com/
- 文档：https://reposilite.com/guide/
- GitHub：https://github.com/dzikoysk/reposilite
