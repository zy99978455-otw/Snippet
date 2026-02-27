# Snippet

Snippet 是一个基于 Go 语言构建的文本分享 Web 应用。

这个项目旨在展示 Go 语言在 Web 开发中的核心概念，包括路由处理、中间件使用、数据库集成、HTML 模板渲染、表单处理以及 Session 会话管理。

## ✨ 当前功能特性

* **发布 Snippet**：用户可以创建包含标题、内容和过期时间的文本片段。
* **查看 Snippet**：通过 ID 查看已发布的文本内容。
* **用户认证与授权 (Authentication & Authorization)**：
  * 支持用户注册、登录、安全注销。
  * 基于 Session 的路由保护（未登录用户无法发布 Snippet）。
* **极致安全加固 (Security Enhancements)**：
  * **HTTPS 支持**：自动启用 TLS 加密，强制使用高性能椭圆曲线（X25519, P256）优化 TLS 握手。
  * **防 CSRF 攻击**：通过 `justinas/nosurf` 实现基于 Token 的跨站请求伪造防护。
  * **防 Session 固定攻击**：在用户权限变更（登录/注销）时自动轮换 Session ID (`RenewToken`)。
  * **密码安全**：使用 `bcrypt` 算法对用户密码进行哈希加密存储。
  * **服务器防御**：配置 `IdleTimeout`、`ReadTimeout` 和 `WriteTimeout` 防御慢速连接攻击（如 Slowloris）。
* **单文件部署 (Single Binary Deployment)**：
  * 使用 Go 1.16+ 的 `//go:embed` 技术，将 HTML 模板和静态资源（CSS, JS）完美嵌入到编译后的二进制文件中，实现极简部署。
* **高质量代码保障 (Testing)**：
  * 引入完整的单元测试机制（使用 `net/http/httptest` 模拟请求）。
  * 采用优雅的**表格驱动测试 (Table-driven tests)** 模式。
  * 包含自定义的泛型断言辅助包 (`internal/assert`) 以提高测试代码复用率。
* **数据持久化**：使用 MySQL 存储所有数据（Snippets 和 Users）。
* **表单与 Session 管理**：
  * 自动解码 HTML 表单并进行自定义验证 (`validator` 包)。
  * 基于 MySQL 的服务端 Session 存储，支持 Flash Messages。

## 🛠️ 技术栈

* **语言**: Go (Golang) 
* **数据库**: MySQL
* **关键依赖库**:
    * [`github.com/go-sql-driver/mysql`](https://github.com/go-sql-driver/mysql): MySQL 驱动
    * [`github.com/justinas/alice`](https://github.com/justinas/alice): 中间件链管理
    * [`github.com/go-playground/form`](https://github.com/go-playground/form): 表单解码
    * [`github.com/alexedwards/scs/v2`](https://github.com/alexedwards/scs): Session 管理
    * [`golang.org/x/crypto/bcrypt`](https://pkg.go.dev/golang.org/x/crypto/bcrypt): 密码哈希加密
    * [`github.com/justinas/nosurf`](https://github.com/justinas/nosurf): CSRF 防护中间件

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

### 4. 生成 TLS 证书 (HTTPS 必需)
在运行服务器之前，需要生成开发用的自签名证书：

```Bash
# 创建存放证书的目录
mkdir tls
cd tls

# 使用 Go 标准库工具生成证书 (适用于 Linux/macOS/WSL)
go run $(go env GOROOT)/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost

# 返回项目根目录
cd ..
```
### 5. 运行应用
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

### 6. 访问应用
打开浏览器访问：https://localhost:4000

⚠️ 注意：由于使用的是自签名证书，浏览器会提示“连接不安全”。这是正常现象，请点击“高级” -> “继续访问” (Proceed) 即可。
访问浏览器：[http://localhost:4000](http://localhost:4000)

📂 项目结构
```Plaintext
snippetbox/
├── cmd/
│   └── web/
│       ├── main.go        # 应用入口 (HTTPS server, timeouts)
│       ├── handlers.go    # HTTP 处理函数
│       ├── handlers_test.go  # 处理器自动化测试
│       ├── helpers.go     # 辅助函数 (如渲染、错误处理)
│       ├── main.go        # 应用入口 (依赖注入、服务器配置)
│       ├── routes.go      # 路由定义 (ServeMux)
│       ├── middleware.go  # 中间件逻辑
│       ├── middleware_test.go # 中间件自动化测试
│       ├── helpers.go     # 辅助函数 (如 render, serverError)
│       └── templates.go   # 模板缓存与数据结构
│       └── templates_test.go # 模板格式化自动化测试
├── internal/
│   ├── assert/            # 通用泛型测试断言库
│   │   └── assert.go
│   ├── models/            # 数据库模型与操作
│   │   ├── errors.go
│   │   ├── snippets.go
│   │   └── users.go
│   └── validator/         # 表单验证逻辑
├── ui/
│   ├── html/              # HTML 模板文件
│   └── static/            # 静态资源 (CSS, JS, Images)
│   └── efs.go             # 嵌入式文件系统指令 (go:embed)
├── tls/                   # TLS 证书 (不提交到 Git)
├── go.mod
├── README.md              # 英文说明文档
└── README_ZH.md           # 中文说明文档
```
***

本项目由 **LeoZhao(https://github.com/zy99978455-otw)** 开发与维护。

如果你觉得这个项目对你有帮助，欢迎给个 Star ⭐️！

## 🙏 致谢

本项目是基于 **Alex Edwards** 的著作 [《Let's Go》](https://lets-go.alexedwards.net/) 进行开发的练习项目。

特别感谢原作者提供了优秀的 Go Web 开发教程，帮助构建了清晰的项目架构和最佳实践。

## 📄 许可证

本项目采用 **MIT 许可证**。详情请参阅 [LICENSE](LICENSE) 文件。