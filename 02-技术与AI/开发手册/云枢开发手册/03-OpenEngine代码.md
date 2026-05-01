# OpenEngine代码

```java

/**
 * 公开引擎
 */
public interface OpenEngine {

    /**
     * 新增
     *
     * @param bizObject 业务对象
     * @return 业务对象id
     */
    @Nonnull
    String createBizObject(@Nonnull BizObject bizObject);

                /**
     * 批量新增
     *
     * @param bizObjects    业务对象列表
     * @param invokeBizRule 是否执行业务规则
     * @return 业务数据id列表(仅当执行业务规则时返回)
     */
    @Nonnull
	    List<String> batchCreateBizObjects(@Nonnull List<BizObject> bizObjects, boolean invokeBizRule);

    /**
     * 删除
     *
     * @param schemaCode  模型编码
     * @param bizObjectId 业务数据id
     */
    void deleteBizObject(@Nonnull String schemaCode, @Nonnull String bizObjectId);

    /**
     * 更新
     *
     * @param bizObject           业务对象
     * @param invokeCalculateRule 是否执行计算规则
     */
    void updateBizObject(@Nonnull BizObject bizObject, boolean invokeCalculateRule);

    /**
     * 统计，仅支持本地数据库统计
     *
     * @param bizObjectQuery 查询对象
     * @return 数量
     */
    long countBizObject(@Nonnull BizObjectQuery bizObjectQuery);

    /**
     * 列表查询
     *
     * @param bizObjectQuery 查询对象
     * @return 业务对象列表
     */
    @Nonnull
    List<BizObject> listBizObjects(@Nonnull BizObjectQuery bizObjectQuery);

    /**
     * 查询
     *
     * @param schemaCode      模型编码
     * @param bizObjectId     业务对象id
     * @param queryChildCodes 查询的子表编码
     * @return 业务对象
     */
    @Nonnull
    BizObject getBizObject(@Nonnull String schemaCode,
                           @Nonnull String bizObjectId,
                           @Nonnull List<String> queryChildCodes);

    /**
     * 执行自定义业务规则
     *
     * @param schemaCode  模型编码
     * @param bizRuleCode 规则编码
     * @param request     请求参数(类型根据业务规则参数类型指定)
     * @return 业务规则返回值
     */
    @Nullable
    Object executeCustomBizRule(@Nonnull String schemaCode, @Nonnull String bizRuleCode, @Nullable Object request);

    /**
     * 查询模型元数据
     *
     * @param schemaCode 模型编码
     * @return 模型元数据
     */
    @Nullable
    BizSchema getSchemaByCode(@Nonnull String schemaCode);

    /**
     * 根据id查询流程实例
     *
     * @param workflowInstanceId 流程实例id
     * @return 流程实例
     */
    @Nullable
    WorkflowInstance getWorkflowInstance(@Nonnull String workflowInstanceId);

    /**
     * 根据业务数据id查询流程实例
     *
     * @param bizObjectId 业务数据id
     * @return 流程实例
     */
    @Nullable
    WorkflowInstance getWorkflowInstanceByBizObjectId(@Nonnull String bizObjectId);

    /**
     * 根据任务id查询任务
     *
     * @param workItemId 任务id
     * @return 流程任务
     */
    @Nullable
    WorkItem getWorkItem(@Nonnull String workItemId);

    /**
     * 根据流程实例id查询任务
     *
     * @param workflowInstanceId 流程实例id
     * @param isFinished         是否已完成
     * @return 流程任务列表
     */
    @Nonnull
    List<WorkItem> listWorkItems(@Nonnull String workflowInstanceId, boolean isFinished);

    /**
     * 发起流程
     *
     * @param departmentId 用户部门Id
     * @param userId       用户Id
     * @param workflowCode 流程编码
     * @param bizObject    业务对象
     * @param finishStart  是否结束发起节点
     * @return 流程实例Id
     */
    @Nonnull
    String startWorkflowInstance(@Nonnull String departmentId,
                                 @Nonnull String userId,
                                 @Nonnull String workflowCode,
                                 @Nonnull BizObject bizObject,
                                 boolean finishStart);

    /**
     * 提交待办任务
     *
     * @param workItemId 待办任务id
     * @param approval   是否同意  true/false
     * @return true/false
     */
    boolean submitWorkItem(@Nonnull String workItemId, boolean approval);

    /**
     * 驳回待办任务
     *
     * @param workItemId           待办任务ID
     * @param rejectToActivityCode 驳回到指定的节点
     * @param submitToReject       是否可以直接提交到驳回的节点
     * @return true/false
     */
    boolean rejectWorkItem(@Nonnull String workItemId, @Nonnull String rejectToActivityCode, boolean submitToReject);

    /**
     * 转办待办任务
     *
     * @param workItemId  待办任务ID
     * @param participant 任务接收人ID
     * @return true/false
     */
    boolean forwardWorkItem(@Nonnull String workItemId, @Nonnull String participant);

    /**
     * 协办待办任务
     *
     * @param workItemId   待办任务ID
     * @param participants 协办人ID
     * @return true/false
     */
    boolean assistWorkItem(@Nonnull String workItemId, @Nonnull List<String> participants);

    /**
     * 传阅待办任务
     *
     * @param workItemId   待办任务ID
     * @param participants 待阅人id
     * @return true/false
     */
    boolean circulateWorkItem(@Nonnull String workItemId, @Nonnull List<String> participants);

    /**
     * 加签待办任务
     *
     * @param workItemId   待办任务ID
     * @param participants 任务接收人id
     * @return true/false
     */
    boolean adjustWorkItem(@Nonnull String workItemId, @Nonnull List<String> participants);

    /**
     * 撤回已办任务
     *
     * @param workflowInstanceId 流程实例ID
     * @param activityCode       节点编码
     * @return true/false
     */
    boolean retrieveWorkItem(@Nonnull String workflowInstanceId, @Nonnull String activityCode);

    /**
     * 是否可以撤回
     *
     * @param workflowInstanceId 流程实例id
     * @param activityCode       节点编码
     * @return true/false
     */
    boolean canRetrieve(@Nonnull String workflowInstanceId, @Nonnull String activityCode);

    /**
     * 通过任务Id获取节点许可的动作
     *
     * @param workItemId 任务ID
     * @return 可执行的动作
     */
    @Nullable
    PermittedAction getPermittedActionByWorkItemId(@Nonnull String workItemId);

    /**
     * 激活指定活动
     * 同时会取消前置节点未完成的任务
     *
     * @param workflowInstanceId 流程实例id
     * @param activityCode       节点编码
     * @return true/false
     */
    boolean activateActivity(@Nonnull String workflowInstanceId, @Nonnull String activityCode);

    /**
     * 调整当前活动处理人
     *
     * @param workflowInstanceId 流程实例ID
     * @param activityCode       节点编码
     * @param participants       参与者
     * @return true/false
     */
    boolean adjustActivity(@Nonnull String workflowInstanceId,
                           @Nonnull String activityCode,
                           @Nonnull List<String> participants);

    /**
     * 取消当前节点活动
     *
     * @param workflowInstanceId 流程实例ID
     * @param activityCode       节点编码
     * @return true/false
     */
    boolean cancelActivity(@Nonnull String workflowInstanceId, @Nonnull String activityCode);

    /**
     * 结束流程
     *
     * @param workflowInstanceId 流程实例ID
     * @return true/false
     */
    boolean finishWorkflowInstance(@Nonnull String workflowInstanceId);

    /**
     * 删除流程
     * 流程相关所有数据都会被删除，包括BO数据
     *
     * @param workflowInstanceId 流程实例id
     * @return true/false
     */
    boolean deleteWorkflowInstance(@Nonnull String workflowInstanceId);

    /**
     * 取消流程
     *
     * @param workflowInstanceId 流程实例ID
     * @return true/false
     */
    boolean cancelWorkflowInstance(@Nonnull String workflowInstanceId);

    /**
     * 保存审批意见
     *
     * @param bizComment 审批意见对象
     * @return 审批意见id
     */
    @Nonnull
    String createBizComment(@Nonnull BizComment bizComment);

    /**
     * 根据流程实例id查询审批意见
     *
     * @param workflowInstanceId 流程实例id
     * @return 审批意见集合
     */
    @Nonnull
    List<BizComment> listBizComments(@Nonnull String workflowInstanceId);

    /**
     * 根据业务服务编码获取业务服务
     *
     * @param serviceCode 业务服务编码
     * @return 业务服务
     */
    @Nullable
    BizService getBizServiceByCode(@Nonnull String serviceCode);

    /**
     * 根据业务服务编码查询方法列表
     *
     * @param serviceCode 业务服务编码
     * @return 业务方法列表
     */
    @Nonnull
    List<BizServiceMethod> getBizServiceMethods(@Nonnull String serviceCode);

    /**
     * 根据业务服务编码、集成方法编码查询方法
     *
     * @param serviceCode 业务服务编码
     * @param methodCode  集成方法编码
     * @return 业务服务方法
     */
    @Nullable
    BizServiceMethod getBizServiceMethodByCode(@Nonnull String serviceCode, @Nonnull String methodCode);

    /**
     * 调用业务集成方法
     *
     * @param schemaCode   调用对象模型编码
     * @param bizObjectId  调用对象id
     * @param serviceCode  集成服务编码
     * @param methodCode   集成方法编码
     * @param args         参数
     * @param milliSeconds 调用控制参数配置，ms
     * @return 调用返回值(返回协议适配器层调用的返回结果)
     */
    @Nullable
    Object invokeBizServiceMethod(@Nullable String schemaCode,
                                  @Nullable String bizObjectId,
                                  @Nonnull String serviceCode,
                                  @Nonnull String methodCode,
                                  @Nonnull Map<String, Object> args,
                                  long milliSeconds);

    /**
     * 新增部门
     *
     * @param department 部门对象
     * @return 部门
     */
    @Nonnull
    Department createDepartment(@Nonnull Department department);

    /**
     * 更新部门
     *
     * @param department 部门对象
     * @return 部门
     */
    @Nonnull
    Department updateDepartment(@Nonnull Department department);

    /**
     * 逻辑删除部门
     *
     * @param department 部门对象
     */
    void removeDepartment(@Nonnull Department department);

    /**
     * 根据部门id查询部门信息
     *
     * @param departmentId 部门id
     * @return 部门对象
     */
    @Nullable
    Department getDepartment(@Nonnull String departmentId);

    /**
     * 根据部门id获取所有上级部门至根部门
     *
     * @param departmentId   部门id
     * @param excludedRemove 是否排除逻辑删除
     * @return 部门对象集合
     */
    @Nonnull
    List<Department> getParentDepartments(@Nonnull String departmentId, boolean excludedRemove);

    /**
     * 根据id列表查询部门列表
     *
     * @param departmentIds 部门id集合
     * @return 部门对象集合
     */
    @Nonnull
    List<Department> getDepartmentsByDepartmentIds(@Nonnull List<String> departmentIds);

    /**
     * 根据用户id获取所属的部门列表
     *
     * @param userId 用户id
     * @return 部门对象集合
     */
    @Nonnull
    List<Department> getDepartmentsByUserId(@Nonnull String userId);

    /**
     * 根据部门id查询子部门列表
     *
     * @param departmentId   部门id
     * @param recursive      是否递归查询
     * @param excludedRemove 是否排除逻辑删除
     * @return 部门列表
     */
    @Nonnull
    List<Department> getChildDepartmentsByDepartmentId(@Nonnull String departmentId,
                                                       boolean recursive,
                                                       boolean excludedRemove);

    /**
     * 获取部门的全名路径
     *
     * @param departmentId 部门
     * @return 部门的全名路径
     */
    @Nullable
    String getDepartmentPathByDepartmentId(@Nonnull String departmentId);

    /**
     * 新增部门用户
     *
     * @param departmentUser 部门用户
     * @return 部门用户
     */
    @Nonnull
    DepartmentUser createDepartmentUser(@Nonnull DepartmentUser departmentUser);

    /**
     * 更新部门用户
     *
     * @param departmentUser 部门用户
     * @return 部门用户
     */
    @Nonnull
    DepartmentUser updateDepartmentUser(@Nonnull DepartmentUser departmentUser);

    /**
     * 删除部门用户
     *
     * @param departmentUser 部门用户对象
     */
    void deleteDepartmentUser(@Nonnull DepartmentUser departmentUser);

    /**
     * 根据部门列表查询部门用户关系列表
     *
     * @param departmentIds 部门id集合
     * @return 部门用户对象集合
     */
    @Nonnull
    List<DepartmentUser> getDepartmentUsersByDepartmentIds(@Nonnull List<String> departmentIds);

    /**
     * 根据用户列表查询部门用户关系列表
     *
     * @param userIds 用户id集合
     * @return 部门用户对象集合
     */
    @Nonnull
    List<DepartmentUser> getDepartmentUsersByUserIds(@Nonnull List<String> userIds);

    /**
     * 新增角色组
     *
     * @param roleGroup 角色组
     * @return 角色组
     */
    @Nonnull
    RoleGroup createRoleGroup(@Nonnull RoleGroup roleGroup);

    /**
     * 修改角色组
     *
     * @param roleGroup 角色组
     * @return 角色组
     */
    @Nonnull
    RoleGroup updateRoleGroup(@Nonnull RoleGroup roleGroup);

    /**
     * 删除角色组
     *
     * @param roleGroup 角色组对象
     */
    void deleteRoleGroup(@Nonnull RoleGroup roleGroup);

    /**
     * 查询角色组
     *
     * @param ids 角色组id
     * @return 角色组集合
     */
    @Nonnull
    List<RoleGroup> getRoleGroupsByIds(@Nonnull List<String> ids);

    /**
     * 根据角色编码查询角色组
     *
     * @param roleCode 角色编码
     * @return 角色组
     */
    @Nullable
    RoleGroup getRoleGroupByRoleCode(@Nonnull String roleCode);

    /**
     * 根据父级id 来查询相关的字角色组
     *
     * @param parentId 父级id
     * @return 角色集合
     */
    @Nonnull
    List<RoleGroup> getRoleGroupsByParentId(@Nonnull String parentId);

    /**
     * 根据钉钉角色组名查询角色组
     *
     * @param roleGroupName 角色组名称
     * @return 角色组列表
     */
    @Nonnull
    List<RoleGroup> getRoleGroupsByName(@Nonnull String roleGroupName);

    /**
     * 根据钉钉的角色组id查询角色组
     *
     * @param sourceId 钉钉的角色组id
     * @return 角色组
     */
    @Nullable
    RoleGroup getRoleGroupBySourceId(@Nonnull String sourceId);

    /**
     * 查询组织下的角色组列表
     *
     * @param corpId corpId
     * @return 角色组列表
     */
    @Nonnull
    List<RoleGroup> getRoleGroupsByCorpId(@Nonnull String corpId);

    /**
     * 新增角色
     *
     * @param role 角色
     * @return 角色
     */
    @Nonnull
    Role createRole(@Nonnull Role role);

    /**
     * 修改角色
     *
     * @param role 角色
     * @return 角色
     */
    @Nonnull
    Role updateRole(@Nonnull Role role);

    /**
     * 删除角色
     *
     * @param role 角色
     */
    void deleteRole(@Nonnull Role role);

    /**
     * 根据角色ID查询角色
     *
     * @param id 角色id
     * @return 角色
     */
    @Nullable
    Role getRole(@Nonnull String id);

    /**
     * 根据角色编码查询角色
     *
     * @param code 角色编码
     * @return 角色
     */
    @Nullable
    Role getRoleByCode(@Nonnull String code);

    /**
     * 根据角色组id查询角色
     *
     * @param roleGroupId 角色组id
     * @return 角色集合
     */
    @Nonnull
    List<Role> getRolesByRoleGroupId(@Nonnull String roleGroupId);

    /**
     * 根据sourceIds正序查询指定的企业角色集合
     *
     * @param cropId cropId
     * @param sourceIds sourceIds
     * @return 角色集合
     */
    @Nonnull
    List<Role> getRolesBySourceIds(@Nonnull String cropId, @Nonnull List<String> sourceIds);

    /**
     * 根据用户userIds查询每个用户对应角色列表
     *
     * @param userIds 用户id列表
     * @return 每个用户对应角色列表
     */
    @Nonnull
    Map<String, List<Role>> getUserRolesByUserIds(@Nonnull List<String> userIds);

    /**
     * 新增角色用户
     *
     * @param roleUser 角色用户
     * @return 角色用户
     */
    @Nonnull
    RoleUser createRoleUser(@Nonnull RoleUser roleUser);

    /**
     * 更新角色用户
     *
     * @param roleUser 角色用户
     * @return 角色用户
     */
    @Nonnull
    RoleUser updateRoleUser(@Nonnull RoleUser roleUser);

    /**
     * 删除角色用户
     *
     * @param roleUser 角色用户
     */
    void deleteRoleUser(@Nonnull RoleUser roleUser);

    /**
     * 通过roleId 删除角色用户关系
     *
     * @param roleId 角色id
     */
    void deleteRoleUserByRoleId(@Nonnull String roleId);

    /**
     * 根据用户id 查询角色用户关系
     *
     * @param userId 用户id
     * @return 部门用户
     */
    @Nonnull
    List<RoleUser> getRoleUsersByUserId(@Nonnull String userId);

    /**
     * 根据roleId查询角色用户集合
     *
     * @param roleId 角色Id
     * @return 角色用户集合
     */
    @Nonnull
    List<RoleUser> getRoleUsersByRoleId(@Nonnull String roleId);

    /**
     * 根据角色id和用户userId查询角色用户
     *
     * @param roleId 角色id
     * @param userId 用户id
     * @return 角色用户
     */
    @Nullable
    RoleUser getRoleUserByRoleIdAndUserId(@Nonnull String roleId, @Nonnull String userId);

    /**
     * 新增用户
     *
     * @param user 用户
     * @return 用户
     */
    @Nonnull
    User createUser(@Nonnull User user);

    /**
     * 更新用户
     *
     * @param user 用户对象
     * @return 用户
     */
    @Nonnull
    User updateUser(@Nonnull User user);

    /**
     * 逻辑删除用户
     *
     * @param user 用户对象
     */
    void removeUser(@Nonnull User user);

    /**
     * 根据id 查询用户
     *
     * @param id 用户表id
     * @return 用户对象
     */
    @Nullable
    User getUserById(@Nonnull String id);

    /**
     * 根据手机号查询用户对象
     *
     * @param mobile         手机号码
     * @param excludedRemove 是否排除逻辑删除
     * @return 用户对象
     */
    @Nullable
    User getUserByMobile(@Nonnull String mobile, boolean excludedRemove);

    /**
     * 根据用户名查询用户对象
     *
     * @param username       用户名
     * @param excludedRemove 是否排除逻辑删除
     * @return 用户对象
     */
    @Nullable
    User getUserByUsername(@Nonnull String username, boolean excludedRemove);

    /**
     * 通过用户id批量查询用户
     *
     * @param ids 用户id集合
     * @return 用户id, 用户集合
     */
    @Nonnull
    Map<String, User> getUsers(@Nonnull List<String> ids);

    /**
     * 根据roleId查询对应角色的用户集合
     *
     * @param roleId 角色Id
     * @return 用户集合
     */
    @Nonnull
    List<User> getUsersByRoleId(@Nonnull String roleId);

    /**
     * 根据部门id 查询用户列表
     *
     * @param departmentId 部门id
     * @return 用户列表
     */
    @Nonnull
    List<User> getUsersByDepartmentId(@Nonnull String departmentId);

    /**
     * 根据账号名称或者用户名称模糊查询用户列表
     *
     * @param keyword 关键字
     * @param offset  偏移量
     * @param limit   查询条数
     * @return 用户列表
     */
    @Nonnull
    List<User> searchUsersByName(@Nonnull String keyword, int offset, int limit);

    /**
     * 根据用户id查询管理员
     *
     * @param userId 用户id
     * @return 管理员列表
     */
    @Nonnull
    List<Admin> getAdminsByUserId(@Nonnull String userId);

    /**
     * 获取权限组及其内部信息
     *
     * @param userId  用户id
     * @param appCode 应用编码
     * @return 权限组列表
     */
    @Nonnull
    List<PermissionGroup> getPermissionGroups(@Nonnull String userId, @Nonnull String appCode);

    /**
     * 通过用户信息和模型编码获取数据项权限组信息
     *
     * @param userId     用户id
     * @param schemaCode 模型编码
     * @return 数据项权限组列表
     */
    @Nonnull
    List<BizPropertyPermissionGroup> getPropertyPermissionGroups(@Nonnull String userId, @Nonnull String schemaCode);

    /**
     * 构建权限查询条件
     *
     * @param userId     用户id
     * @param schemaCode 模型编码
     * @return 查询条件
     */
    @Nonnull
    FilterExpression getFilterExpression(@Nonnull String userId, @Nonnull String schemaCode);

    /**
     * 根据用户id和数据模型编码获取有权限的业务规则编码集合
     *
     * @param userId     用户id
     * @param schemaCode 模型编码
     * @return 有权限的业务规则编码集合
     */
    @Nonnull
    Set<String> getPermittedBizRuleCodes(@Nonnull String userId, @Nonnull String schemaCode);

    /**
     * 文件上传
     *
     * @param originalFilename 文件名
     * @param inputStream      文件流
     * @return 文件refId
     */
    @Nonnull
    String uploadFile(@Nonnull String originalFilename, @Nonnull InputStream inputStream);

    /**
     * 文件下载
     *
     * @param refId 文件refId
     * @return 文件流
     */
    @Nullable
    InputStream downloadFile(@Nonnull String refId);

    /**
     * 获取附件信息
     *
     * @param refId 文件refId
     * @return 附件信息
     */
    @Nullable
    BizAttachment getBizAttachmentByRefId(@Nonnull String refId);

    /**
     * 根据id 查询字典信息
     *
     * @param id 字典主键id
     * @return 字典模型
     */
    @Nullable
    Dictionary getDictionaryById(@Nonnull String id);

    /**
     * 根据编码查询字典信息
     *
     * @param code 字典编码
     * @return 字典模型
     */
    @Nullable
    Dictionary getDictionaryByCode(@Nonnull String code);

    /**
     * 根据字典id获取字典数据
     *
     * @param dictionaryId 字典id
     * @return 字典数据项列表
     */
    @Nonnull
    List<DictionaryRecord> getDictionaryRecordsByDictionaryId(@Nonnull String dictionaryId);

    /**
     * 根据字典数据ID查询其子级数据
     *
     * @param parentId 字典父级id
     * @return 字典数据项列表
     */
    @Nonnull
    List<DictionaryRecord> getDictionaryRecordsByParentId(@Nonnull String parentId);

    /**
     * 查询系统设置参数
     *
     * @param settingCode 系统设置项编码
     * @return 系统设置项
     */
    @Nullable
    SystemSetting getSystemSettingBySettingCode(@Nonnull String settingCode);

    /**
     * 根据类型查询系统设置参数
     *
     * @param settingType 系统设置类型
     * @return 系统设置项列表
     */
    @Nonnull
    List<SystemSetting> getSystemSettingsBySettingType(@Nonnull SettingType settingType);

    /**
     * 创建job
     *
     * @param groupCode 调度分组编码
     * @param job       调度任务对象
     * @return 调度任务对象
     */
    @Nonnull
    Job createJob(@Nonnull String groupCode, @Nonnull Job job);

    /**
     * 根据组名和任务名更新job
     *
     * @param groupCode 调度分组编码
     * @param code      调度任务编码
     * @param job       调度对象
     * @return 调度对象
     */
    @Nonnull
    Job updateJob(@Nonnull String groupCode, @Nonnull String code, @Nonnull Job job);

    /**
     * 删除job
     *
     * @param groupCode 调度分组编码
     * @param code      调度任务编码
     * @return true/false
     */
    boolean deleteJob(@Nonnull String groupCode, @Nonnull String code);

    /**
     * 获取一个调度分组下所有的jobs
     *
     * @param groupCode 调度分组编码
     * @return 调度分组下所有Job的列表
     */
    @Nonnull
    List<Job> getJobs(@Nonnull String groupCode);

    /**
     * 根据组名和任务名获取job详情
     *
     * @param groupCode 调度分组编码
     * @param code      调度任务编码
     * @return 如果存在则返回Job，否则返回null
     */
    @Nullable
    Job getJob(@Nonnull String groupCode, @Nonnull String code);

    /**
     * 根据groupCode和code暂停一个job
     *
     * @param groupCode 调度分组编码
     * @param code      调度任务编码
     * @return true/false
     */
    boolean pauseJob(@Nonnull String groupCode, @Nonnull String code);

    /**
     * 根据groupCode和code重启一个job
     *
     * @param groupCode 调度分组编码
     * @param code      调度任务编码
     * @return true/false
     */
    boolean resumeJob(@Nonnull String groupCode, @Nonnull String code);

    /**
     * 发送消息
     *
     * @param message 消息模型
     */
    void sendMessage(@Nonnull Message message);

    /**
     * 发送简单文本消息
     *
     * @param message 简单文本消息模型
     */
    void sendSimpleTextMessage(@Nonnull SimpleTextMessage message);

    /**
     * 获取当前用户id
     *
     * @return 用户id
     */
    @Nullable
    String getUserId();

    /**
     * 设置上下文
     *
     * @param context 上下文
     */
    void setContext(OpenEngineContext context);

    /**
     * 获取上下文
     *
     * @return 上下文
     */
    OpenEngineContext getContext();

}
```
