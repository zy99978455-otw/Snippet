# Snippet

Snippet 是一个基于 Go 语言构建的文本分享 Web 应用。

这个项目旨在展示 Go 语言在 Web 开发中的核心概念，包括路由处理、中间件使用、数据库集成、HTML 模板渲染、表单处理以及 Session 会话管理。

## ✨ 当前功能特性

* **发布 Snippet**：用户可以创建包含标题、内容和过期时间的文本片段。
* **查看 Snippet**：通过 ID 查看已发布的文本内容。
* **数据持久化**：使用 MySQL 存储所有数据。
* **中间件架构**：使用 `alice` 管理中间件链（Panic 恢复、请求日志、安全头）。
* **表单处理**：
    * 自动将 HTML 表单数据解码为 Go 结构体 (`go-playground/form`)。
    * 自定义表单验证逻辑 (`validator` 包)。
* **Session 管理**：
    * 基于 MySQL 的服务端 Session 存储 (`alexedwards/scs`)。
    * 支持 Flash Messages（一次性通知消息）。
* **动态 HTML**：使用 Go 标准库 `html/template` 进行渲染，包含公共布局和局部模板。
* **静态文件服务**：处理 CSS、JS 和图片资源。

## 🛠️ 技术栈

* **语言**: Go (Golang) 
* **数据库**: MySQL
* **关键依赖库**:
    * [`github.com/go-sql-driver/mysql`](https://github.com/go-sql-driver/mysql): MySQL 驱动
    * [`github.com/justinas/alice`](https://github.com/justinas/alice): 中间件链管理
    * [`github.com/go-playground/form`](https://github.com/go-playground/form): 表单解码
    * [`github.com/alexedwards/scs/v2`](https://github.com/alexedwards/scs): Session 管理

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/zy99978455-otw/Snippet.git
cd snippet
```
### 2. 数据库设置
你需要安装 MySQL 并创建一个名为 snippetbox 的数据库。然后运行以下 SQL 脚本来创建必要的表（snippets 和 sessions）：

```sql
-- 切换数据库
USE snippetbox;

-- 1. 创建 snippets 表
CREATE TABLE snippets (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL
);

CREATE INDEX idx_snippets_created ON snippets(created);
-- 2. 添加一些初始测试数据
INSERT INTO snippets (title, content, created, expires) VALUES (
    'An old silent pond',
    'An old silent pond...\nA frog jumps into the pond,\nsplash! Silence again.',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'Over the wintry forest',
    'Over the wintry forest,\nwinds howl in rage\nwith no leaves to blow.',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'First autumn morning',
    'First autumn morning\nthe mirror I stare into\nshows my father''s face.',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 7 DAY)
);
-- 3. 创建 sessions 表 (用于 scs 库)
CREATE TABLE sessions (
    token CHAR(43) PRIMARY KEY,
    data BLOB NOT NULL,
    expiry TIMESTAMP(6) NOT NULL
);

CREATE INDEX sessions_expiry_idx ON sessions (expiry);
```
### 3. 创建数据库用户
为了安全起见，建议创建一个专门的 web 用户：
```sql
CREATE USER 'web'@'localhost' IDENTIFIED BY 'pass';
GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'localhost';
```
### 4. 运行应用
确保依赖已下载：
```Bash
go mod tidy
```
运行 Web 服务器：
```Bash
go run ./cmd/web
```
默认情况下，服务器将在 `:4000` 端口启动。 你也可以通过命令行参数修改配置：
```Bash
go run ./cmd/web -addr=":8080" -dsn="web:pass@/snippetbox?parseTime=true"
```
访问浏览器：[http://localhost:4000](http://localhost:4000)

📂 项目结构
```Plaintext
snippetbox/
├── cmd/
│   └── web/
│       ├── main.go        # 应用入口，依赖注入
│       ├── handlers.go    # HTTP 处理函数
│       ├── routes.go      # 路由定义 (ServeMux)
│       ├── middleware.go  # 中间件逻辑
│       ├── helpers.go     # 辅助函数 (如 render, serverError)
│       └── templates.go   # 模板缓存与数据结构
├── internal/
│   ├── models/            # 数据库模型与操作
│   └── validator/         # 表单验证逻辑
├── ui/
│   ├── html/              # HTML 模板文件
│   └── static/            # 静态资源 (CSS, JS, Images)
├── go.mod
└── README.md
```
***

本项目由 **LeoZhao(https://github.com/zy99978455-otw)** 开发与维护。

如果你觉得这个项目对你有帮助，欢迎给个 Star ⭐️！

## 🙏 致谢

本项目是基于 **Alex Edwards** 的著作 [《Let's Go》](https://lets-go.alexedwards.net/) 进行开发的练习项目。

特别感谢原作者提供了优秀的 Go Web 开发教程，帮助构建了清晰的项目架构和最佳实践。

## 📄 许可证

本项目采用 **MIT 许可证**。详情请参阅 [LICENSE](LICENSE) 文件。