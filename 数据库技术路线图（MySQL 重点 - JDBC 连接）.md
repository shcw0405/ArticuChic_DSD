# 数据库技术路线图（MySQL 重点 - JDBC 连接）

根据数据模块的需求规格说明书以及您对中等规模数据库偏好使用 MySQL 的要求，并采用 JDBC 进行数据库连接，这里为您提供一份针对数据库部分的技术路线图建议。

这份路线图侧重于核心数据库技术、**JDBC 交互层**以及必要的支持工具。

## 1. 核心数据库引擎

*   **技术:** **MySQL Community Server** (或者 Percona Server for MySQL / MariaDB)。
*   **版本:** 建议使用最新的稳定 LTS 版本（例如 `MySQL 8.0.x` 或更高版本）。
*   **理由:** （同前）满足需求、成熟可靠、社区支持、性能足够、支持必要数据类型。

## 2. 数据库交互层（应用程序端 - 使用 JDBC）

*   **技术:** **JDBC (Java Database Connectivity)**
*   **核心依赖:**
    *   **JDBC Driver:** MySQL Connector/J (官方 MySQL JDBC 驱动程序)。
    *   **Connection Pooling:** **必须使用连接池**来管理数据库连接，以提高性能和资源利用率。强烈推荐：
        *   **HikariCP:** 高性能、轻量级、广泛使用的连接池库（常为 Spring Boot 默认）。
        *   **Apache Commons DBCP2:** 另一个成熟、稳定的选择。
        *   **(备选) C3P0:** 也是一个可选的连接池。
*   **开发实践:**
    *   **直接 JDBC:** 使用标准的 `java.sql` 包 (`Connection`, `PreparedStatement`, `ResultSet` 等)。
    *   **资源管理:** 必须严格使用 `try-with-resources` 语句或在 `finally` 块中手动关闭 `Connection`, `Statement`, 和 `ResultSet`，以防止资源泄露。连接池负责管理连接的生命周期，但代码中获取和使用的连接、语句、结果集仍需妥善关闭。
    *   **防止 SQL 注入:** **必须**使用 `PreparedStatement` 并通过占位符 (`?`) 绑定参数，**严禁**直接拼接字符串构造 SQL 语句。
    *   **手动对象映射:** 需要编写代码将 `ResultSet` 中的数据手动映射到 Java 对象（POJOs）。
*   **可选的 JDBC 辅助库 (简化开发):**
    *   **Spring JDBC (`JdbcTemplate`):** 如果项目使用 Spring 框架，`JdbcTemplate` 极大地简化了 JDBC 操作，自动处理资源管理、异常转换，并提供方便的结果映射回调。
    *   **Apache Commons DbUtils:** 一个轻量级的库，旨在减少 JDBC 的样板代码。
    *   **JDBI:** 提供更流畅 API 风格的 JDBC 抽象库。
*   **理由 (选择 JDBC vs ORM):**
    *   **优点:**
        *   **完全控制 SQL:** 对执行的 SQL 语句有最直接的控制，可能实现高度优化的查询。
        *   **无 ORM 抽象层开销:** 理论上可能在特定场景下有微小的性能优势（但连接池性能通常影响更大）。
        *   **概念简单 (对于基础操作):** 对于非常简单的 CRUD，直接 JDBC 可能感觉更直接。
    *   **缺点:**
        *   **代码冗余/样板代码:** 需要编写大量重复代码来处理连接、语句、结果集和异常。
        *   **手动资源管理易出错:** 忘记关闭资源可能导致泄露。
        *   **SQL 注入风险高:** 若不严格使用 `PreparedStatement`，易引入安全漏洞。
        *   **手动对象映射繁琐:** 将 `ResultSet` 映射到对象既枯燥又容易出错。
        *   **可移植性较差:** 更容易与特定数据库的 SQL 方言耦合。
        *   **重构困难:** 数据库模式变更时，需要在代码库中查找并修改分散的 SQL 字符串。

## 3. 模式管理 / 迁移

*   **技术:** 数据库迁移工具（不受交互层选择影响）
*   **具体库推荐 (Java 环境下):**
    *   **Flyway:** 简单易用，基于 SQL 脚本的版本控制。
    *   **Liquibase:** 功能更强大，支持 XML, YAML, JSON 或 SQL 格式定义变更，提供更多重构和数据库无关性选项。
*   **理由:** （同前）版本控制、可重复部署、协作。

## 4. 数据处理与清洗集成

*   **方法:** （同前）建议在**应用程序层**（例如使用 Java 的相关数据处理库，或如果分析是独立服务则使用 Python/Pandas/NumPy）进行。
*   **数据库角色:** （同前）存储原始数据和处理后的数据，高效检索数据供处理。

## 5. 处理传感器数据

*   **模式设计:** （同前）使用 `TIMESTAMP`, `FLOAT`/`DECIMAL` 等类型，考虑 `SensorReadings`, `Session`, `Patient`, `Device` 表结构。
*   **索引:** （同前）对 `session_id`, `timestamp` 等关键列建立索引。
*   **潜在优化:** （同前）考虑 MySQL 分区。

## 6. 备份与恢复

*   **技术:** （同前）`mysqldump`, 二进制日志，或云提供商方案。
*   **策略:** （同前）定期备份，定义 RPO/RTO。

## 7. 安全性

*   **方法:** （同前）强密码、专用用户、最小权限、安全存储凭证、SSL/TLS 连接。
*   **特别强调 (JDBC):** **强制使用 `PreparedStatement` 防止 SQL 注入。**

## 总结图示 (概念层 - JDBC 版)

## 总结图示 (概念层 - JDBC 版)

```mermaid
graph LR
    A["应用程序层<br/>(Java)<br/>- 业务逻辑<br/>- 数据处理<br/>- API 接口"] --> B("数据库交互层<br/>(JDBC, HikariCP/DBCP2)<br/>- MySQL Connector/J<br/>- (可选) JdbcTemplate/DbUtils/JDBI");
    B --> C["MySQL 数据库<br/>(引擎, 存储)<br/>- 表 (患者, 会话, 传感器读数)<br/>- 索引<br/>- 备份 (mysqldump)"];
    D["模式管理<br/>(Flyway / Liquibase)"] --> C;

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#ccf,stroke:#333,stroke-width:2px
    style C fill:#cdf,stroke:#333,stroke-width:2px
    style D fill:#fcc,stroke:#333,stroke-width:2px