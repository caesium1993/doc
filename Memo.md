# MEMO

## DEV

## PROJECT

- 门户验收签报问题汇总
  - 验收签报关联《关于将中央监管信息平台项目工程验收通过视同公司项目验收通过的请示》（找一个模板）
  - 《工程验收申请单》的用印申请
    1. 验收签报中增加请示事项，请公司领导签字盖章
  - 《用户反馈报告》的签字
    1. 向信息中心确认
  - 验收评审会的时间如何确定
    1. 至少提前3天发函
  - 验收流程是怎样的？
    1. 运维总体组召开需求确认会，并出具会议纪要作为验收签报附件
    2. 向信息中心及公司领导（罗总、程总）汇报门户建设情况和申请验收情况
    3. 由张处协调验收评审会的会议时间和参会人员
    4. 根据协定的会议时间、需求确认会的会议纪要等发起申请验收签报
    5. 召开评审会
    6. 按公司项目管理规定，通过验收后，上一个项目总结报告

## OTHER

### Code Review of App Module in IAM and Support

- iam问题：
  - cerateApp():方法名拼写
  - cerateApp() 实现没有考虑边界条件，比如：app是否已存在，id及name等字段是否合法等，不能为空的字段没校验....
  - cerateApp() 实现应依托support完成
  - editApp() 明确接口入参，而不是通过笼统的SysAppDto赋值，Dto里有些属性和本接口无关，如createdBy等
  - editApp() 没有处理边界条件
  - deleteSysApp() 没有处理边界条件，删除应用时关联的表数据也要处理
  - 要支持批量删除
  - 没有异常处理，比如入参不合法或出现唯一性冲突等，应向上层抛出IllegalArgumentsException并给出明确的异常message，抛出的异常应在SysAppService接口中定义清楚，并在接口documentation中注明
  - findAll() 明确SysAppDto的取值是否精确，对必要的属性进行@NotNull、@NotEmpty等标注
  - findAll() 异常处理
  - findAll() 返回值应明确简洁的返回分页查询结果，而非IamResponseEntity
  - 不要在iam中重新创建IamResponseEntity，user、org、app、permission、dataperm等领域模型的service交互尽量不要使用泛型封装的dto，而应简洁明确的提供返回结果，在documentation中标注可能出现的exception及返回值情况。特殊情况必须如使用泛型封装的dto，使用SupportResponseEntity

- suuport问题：
  - 在support的app module中创建 SysAppInfoService和SysAppManagementService
  - 上述iam接口主体逻辑应在support中实现，iam仅做封装
  - 除iam封装的接口外，support的app module还应提供包括应用鉴权、sysAppUrl管理、应用分页链接查询、根据appid及uriid查找对应链接等相关的服务
  - 同理，异常处理

### Code Review of Permission Module in IAM and Support

- iam:
  - delete、save等接口应保证操作的事务性，即：一条记录操作失败，整个操作回滚。
  - 对于发生异常的操作回滚或是非法入参应抛出Exception，并给出明确的Exception Message，目前PermissionService的几个接口实现都缺乏这个处理

  - getPermissionByUserId

- support: