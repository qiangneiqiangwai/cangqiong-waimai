# 项目上下文：苍穹外卖 (Sky Take Out)

## 概览
这是一个用于外卖应用（“苍穹外卖”）的 Java Spring Boot 后端项目。它遵循标准的 Maven 多模块架构。

### 技术栈
*   **语言:** Java (JDK 8+)
*   **框架:** Spring Boot 2.7.3
*   **构建工具:** Maven
*   **数据库:** MySQL
*   **ORM:** MyBatis (配合 PageHelper 进行分页)
*   **连接池:** Druid
*   **文档:** Knife4j (Swagger)
*   **其他:** Lombok, FastJSON, JJWT (JSON Web Token), AspectJ

## 架构
该项目分为三个主要的 Maven 模块：

1.  **`sky-common`**: 包含通用工具、常量、异常和属性配置。
    *   *角色*: “工具箱”。
    *   *关键内容*: `Result` 包装类, `BaseContext`, `AliOssUtil`, `JwtUtil`。
2.  **`sky-pojo`**: 包含所有的数据对象。
    *   *角色*: “数据载体”。
    *   *关键内容*:
        *   `entity`: 数据库表映射 (例如 `Employee`, `Dish`)。
        *   `dto`: 前端到后端请求的数据传输对象 (Data Transfer Objects)。
        *   `vo`: 后端到前端响应的视图对象 (View Objects)。
3.  **`sky-server`**: 包含业务逻辑的主要应用模块。
    *   *角色*: “大脑”。
    *   *结构*:
        *   `controller`: 请求处理。
        *   `service`: 业务逻辑。
        *   `mapper`: 数据库交互 (MyBatis)。
        *   `config`: Spring 配置 (WebMvc, Redis 等)。
        *   `interceptor`: 请求预处理 (例如 JWT 验证)。

## 安装与运行

### 前置要求
*   Java JDK 8 或 11
*   Maven 3.x
*   MySQL 5.7+ 运行在 `localhost:3306`

### 配置
数据库配置位于 `sky-server/src/main/resources/application-dev.yml`:
*   **数据库名:** `sky_take_out`
*   **用户名:** `root`
*   **密码:** `root`
*   **端口:** `8080` (定义在 `application.yml` 中)

### 运行应用
1.  确保 MySQL 数据库 `sky_take_out` 存在且可访问。
2.  导航到根目录。
3.  通过 Maven 运行（如果已安装 Maven）：
    ```bash
    mvn clean install
    cd sky-server
    mvn spring-boot:run
    ```
4.  **入口点:** `sky-server` 中的 `com.sky.SkyApplication`。

## 开发规范

*   **分层:** Controller -> Service -> Mapper。
*   **数据流:**
    *   请求: `DTO` -> `Controller` -> `Service` -> `Entity` -> `Mapper`。
    *   响应: `Mapper` -> `Entity` -> `Service` -> `VO` -> `Result<VO>` -> `Controller`。
*   **命名:**
    *   类: PascalCase (大驼峰，例如 `EmployeeController`)。
    *   方法/变量: camelCase (小驼峰，例如 `saveEmployee`)。
    *   数据库: snake_case (蛇形命名，通过 `map-underscore-to-camel-case: true` 映射到 camelCase)。
*   **异常:** 抛出自定义异常 (例如 `AccountNotFoundException`)，这些异常在 `GlobalExceptionHandler` 中被全局处理。

## 关键文件
*   `pom.xml`: 根 Maven 配置文件。
*   `sky-server/src/main/resources/application.yml`: 主配置文件。
*   `sky-server/src/main/java/com/sky/SkyApplication.java`: 应用入口点。
*   `sky-common/src/main/java/com/sky/result/Result.java`: 统一 API 响应包装类。
*   `day2.md`: 项目文档/教程笔记。