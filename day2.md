✦ 这是一个典型的 Maven 多模块 + Spring Boot + MyBatis 架构。

在这个架构中，sky-server 是核心业务模块（相当于“大脑”），而 sky-pojo 和 sky-common 是它的支撑模块（相当于“血液”和“工具箱”）。

下面我为你详细拆解 sky-server 的内部结构，以及这三个模块是如何联动的。

  ---

1. sky-server 内部结构与分层 (MVC 模式)

sky-server 负责处理所有的业务请求。它的文件夹（包）结构遵循经典的 MVC 三层架构：


┌───────────────┬────────────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────┬───────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 包名 (Pack... │ 角色 (R... │ 作用 (Function)                                                                                               │ 比喻                                                                                              │
├───────────────┼────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ `controller`    │ 控制层     │ 入口。接收前端发送的 HTTP 请求，解析参数，校验参数，然后指挥 Service 干活。最后将结果封装成统一格式返回给...  │ 服务员：拿着菜单（接口文档）接待顾客，把顾客的点单（参数）传给厨房。
│
│ `service`       │ 业务层     │ 逻辑核心。处理具体的业务逻辑（如：计算价格、判断库存、密码加密）。通常包含 Service 接口和 impl 实现类。       │ 厨师：根据菜单做菜，负责切菜、炒菜、调味（业务逻辑）。
│
│ `mapper`        │ 持久层     │ 数据交互。直接与数据库打交道。定义 SQL 语句（或通过 XML 定义），负责数据的增删改查。                          │ 库管/采购：负责去仓库（数据库）拿食材（数据），或者把做好的半成品放回仓库。
│
│ `config`        │ 配置层     │ 存放 Spring 的配置类。例如配置拦截器（Interceptor）、日期格式转换、Redis 配置等。                             │ 餐厅经理：制定餐厅规则，安排谁负责安保（拦截器），谁负责打扫。
│
│ `interceptor`   │ 拦截器     │ 在请求到达 Controller 之前进行预处理。比如：校验用户是否登录（JWT 校验）。                                    │ 保安：检查顾客有没有入场券（Token），没有就不让进。
│
│ `handler`       │ 处理器     │ 全局异常处理（GlobalExceptionHandler）。当代码报错时，统一捕获并返回友好的提示。                              │ 客服/公关：当出事故（报错）时，出面安抚顾客（返回友好提示），而不是让顾客看到厨房着火（报错堆...
│
└───────────────┴────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────┘

  ---

2. 三大模块如何联动？

这三个模块在 pom.xml 中定义了依赖关系，通常是：
sky-server 依赖 sky-pojo 和 sky-common。
A. 联动 sky-pojo (数据载体)
sky-pojo 里面存放的全是简单的 Java 对象，它们是各层之间传递数据的载体。

* Entity (实体类)：对应数据库表。
    * 场景：Mapper 层从数据库查出来的数据，会映射成 Entity 对象。
* DTO (Data Transfer Object)：数据传输对象。
    * 场景：前端 -> 后端。前端传来的参数可能比数据库表少（比如登录只传用户名密码），或者多（比如传了验证码），这时 Controller 接收的就是 DTO。
* VO (View Object)：视图对象。
    * 场景：后端 -> 前端。后端返回给前端的数据，可能需要隐藏某些字段（如密码），或者格式化日期，这时 Service 或 Controller 会把 Entity 转换成 VO 返回。

联动流程示例（新增员工）：
1. 前端发 JSON 数据 -> Controller (接收为 EmployeeDTO)。
2. Controller 传 EmployeeDTO 给 Service。
3. Service 使用 BeanUtils 将 EmployeeDTO 里的数据拷贝到一个新的 Employee (Entity) 对象中，并补充某些默认值（如创建时间）。
4. Service 传 Employee (Entity) 给 Mapper。
5. Mapper 将 Employee (Entity) 的属性拼接到 SQL 中，存入数据库。

B. 联动 sky-common (公共工具)
sky-common 存放通用的、不依赖具体业务的代码，可以被任何模块复用。

* Constant (常量)：如 MessageConstant（提示信息）、StatusConstant（状态码）。
    * 联动：Service 层判断逻辑时使用（如 if(status == StatusConstant.ENABLE)）。
* Context (上下文)：如 BaseContext。
    * 联动：Interceptor 存入用户 ID，Service 层取出用户 ID。
* Result (统一返回结果)：Result<T> 类。
    * 联动：Controller 层最终返回给前端的数据，都会被包裹在 Result.success(data) 或 Result.error(msg) 中，保证前后端交互格式统一。
* Utils (工具类)：如 AliOssUtil (文件上传), JwtUtil (令牌生成)。
    * 联动：Service 层需要上传图片时，直接调用 AliOssUtil。

  ---

3. 一个完整的请求流转图 (以“新增员工”为例)

    1 用户发起请求 (POST /admin/employee)
    2       ↓
    3 1. [sky-server] Interceptor (拦截器)
    4    - 检查 JWT 令牌
    5    - 将 userId 存入 BaseContext (ThreadLocal) -- (用到 sky-common)
    6       ↓
    7 2. [sky-server] EmployeeController
    8    - 接收参数 EmployeeDTO -- (用到 sky-pojo)
    9    - 调用 service.save(employeeDTO)
10       ↓
11 3. [sky-server] EmployeeServiceImpl
12    - 创建 Employee (Entity) 对象 -- (用到 sky-pojo)
13    - 对象属性拷贝 (DTO -> Entity)
14    - 补充创建人ID: BaseContext.getCurrentId() -- (用到 sky-common)
15    - 密码加密 -- (用到 sky-common 的工具或 JDK)
16    - 调用 mapper.insert(employee)
17       ↓
18 4. [sky-server] EmployeeMapper
19    - 执行 SQL: INSERT INTO employee ...
20       ↓
21 5. [Database] (sky.sql)
22    - 数据落库
23       ↓
24 6. 返回流程
25    - Mapper -> Service -> Controller
26    - Controller 返回 Result.success() -- (用到 sky-common)

总结
* sky-server：干活的主力，包含业务逻辑。
* sky-pojo：负责在各层之间“搬运数据”。
* sky-common：负责提供“扳手、螺丝刀”等通用工具和标准件。

这种分层让代码结构清晰，维护方便。如果将来要加一个“用户端小程序”的模块（比如 sky-client），它也可以直接引用 sky-pojo 和 sky-common，复用大量代码。

2.员工分页查询
总结全流程

1. DTO 像一个容器，装着前端传来的 page: 1, pageSize: 10, name: "张"。
2. Controller 接到容器，转手交给 Service。
3. Service 先喊一声 PageHelper.startPage（开启外挂），然后让 Mapper 去查。
4. Mapper 准备执行 SQL：SELECT * FROM employee WHERE name LIKE '%张%'。
5. PageHelper（外挂）拦截了这条 SQL，偷偷改成了：
    * 先查总数：SELECT COUNT(*) FROM employee WHERE name LIKE '%张%'
    * 再查数据：SELECT * FROM employee WHERE name LIKE '%张%' ORDER BY ... LIMIT 0, 10
6. Service 拿到 PageHelper 返回的“总数”和“10条数据”，把它们装进 PageResult。
7. Controller 把 PageResult 包装好，发回给前端