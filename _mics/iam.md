# IAM 3rd PHASE

## 分级授权需求梳理

## IAM

----------------------------

### REQUIREMENT

- 提供开箱可用的客户端工具包，支持分级授权，权限查询等，便于快速集成
- 管理端提供类似URMS的权限管理服务，方便业务系统通过iam进行访问控制

### IAM功能设计

- 管理端（以独立应用的形式发布）：

  - 用户管理：
    - 主体功能参照门户的“用户管理”模块，以列表页显示各类型用户，支持模糊查询，修改，创建，删除等,功能和门户一样不赘述
    - 支持用户信息的修改，包括姓名、证件号、证件类型、手机、邮箱、座机、照片等
    - 支持给用户绑定组织，组织的查询列表应展示组织的树状结构
      - *绑定组织时可编辑用户与组织的关联信息，如职位，是否为主组织，入职时间*
    - *给用户分组（待定，第一版原型暂不考虑）*
      - *新建分组：分组的名称、id等*
      - ***把监管主体、市场主体、社会公众等用户类别看做是系统的默认分组？***
    - 给用户授权，并指定数据权限(当前支持数据权限类型为：辖区、业务、可分配的角色、组织)
      - 选中角色后，显示该角色可选的数据权限，用户选择数据权限并保存，下面为几种典型场景用于原型mock数据：
      - admin给李成龙授予北京辖区上市业务的监管主体用户角色：在角色列表中选中“监管主体用户角色” -> 动态显示“辖区”和“业务”两个类数据权限的复选框 -> 选择“辖区”为北京，“业务”为上市
      - admin给宋懂懂授予北京辖区的派出机构用户管理员，宋懂懂可以管理北京局的监管主体用户，并给他们授权不同业务线的监管主体用户角色：角色列表选中“派出机构监管主体用户管理员” -> 动态显示“辖区”和“角色”两个数据权限 -> 选择辖区为“北京”，角色为“监管主体用户” -> 动态显示“监管主体用户”角色的数据权限“辖区”和“业务” -> 辖区仅北京可选，业务选择全选 -> 宋懂懂 “派出机构监管主体用户管理员” 的数据权限动态变成3个：辖区、角色和业务

  - *应用管理：*
    - *以列表形式显示应用，支持应用的新增、编辑、名称及id的查询、删除、资源管理*
    - *应用可以维护的字段：应用名称、应用标识、应用密钥*
    - *资源管理：显示应用已关联的资源，支持修改，资源列表显示的字段：资源类型、资源名称、资源标识、资源取值*

  - 资源管理：
    - 创建，修改，删除资源，资源类型可以是url、组件、栏目、数据、应用
    - 资源可维护的信息:资源类型、资源名称、资源标识、资源描述、资源取值、所关联的应用
    - 可以为资源限制访问权限:授权的对象可以是角色或角色的数据权限，可定义资源允许的操作：读、写、删除、全部。几种典型场景用于原型mock数据：
      - 舆情赛马频道（应用）设置为仅监管主体用户可访问
      - 刘成龙发布的seq为12的调查问卷设置仅北京局上市处的监管主体用户可访问：资源类型选中数据，列表中选中该调查问卷，授权对象类型为“角色”，选中监管主体用户角色后动态显示其对应的“辖区”和“业务”两类数据权限，选中“北京”和“上市”后确认。
    - *授权的操作可以为读写删甚至是抽象许可权（后续优化的备忘，原型暂不考虑）*

  - 角色管理：
    - 创建管理角色，同门户的角色管理，不赘述
    - 管理角色的数据权限类型
    - 新增/修改数据权限可维护的字段有：数据权限的id，名称，实现类的名称
    - 当前有在辖区、业务、可分配的角色、部门4种数据权限
    - *给角色分组（待定，第一版原型暂不考虑）*

  - *组织管理：*
    - 合适的组件显示当前的完整的组织树
    - 选中某个父节点后，可创建组织（功能和门户一致，不赘述），删除组织（可选择删除当前组织及其下所有子组织，或仅删除当前组织并将其子组织挂于其他节点），修改组织（修改组织信息的功能与门户一致，同时也支持修改组织关系，即重新选择当前的父节点，其子节点随其一起变动）
    - 可以显示叶子结点组织下的用户列表，支持查询

  - （*）授权规则配置：
    - 角色互斥组管理
    - 基数约束管理
    - 先决条件约束
    - ... 

  - *通过角色控制当前用户可操作的iam的栏目，并给出常见操作的提示，类似 aws console*

- 客户端（以jar的形式集成，service在后续版本中通过RestTemplate实现）
  - 角色管理的相关接口
  - 用户管理的相关接口
  - 组织管理的相关接口
  - 资源管理的相关接口