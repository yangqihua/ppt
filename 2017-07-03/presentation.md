<!--markpress-opt

{
	"layout": "horizontal",
	"autoSplit": true,
	"sanitize": false,
	"theme": "light",
	"noEmbed": false
}

markpress-opt-->
# 权限控制及半自动化编程

## 1.权限控制（shiro权限控制）
> [shiro](http://shiro.apache.org/)不仅可以用在JavaSE环境，也可以用在JavaEE环境。Shiro可以帮助我们完成：认证、授权、加密、会话管理、与Web集成、缓存等。相比[Spring Security](http://projects.spring.io/spring-security/)更容易上手。

## 2.半自动化编程
> 半自动化编程在很多项目中将重复的工作交给计算机去完成，开发只专注于业务，可以将开发效率提高到40% ~ 60%。



# 1.权限控制

- what：
  + 1.简单的权限控制：某用户拥有某种角色，某种角色拥有操作某菜单的权限（如某教务系统，admin用户拥有超级管理员角色，超级管理员角色拥有操作所有菜单的权限），这种简单模式存在三种实体，两种关系。
  + 2.更为复杂的权限控制：某用户属于某部门，担任某部门xxx职位，拥有某几种角色，某种角色拥有某几种权限集合，某种权限集合拥有操作多种菜单的权限（如某教务系统，admin为教务处主任，拥有部门管理员角色，部门管理员有部门管理员权限集合，部门管理员权限集合有增删改查某几种菜单的功能）。这种模式操作多种实体，更为复杂的关联关系。
- why：当系统登录用户存在等级区分时，常常需要权限控制。
- how：所有的实体及关联关系都需要数据库表来维护，才能进行动态管理。

# 1-1.shiro实际应用

- 简单的权限控制
  + 1.数据库的设计，三种实体，两种关系，共五张表。
    ```mysql
    user : user_id、username、password
    role : role_id、role_name
    menu : menu_id、parent_id、name、url、auth、type、icon、order_num
    user_role : user_id、role_id
    role_menu : role_id、menu_id
    ```

# 1-1.shiro实际应用

- 简单的权限控制
  + 2.前端使用方式，因vue为例：
    ```
    getMenuList: function (event) {
      $.getJSON("sys/menu/user?_"+$.now(), function(r){
        vm.menuList = r.menuList;
        window.permissions = r.permissions;
      });
    },
    hasPermission: function(permission) {
        if (window.permissions.indexOf(permission) > -1) {
            return true;
        } else {
            return false;
        }
    },
    <div class="grid-btn">
      <a v-if="hasPermission('sys:menu:save')" class="btn btn-primary" @click="add"><i class="fa fa-plus"></i>&nbsp;新增</a>
      <a v-if="hasPermission('sys:menu:update')" class="btn btn-primary" @click="update"><i class="fa fa-pencil-square-o"></i>&nbsp;修改</a>
      <a v-if="hasPermission('sys:menu:delete')" class="btn btn-primary" @click="del"><i class="fa fa-trash-o"></i>&nbsp;删除</a>
    </div>
    ```




# 1-1.shiro实际应用

- 简单的权限控制
  + 3.后台的权限校验：
    ```java
    /**
     * 用户信息
     */
    @RequestMapping("/info/{userId}")
    @RequiresPermissions("sys:user:info")
    public R info(@PathVariable("userId") Long userId){
      SysUserEntity user = sysUserService.queryObject(userId);
      
      //获取用户所属的角色列表
      List<Long> roleIdList = sysUserRoleService.queryRoleIdList(userId);
      user.setRoleIdList(roleIdList);
      
      return R.ok().put("user", user);
    }
    ```

# 1-1.shiro实际应用

- 简单的权限控制
  + 4.[实际案例演示](http://localhost:8088/)

# 2.半自动化编程技术

- what：
  + 不是每一行代码都需要自己手动去编写，一些固定模式的代码可以让计算机来生成，如在常见的spring+mybatis项目中，一般会有controller、service、dao/repository、Entity这四种包，mybatis还需维护其mapper关系，前端有list.html，list.js等文件，一般在cms系统或后台管理系统中，一个controller、service、dao、entity、mapper、list.html、list.js对应一种业务，且这种业务一般会存在增删改查，对应数据库的CURD业务，所以只要找到了规律，重复的工作就可以让计算机来做。
- why：人类擅长做寻找规律的事情，计算机擅长重复的做有规律的事情（ps:两者结合便是AI😂）
- how：需提供该业务数据库表结构。


# 2-1.半自动化编程应用

- 半自动化编程
  + 1.获取数据库表的信息：
    ```
    <select id="queryTable" resultType="map">
      select table_name tableName, engine, table_comment tableComment, create_time createTime from information_schema.tables 
        where table_schema = (select database()) and table_name = #{tableName}
    </select> 
    
    <select id="queryColumns" resultType="map">
      select column_name columnName, data_type dataType, column_comment columnComment, column_key columnKey, extra from information_schema.columns
        where table_name = #{tableName} and table_schema = (select database()) order by ordinal_position
    </select>
    ```
  
# 2-1.半自动化编程应用

- 半自动化编程
  + 2.定义类与前端模板,以controller的部分代码为例
    ```
    /**
     * 修改
     */
    @RequestMapping("/update")
    @RequiresPermissions("${pathName}:update")
    public R update(@RequestBody ${className}Entity ${classname}){
      ${classname}Service.update(${classname});
      
      return R.ok();
    }
    
    /**
     * 删除
     */
    @RequestMapping("/delete")
    @RequiresPermissions("${pathName}:delete")
    public R delete(@RequestBody ${pk.attrType}[] ${pk.attrname}s){
      ${classname}Service.deleteBatch(${pk.attrname}s);
      
      return R.ok();
    }
    ```

# 2-1.半自动化编程应用

- 半自动化编程
  + 3.velocity模板技术生成代码
  ```
    //封装模板数据
    Map<String, Object> map = new HashMap<>();
    map.put("tableName", tableEntity.getTableName());
    map.put("comments", tableEntity.getComments());
    map.put("pk", tableEntity.getPk());
    map.put("className", tableEntity.getClassName());
    map.put("classname", tableEntity.getClassname());
    map.put("pathName", tableEntity.getClassname().toLowerCase());
    map.put("columns", tableEntity.getColumns());
    map.put("package", config.getString("package"));
    map.put("author", config.getString("author"));
    map.put("email", config.getString("email"));
    map.put("datetime", DateUtils.format(new Date(), DateUtils.DATE_TIME_PATTERN));
    VelocityContext context = new VelocityContext(map);
    //获取模板列表
    List<String> templates = getTemplates();
    for(String template : templates){
      //渲染模板
      StringWriter sw = new StringWriter();
      Template tpl = Velocity.getTemplate(template, "UTF-8");
      tpl.merge(context, sw); 
    }
  ```

# 2-1.半自动化编程应用

- 半自动化编程
  + 4.[实际案例演示](http://localhost:8088/)

# 3.The End

## 下期分享
  - 自动化部署&自动化运维
  - 微服务
