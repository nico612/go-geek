要实现服务授权，首先要根据业务选择一个授权模式，不同的权限模型具有不同的特点，可以满足不同的需求。常见的权限模型有下面这 5 种：

- 权限控制列表（ACL，Access Control List）；
- 自主访问控制（DAC，Discretionary Access Control）；
- 强制访问控制（MAC，Mandatory Access Control）；
- 基于角色的访问控制（RBAC，Role-Based Access Control）；
- 基于属性的权限验证（ABAC，Attribute-Based Access Control）。

当前用的比较多的是 RBAC 模式。关于 RBAC 模式的介绍，网上已经有很多文章，这里不再赘述，这里有一篇文章供你参考：[详细了解RBAC（Role-Based Access Control）](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F513142061)。RBAC 能够满足绝大部分的企业应用授权场景，例如 Kubernetes 就使用了 RBAC 授权模式。miniblog 也使用了 RBAC 授权模式。

go中常用的三方库为：casbin

## Casbin

[Casbin 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fcasbin.org%2Fzh%2Fdocs%2Foverview)。文档内容很多，主要看 [访问控制模型](https://casbin.org/zh/docs/category/model)这一块来充分理解模型的定义和用法，然后观看视频[[Gin 教程 第10章：30分钟内学习 Casbin 基础模型](https://www.bilibili.com/video/BV1qz4y167XP)]更好的理解，最后结合 gin-vue-admin 开源库中的实现来实战。

官方文档中比较实用的内容：

- casbin 提供一个在线编辑器，可以用来进行权限验证：[Editor | Casbin](https://link.juejin.cn/?target=https%3A%2F%2Fcasbin.org%2Fzh%2Feditor)；
- 有很多基于 casbin 实现授权的文章和课程：[使用指南](https://link.juejin.cn/?target=https%3A%2F%2Fcasbin.org%2Fzh%2Fdocs%2Ftutorials)；
- casbin 授权依赖 Model 文件来描述授权模式，依赖 Policy 文件来指定权限。并且官网提供了不同授权模式的 Model 文件和 Policy 文件示例：[支持的模型](https://link.juejin.cn/?target=https%3A%2F%2Fcasbin.org%2Fzh%2Fdocs%2Fsupported-models)；
- casbin 的 Model 文件和 Policy 文件可以从不同的位置加载；
- casbin 针对不同的 Web 框架已经有相应的中间件实现：[中间件](https://link.juejin.cn/?target=https%3A%2F%2Fcasbin.org%2Fzh%2Fdocs%2Fmiddlewares)；
- 当然，还有很多其他有用的文档。





用户表： sys_user

| id（用户ID） | username（用户名） | authority_id （角色id） |
| :----------: | :----------------: | :---------------------: |
|      1       |        张三        |           888           |

角色表： sys_authorities

| authority_id（角色id） | authority_name（角色名） | parent_id（父角色id) | default_router（默认路由) |
| :--------------------: | :----------------------: | :------------------: | :-----------------------: |
|          888           |         普通用户         |          0           |         dashboard         |

用户_角色表： sys_user_authorities

| user_id | authority_authority_id |
| ------- | ---------------------- |
| 1       | 888                    |

casbin策略规则表，使用cashbin提供的go语言版本的adpter自动创建的表：casbin_rule

| id   | ptype | v0（角色id） | v1（资源）               | v2（操作） | v3   | v4   | V5   |
| ---- | ----- | ------------ | ------------------------ | ---------- | ---- | ---- | ---- |
| 2    | p     | 888          | /user/setUserAuthorities | POST       |      |      |      |
|      |       |              |                          |            |      |      |      |

api 表设计：sys_api 表

| id   | path                     | description | api_group | method |
| ---- | ------------------------ | ----------- | --------- | ------ |
| 1    | /user/setUserAuthorities | 设置权限组  | 系统用户  | post   |
|      |                          |             |           |        |

