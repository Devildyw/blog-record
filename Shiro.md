# Shiro

## 简介

- **Apache Shiro**是一个Java的一个安全(权限)框架。
- Shiro可以非常容易的开发出足够好的应用，其不仅可以用在JavaSE环境，也可以用在JavaEE环境。
- Shiro可以完成：认证、授权、加密、会话管理、与Web集成、缓存等。

## Shiro的功能

基本的功能点如下图所示：

[![img](https://www.docs4dev.com/images/apache-shiro/1.5.3/ShiroFeatures.png)](https://www.docs4dev.com/images/apache-shiro/1.5.3/ShiroFeatures.png)

Shiro 把 Shiro 开发团队称为“应用程序的四大基石”——身份验证，授权，会话管理和加密作为其目标。

- **Authentication**：有时也简称为“登录”，这是一个证明用户是他们所说的他们是谁的行为。
- **Authorization**：访问控制的过程，也就是绝对“谁”去访问“什么”。
- **Session** **Management**：管理用户特定的会话，即使在非 Web 或 EJB 应用程序。
- **Cryptography**：通过使用加密算法保持数据安全同时易于使用。

也提供了额外的功能来支持和加强在不同环境下所关注的方面，尤其是以下这些：

- Web Support：Shiro 的 web 支持的 API 能够轻松地帮助保护 Web 应用程序。
- Caching：缓存是 Apache Shiro 中的第一层公民，来确保安全操作快速而又高效。
- Concurrency：Apache Shiro 利用它的并发特性来支持多线程应用程序。
- Testing：测试支持的存在来帮助你编写单元测试和集成测试，并确保你的能够如预期的一样安全。
- “Run As”：一个允许用户假设为另一个用户身份（如果允许）的功能，有时候在管理脚本很有用。
- “Remember Me”：在会话中记住用户的身份，所以他们只需要在强制时候登录。

## Shiro架构

从外部来看，即从应用程序的角度来观察如何使用Shiro完成工作：

[![img](https://www.docs4dev.com/images/apache-shiro/1.5.3/ShiroBasicArchitecture.png)](https://www.docs4dev.com/images/apache-shiro/1.5.3/ShiroBasicArchitecture.png)

- **Subject**：在我们的教程中已经提到，Subject 实质上是一个当前执行用户的特定的安全“视图”。鉴于”User” 一词通常意味着一个人，而一个 Subject 可以是一个人，但它还可以代表第三方服务，daemon account，cron job，或其他类似的任何东西——基本上是当前正与软件进行交互的任何东西。

所有 Subject 实例都被绑定到（且这是必须的）一个 SecurityManager 上。当你与一个 Subject 交互时，那些交互作用转化为与 SecurityManager 交互的特定 subject 的交互作用。

- **SecurityManager**：SecurityManager 是 Shiro 架构的心脏，并作为一种“保护伞”对象来协调内部的安全组件共同构成一个对象图。然而，一旦 SecurityManager 和它的内置对象图已经配置给一个应用程序，那么它单独留下来，且应用程序开发人员几乎使用他们所有的时间来处理 Subject API。

我们稍后会更详细地讨论 SecurityManager，但重要的是要认识到，当你正与一个 Subject 进行交互时，实质上是幕后的 SecurityManager 处理所有繁重的 Subject 安全操作。这反映在上面的基本流程图。

- **Realms**：Realms 担当 Shiro 和你的应用程序的安全数据之间的“桥梁”或“连接器”。当它实际上与安全相关的数据如用来执行身份验证（登录）及授权（访问控制）的用户帐户交互时，Shiro 从一个或多个为应用程序配置的 Realm 中寻找许多这样的东西。

在这个意义上说，Realm 本质上是一个特定安全的 DAO：它封装了数据源的连接详细信息，使 Shiro 所需的相关的数据可用。当配置 Shiro 时，你必须指定至少一个 Realm 用来进行身份验证和/或授权。SecurityManager 可能配置多个 Realms，但至少有一个是必须的。

Shiro 提供了立即可用的 Realms 来连接一些安全数据源（即目录），如LDAP，关系数据库（JDBC），文本配置源，像 INI 及属性文件，以及更多。你可以插入你自己的 Realm 实现来代表自定义的数据源，如果默认地Realm 不符合你的需求。

像其他内置组件一样，Shiro SecurityManager 控制Realms 是如何被用来获取安全和身份数据来代表 Subject 实例的。

### 内部架构

[![img](https://www.docs4dev.com/images/apache-shiro/1.5.3/ShiroArchitecture.png)](https://www.docs4dev.com/images/apache-shiro/1.5.3/ShiroArchitecture.png)

- **Subject**(org.apache.shiro.subject.Subject)

 当前与软件进行交互的实体（用户，第三方服务，cron job，等等）的安全特定“视图”。

- **SecurityManager**(org.apache.shiro.mgt.SecurityManager)

 如上所述，SecurityManager 是 Shiro 架构的心脏。它基本上是一个“保护伞”对象，协调其管理的组件以确保它们能够一起顺利的工作。它还管理每个应用程序用户的 Shiro 的视图，因此它知道如何执行每个用户的安全操作。

- **Authenticator**(org.apache.shiro.authc.Authenticator)

 Authenticator 是一个对执行及对用户的身份验证（登录）尝试负责的组件。当一个用户尝试登录时，该逻辑被 Authenticator 执行。Authenticator 知道如何与一个或多个 Realm 协调来存储相关的用户/帐户信息。从这些Realm 中获得的数据被用来验证用户的身份来保证用户确实是他们所说的他们是谁。

- **Authentication** **Strategy**(org.apache.shiro.authc.pam.AuthenticationStrategy)

 如果不止一个 Realm 被配置，则 AuthenticationStrategy 将会协调这些 Realm 来决定身份认证尝试成功或失败下的条件（例如，如果一个 Realm 成功，而其他的均失败，是否该尝试成功？ 是否所有的 Realm 必须成功？或只有第一个成功即可？）。

- **Authorizer**(org.apache.shiro.authz.Authorizer)

 Authorizer 是负责在应用程序中决定用户的访问控制的组件。它是一种最终判定用户是否被允许做某事的机制。与 Authenticator 相似，Authorizer 也知道如何协调多个后台数据源来访问角色恶化权限信息。Authorizer 使用 该信息来准确地决定用户是否被允许执行给定的动作。

- **SessionManager**(org.apache.shiro.session.SessionManager)

 SessionManager 知道如何去创建及管理用户 Session 生命周期来为所有环境下的用户提供一个强健的 Session 体验。这在安全框架界是一个独有的特色——Shiro 拥有能够在任何环境下本地化管理用户 Session 的能力， 即使没有可用的 Web/Servlet 或 EJB 容器，它将会使用它内置的企业级会话管理来提供同样的编程体验。SessionDAO 的存在允许任何数据源能够在持久会话中使用。

- **SessionDAO**(org.apache.shiro.session.mgt.eis.SessionDAO)

 SesssionDAO 代表 SessionManager 执行 Session 持久化（CRUD）操作。这允许任何数据存储被插入到会话管理的基础之中。

- **CacheManager**(org.apahce.shiro.cache.CacheManager)

 CacheManager 创建并管理其他 Shiro 组件使用的 Cache 实例生命周期。因为 Shiro 能够访问许多后台数据源， 由于身份验证，授权和会话管理，缓存在框架中一直是一流的架构功能，用来在同时使用这些数据源时提高 性能。任何现代开源和/或企业的缓存产品能够被插入到 Shiro 来提供一个快速及高效的用户体验。

- **Cryptography**(org.apache.shiro.crypto.*)

 Cryptography 是对企业安全框架的一个很自然的补充。Shiro 的crypto 包包含量易于使用和理解的cryptographic Ciphers，Hasher（又名 digests）以及不同的编码器实现的代表。所有在这个包中的类都被精心地设计以易于使用和易于理解。任何使用 Java 的本地密码支持的人都知道它可以是一个难以驯服的具有挑战性的动物。Shiro 的 cryptoAPI 简化了复杂的 Java 机制，并使加密对于普通人也易于使用。

- **Realms**(org.apache.shiro.realm.Realm)

 如上所述，Realms 在 Shiro 和你的应用程序的安全数据之间担当“桥梁”或“连接器”。当它实际上与安全相关的数据如用来执行身份验证（登录）及授权（访问控制）的用户帐户交互时，Shiro 从一个或多个为应用程序配置的Realm 中寻找许多这样的东西。你可以按你的需要配置多个 Realm（通常一个数据源一个 Realm），且 Shiro 将为身份验证和授权对它们进行必要的协调。

## 自定义Realm

自定义`Realm`需要继承`AuthorizingRealm` 并且重写它的两个方法`AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals)`和`AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException`

```JAVA
public class UserRealm extends AuthorizingRealm {


    @Autowired
    private UserService userService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String principal = (String) principals.getPrimaryPrincipal();

        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        //通过我们写的方法将数据库关于该用户的信息全部查出
        List<String> roles = userService.queryUserRoleByUsername(principal);
        List<String> permissions = userService.queryUserPermissionByUsername(principal);
		//这里添加用户的角色权限信息
        authorizationInfo.addRoles(roles);
        authorizationInfo.addStringPermissions(permissions);
        return authorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String principal = (String) token.getPrincipal();
        //通过我们配置的数据源查询用户密码
        //这里主要查询用户的验证信息
        String password = userService.queryUserPasswordByUsername(principal);
        if (StringUtils.isBlank(password)){
            throw new UnknownAccountException("账户不存在");
        }
        return new SimpleAuthenticationInfo(principal,password,getName());
    }
}
```

密码肯定不能用明文存入数据库，所以我们要告诉后续的验证程序我们用的时什么加密算法。

`AuthorizingRealm`提供了一个方法让我们指定Shiro框架采用什么加密算法对传入的token中的密码信息进行加密后与数据库中的密文密码进行验证。

指定的方式有三种

1. 自定Realm的构造方法中指定

   ```JAVA
   public class UserRealm extends AuthorizingRealm {
   
       public UserRealm() {
           //调用AuthorizingRealm提供的方法
           setCredentialsMatcher(new HashedCredentialsMatcher("MD5"));
       }
       //省略下方重写方法了
    	......
   }
   ```

   因为我们在ShiroConfig中会手动创建Bean在加入到IOC容器中所以在构造方法中调用`setCredentialsMatcher`是可行的。

2. 在ShiroConfig中 创建了自定义Realm后在调用`setCredentialsMatcher`方法指定

   ```JAVA
   @Bean
      public UserRealm userRealm() {
          UserRealm userRealm = new UserRealm();
          userRealm.setCredentialsMatcher(new HashedCredentialsMatcher("MD5"));
          return userRealm;
      }
   ```

   本身就是继承了`AuthorizingRealm`，所以父类的方法也都拥有。

3. 自定手动在自定义Realm的内部编写一个方法 然后在其上添加注解`@PostConstruct`

   ```JAVA
   /**
       * @Description 密码匹配器
       */
      @PostConstruct
      public void initCredentialsMatcher(){
          HashedCredentialsMatcher md5 = new HashedCredentialsMatcher("MD5");
          //设置提交的AuthenticationToken的凭据在与存储在系统中的凭据进行比较之前将被散列的次数。 根据用户注册是密码加密的次数的设置相映
          md5.setHashIterations();
          setCredentialsMatcher(md5);
      }
   ```

   `@PostConstruct`

   Java中该注解的说明：@PostConstruct该注解被用来修饰一个非静态的void（）方法。被@PostConstruct修饰的方法会在服务器加载`Servlet`的时候运行，并且只会被服务器执行一次。`@PostConstruct`在构造函数之后执行，`init（）`方法之前执行。

------

## 常用API

### `Subject`



| Subject 登录相关方法 | 描述                                   |
| -------------------- | -------------------------------------- |
| isAuthenticated()    | 返回true 表示已经登录，否则返回false。 |

| Subject 角色相关方法             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| hasRole(String roleName)         | 返回true 如果Subject 被分配了指定的角色，否则返回false。     |
| hasRoles(List roleNames)         | 返回true 如果Subject 被分配了所有指定的角色，否则返回false。 |
| hasAllRoles(CollectionroleNames) | 返回一个与方法参数中目录一致的hasRole 结果的集合。有性能的提高如果许多角色需要执行检查（例如，当自定义一个复杂的视图）。 |
| checkRole(String roleName)       | 安静地返回，如果Subject 被分配了指定的角色，不然的话就抛出AuthorizationException。 |
| checkRoles(CollectionroleNames)  | 安静地返回，如果Subject 被分配了所有的指定的角色，不然的话就抛出AuthorizationException。 |
| checkRoles(String… roleNames)    | 与上面的checkRoles 方法的效果相同，但允许Java5 的var-args 类型的参数 |

| Subject 资源相关方法               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| isPermitted(Permission p)          | 返回true 如果该Subject 被允许执行某动作或访问被权限实例指定的资源，否则返回false |
| isPermitted(List perms)            | 返回一个与方法参数中目录一致的isPermitted 结果的集合。       |
| isPermittedAll(Collectionperms)    | 返回true 如果该Subject 被允许所有指定的权限，否则返回false有性能的提高如果需要执行许多检查（例如，当自定义一个复杂的视图） |
| isPermitted(String perm)           | 返回true 如果该Subject 被允许执行某动作或访问被字符串权限指定的资源，否则返回false。 |
| isPermitted(String…perms)          | 返回一个与方法参数中目录一致的isPermitted 结果的数组。有性能的提高如果许多字符串权限检查需要被执行（例如，当自定义一个复杂的视图）。 |
| isPermittedAll(String…perms)       | 返回true 如果该Subject 被允许所有指定的字符串权限，否则返回false。 |
| checkPermission(Permission p)      | 安静地返回，如果Subject 被允许执行某动作或访问被特定的权限实例指定的资源，不然的话就抛出AuthorizationException 异常。 |
| checkPermission(String perm)       | 安静地返回，如果Subject 被允许执行某动作或访问被特定的字符串权限指定的资源，不然的话就抛出AuthorizationException 异常。 |
| checkPermissions(Collection perms) | 安静地返回，如果Subject 被允许所有的权限，不然的话就抛出AuthorizationException 异常。有性能的提高如果需要执行许多检查（例如，当自定义一个复杂的视图） |
| checkPermissions(String… perms)    | 和上面的checkPermissions 方法效果相同，但是使用的是基于字符串的权限。 |

------

## Web项目下Shiro内置的过滤器

拦截器对应的不同功能。

| 过滤器     | 过滤器类                     | 说明                                                         | 默认   |
| ---------- | ---------------------------- | ------------------------------------------------------------ | ------ |
| **authc**  | **FormAuthenticationFilter** | **基于表单的过滤器；如“/=authc”，如果没有登录会跳到相应的登录页面登录** | **无** |
| **logout** | **LogoutFilter**             | **退出过滤器，主要属性：redirectUrl：退出成功后重定向的地址，如“/logout=logout”** | **/**  |
| **anon**   | **AnonymousFilter**          | **匿名过滤器，即不需要登录即可访问；一般用于静态资源过滤；示例“/static/=anon”** | **无** |

| **过滤器** | **过滤器类**                       | **说明**                                                     | **默认** |
| ---------- | ---------------------------------- | ------------------------------------------------------------ | -------- |
| **roles**  | **RolesAuthorizationFilter**       | **角色授权拦截器，验证用户是否拥有所有角色；主要属性： loginUrl：登录页面地址（/login.jsp）；unauthorizedUrl：未授权后重定向的地址；示例“/admin/=roles[admin]”** | **无**   |
| **perms**  | **PermissionsAuthorizationFilter** | **权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例“/user/=perms[“user:create”]”** | **无**   |
| **port**   | **PortFilter**                     | **端口拦截器，主要属性：port（80）：可以通过的端口；示例“/test= port[80]”，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样** | **无**   |
| **rest**   | **HttpMethodPermissionFilter**     | **rest风格拦截器，自动根据请求方法构建权限字符串（GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create）构建权限字符串；示例“/users=rest[user]”，会自动拼出“user:read,user:create,user:update,user:delete”权限字符串进行权限匹配（所有都得匹配，isPermittedAll）** | **无**   |
| **ssl**    | **SslFilter**                      | **SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口（443）；其他和port拦截器一样；** | **无**   |

## 自定义过滤器

自定义过滤器需要继承`AuthorizationFilter`并重写它的`isAccessAllowed`方法

```JAVA
public class RolesOrAuthorizationFilter extends AuthorizationFilter {


    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        //获取subject 底层调用的同样是SecurityUtils.getSubject()
        Subject subject = getSubject(request, response);
        String[] rolesArray = (String[]) mappedValue;

        if (rolesArray.length==0||rolesArray==null){
            //对于角色没有要求直接返回true
            return true;
        }
        Set<String> roles = CollectionUtils.asSet(rolesArray);
        for (String role : roles) {
            //满足一个身份就返回true
            if (subject.hasRole(role)){
                return true;
            }
        }
        return false;
    }
}
```

### 添加自己的过滤器使其生效

`ShiroConfig.java` 中添加如下

```JAVA
@Configuration
public class ShiroConfiguration {
    //创建 ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        
        //使自定义的过滤器生效 这里添加我们刚刚自定义的过滤器
        shiroFilterFactoryBean.setFilters(filters());
       	
        ......

        return shiroFilterFactoryBean;
    }

	/**
     * @Description 自定义过滤器
     */
    private Map<String, Filter> filters(){
        HashMap<String, Filter> map = new HashMap<>();
        //这里的key 就是后面用于指定过滤器的字段
        map.put("role-or",new RolesOrAuthorizationFilter());
        return map;
    }

	//省略其他配置
    .... 
}
```

## Shiro基础五张表

Shiro最为重要的5张表分别为t_user（用户表）、t_role（角色表）、t_permissions （权限表）、t_user_role（用户角色对应表）、t_role_permissions（角色权限对应表）。

[![在这里插入图片描述](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/2021062816011333.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/2021062816011333.png)

## 快速开始（集成SpringBoot+Mybatis-plus）

整合思路

**这里我使用的是`Shiro-Spring来整合`后面我会使用Shiro官方推出的与SpringBoot整合的jar包进行整合**.

[![在这里插入图片描述](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/20210529132728453.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/20210529132728453.png)

### 建表

Shiro最为重要的5张表分别为t_user（用户表）、t_role（角色表）、t_permissions （权限表）、t_user_role（用户角色对应表）、t_role_permissions（角色权限对应表）。

```SQL
/*
 Navicat Premium Data Transfer

 Source Server         : 阿丁
 Source Server Type    : MySQL
 Source Server Version : 50736
 Source Host           : 124.222.35.20:3319
 Source Schema         : shiro

 Target Server Type    : MySQL
 Target Server Version : 50736
 File Encoding         : 65001

 Date: 24/06/2022 15:51:46
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for permission
-- ----------------------------
DROP TABLE IF EXISTS `permission`;
CREATE TABLE `permission`  (
  `p_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '权限编号',
  `p_name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '权限',
  PRIMARY KEY (`p_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for role
-- ----------------------------
DROP TABLE IF EXISTS `role`;
CREATE TABLE `role`  (
  `r_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '角色编号',
  `r_name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色名称',
  `r_state` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色说明',
  PRIMARY KEY (`r_id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for role_permission
-- ----------------------------
DROP TABLE IF EXISTS `role_permission`;
CREATE TABLE `role_permission`  (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `rid` int(11) NOT NULL COMMENT '角色id',
  `pid` int(11) NOT NULL COMMENT '权限id',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户编号',
  `username` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户名称',
  `password` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户密码',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for user_role
-- ----------------------------
DROP TABLE IF EXISTS `user_role`;
CREATE TABLE `user_role`  (
  `id` int(11) NOT NULL COMMENT '主键',
  `uid` int(11) NOT NULL COMMENT '用户id',
  `rid` int(11) NOT NULL COMMENT '角色id',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```

### 项目结构

[![image-20220624154218271](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220624154218271.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220624154218271.png)

### maven依赖

`pom.xml`添加相关依赖

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.dyw</groupId>
    <artifactId>Shiro-SpringBoot-Demo01</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <artifactId>spring-boot-starter-parent</artifactId>
        <groupId>org.springframework.boot</groupId>
        <version>2.6.7</version>
    </parent>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--thymeleaf模板-->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring5</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-core</artifactId>
            <version>5.8.0.M1</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.9</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.9.0</version>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-java8time</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.6.7</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
application.yml
YML
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://yourip:3306/shiro?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: username
    password: password
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

  mapper-locations: classpath*:org/dyw/shiro/mapper/xml/*.xml
  type-aliases-package: com.dyw.shiro.entity
```

`因为Shiro-Spring`对于SpringBoot没有特殊的适配所以application.yml中无需对shiro进行文本配置。但是我们可以再Spring.xml中配置或是使用配置类的方式进行配置

### 实体类

这里我们只做演示，所以权限的指定直接在数据库中添加，这里只建用户表（因为无论如何它的信息是要从数据库中获取并且用来验证的）。

```JAVA
@Data
@TableName(value = "user", autoResultMap = true)
public class User {
    private Integer id;

    private String username;

    private String password;
}
```

### Service类

`UserService.java` 作用就是从数据库中查询用户相关的验证/鉴权信息

```JAVA
public interface UserService {
    /**
     * 根据用户名查找用户密码
     * @param username 用户名
     * @return password
     */
    String queryUserPasswordByUsername(String username);

    List<String> queryUserPermissionByUsername(String username);

    List<String> queryUserRoleByUsername(String username);
}
```

`UserServiceImpl.java`

```JAVA
@Service
public class UserServiceImpl implements UserService {
    @Resource
    private UserMapper userMapper;

    @Override
    public String queryUserPasswordByUsername(String username) {
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(User::getUsername,username).last("limit 1");
        return userMapper.selectOne(queryWrapper).getPassword();
    }

    @Override
    public List<String> queryUserPermissionByUsername(String username) {
        return userMapper.selectUserPermissionById(username);
    }

    @Override
    public List<String> queryUserRoleByUsername(String username) {
        return userMapper.selectRoleById(username);
    }
}
```

`UserMapper.java`


```JAVA
public interface UserMapper extends BaseMapper<User> {
    @Select("SELECT permission.p_name FROM ((`user` inner JOIN user_role on `user`.id = user_role.uid) INNER JOIN role_permission on user_role.rid = role_permission.rid) INNER JOIN permission on role_permission.pid = permission.p_id where user.username = #{username}")
    List<String> selectUserPermissionById(String username);

    @Select("select role.r_name  from (user inner join user_role on user.id = user_role.uid) inner join role on user_role.rid = role.r_id where user.username = #{username}")
    List<String> selectRoleById(String username);
}
```


`LoginService.java `用于登录

```JAVA
public interface LoginService {

    Boolean login(LoginDTO loginDTO);
}
```

`LoginServiceImpl`

```JAVA
@Service
public class LoginServiceImpl implements LoginService {
    @Override
    public Boolean login(LoginDTO loginDTO) {
        Subject subject = SecurityUtils.getSubject();
        String username = loginDTO.getUsername();
        String password = loginDTO.getPassword();
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        subject.login(token);
        if (subject.isAuthenticated()){
            return true;
        }
        return false;
    }
}
```


### 自定义Realm

作用就是获取安全数据。

自定义`Realm`需要继承`AuthorizingRealm` 并且重写它的两个方法`AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals)`和`AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException`

其中的`doGetAuthenticationInfo`方法shi就是`Authenticator`交给Realm做验证操作的方法，通过方法名字我们也可得知，我们自定义的重写它主要是将数据库中关于该用户的信息查出做一些简单验证，然后将密码用户名和Realm名封装交给后续的处理做验证。

`doGetAuthorizationInfo`方法主要是用来获取用户的权限信息的，通过方法名同样可知，我们自定义的重写它主要是将数据库中关于该用户的权限信息和角色信息查出然后进行打包再返回交给后续程序处理。

```JAVA
public class UserRealm extends AuthorizingRealm {


    @Autowired
    private UserService userService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String principal = (String) principals.getPrimaryPrincipal();

        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        //通过我们写的方法将数据库关于该用户的信息全部查出
        List<String> roles = userService.queryUserRoleByUsername(principal);
        List<String> permissions = userService.queryUserPermissionByUsername(principal);

        authorizationInfo.addRoles(roles);
        authorizationInfo.addStringPermissions(permissions);
        return authorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String principal = (String) token.getPrincipal();
        //通过我们配置的数据源查询用户密码
        String password = userService.queryUserPasswordByUsername(principal);
        if (StringUtils.isBlank(password)){
            throw new UnknownAccountException("账户不存在");
        }
        return new SimpleAuthenticationInfo(principal,password,getName());
    }
}
```

### ShiroConfig

**这部分非常重要**

1. 将我们刚刚自定义的Realm手动配置加入Spring容器管理

   ```JAVA
   @Bean
   public UserRealm userRealm() {
       UserRealm userRealm = new UserRealm();
       //这里指定了加密算法为MD5
       userRealm.setCredentialsMatcher(new HashedCredentialsMatcher("MD5"));
       return userRealm;
   }
   ```

   我们创建UserRealm对象的时候指定了密码匹配器，到时后框架会将前端传入的密码进行指定算法的加密然后与数据库中查出的密码进行对比。

2. 创建`DefaultWebSecurityManager`对象将我们刚刚创建UserRealm注入并且将其添加到securityManager中，然后将`DefaultWebSecurityManager`以`securityManager`的实例名称加入IOC容器中。

   ```JAVA
   @Bean(name = "securityManager")
   public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm) {
       DefaultWebSecurityManager securityManager = new
               DefaultWebSecurityManager();
       //关联Realm
       securityManager.setRealm(userRealm);
       return securityManager;
   }
   ```

   `DefaultWebSecurityManager `有着许多默认的配置能够满足我们的一般情况下的需求，但是当需要更为灵活的配置时，它也是支持自定义修改的，我们只需要实现它组件的接口或是继承类重写它的方法，再将自定义的组件加入`DefaultWebSecurityManager中即可`。

   [![image-20220624162519708](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220624162519708.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220624162519708.png)

   这里我们就重写了Realm实现了一个自定义的Realm，并且将其加入了其中。

   到这里我们对于SecurityManager就已经结束了，我们的Security会自动地注入到SecurityUtils中。

3. ShiroFilterFactoryBean 创建Shiro的过滤器工厂，在这里面我们可以指定一些restAPI与权限的对应关系，以及其他的配置；

   [![image-20220624163304667](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220624163304667.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220624163304667.png)

   ```JAVA
   @Bean
   public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
       ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
       //设置安全管理器
       shiroFilterFactoryBean.setSecurityManager(securityManager);
           /*
               添加Shiro内置过滤器，常用的有如下过滤器：
               anon： 无需认证就可以访问
               authc： 必须认证才可以访问
               user： 如果使用了记住我功能就可以直接访问
               perms: 拥有某个资源权限才可以访问
               role： 拥有某个角色权限才可以访问
               */
       Map<String, String> filterMap = new LinkedHashMap<>();
       filterMap.put("/login","anon");
       filterMap.put("/user/add", "perms[insert]");
       filterMap.put("/user/update", "authc");
   
   
       shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
   
       //修改到要跳转的login页面；
       shiroFilterFactoryBean.setLoginUrl("/toLogin");
   
       return shiroFilterFactoryBean;
   }
   ```

拦截器对应的不同功能。

| Filter Name           | 功能                                                         |
| --------------------- | ------------------------------------------------------------ |
| **anno**              | **不需要授权、登录就可以访问。eg:/index**                    |
| **authc**             | **需要登录授权才能访问。eg：/用户中心**                      |
| **authcBasic**        | **Basic HTTP身份验证拦截器**                                 |
| **logout**            | **退出拦截器。退出成功后，会 redirect到设置的/URI**          |
| **noSessionCreation** | **不创建会话连接器**                                         |
| **perms**             | **授权拦截器:perm[‘user:create’]**                           |
| **port**              | **端口拦截器.eg:port[80]**                                   |
| **rest**              | **rest风格拦截器**                                           |
| **roles**             | **角色拦截器。eg：role[administrator]**                      |
| **ssl**               | **ssl拦截器。通过https协议才能通过**                         |
| **user**              | **用户拦截器。eg：登录后（authc），第二次没登陆但是有记住我(remmbner)都可以访问。** |

完整的配置

```JAVA
@Configuration
public class ShiroConfiguration {
    //创建 ShiroFilterFactoryBean
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(securityManager);
            /*
                添加Shiro内置过滤器，常用的有如下过滤器：
                anon： 无需认证就可以访问
                authc： 必须认证才可以访问
                user： 如果使用了记住我功能就可以直接访问
                perms: 拥有某个资源权限才可以访问
                role： 拥有某个角色权限才可以访问
                */
        Map<String, String> filterMap = new LinkedHashMap<>();
        filterMap.put("/login","anon");
        filterMap.put("/user/add", "perms[insert]");
        filterMap.put("/user/update", "authc");


        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);

        //修改到要跳转的login页面；
        shiroFilterFactoryBean.setLoginUrl("/toLogin");

        return shiroFilterFactoryBean;
    }


    //创建 DefaultWebSecurityManager( 步骤二
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new
                DefaultWebSecurityManager();
        //关联Realm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //创建 realm 对象( 步骤一
    @Bean
    public UserRealm userRealm() {
        UserRealm userRealm = new UserRealm();
        userRealm.setCredentialsMatcher(new HashedCredentialsMatcher("MD5"));
        return userRealm;
    }

}
```

到这里就整合完毕了Controller可以自己根据自己想法定义，restfulAPI的过滤操作可以添加到该类中。

### 启用Shiro注解进行鉴权

**如果导入的是`shiro-spring-boot-web-starter`则无需进行如下配置**

针对**`shiro-spring`**要使用Shiro注解进行鉴权需要在ShiroConfig中加上添加如下的配置

```JAVA
/**
 * @Description 保证实现了Shiro内部lifecycle函数的bean执行
 */
@Bean(name = "lifecycleBeanPostProcessor")
public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
    return new LifecycleBeanPostProcessor();
}

/**
 * @Description AOP式方法级权限检查
 */
@Bean
@DependsOn("lifecycleBeanPostProcessor")
public DefaultAdvisorAutoProxyCreator getDefaultAdvisorAutoProxyCreator() {
    DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
    defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
    return defaultAdvisorAutoProxyCreator;
}

/**
 * @Description 配合DefaultAdvisorAutoProxyCreator事项注解权限校验
 */
@Bean
public AuthorizationAttributeSourceAdvisor getAuthorizationAttributeSourceAdvisor(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
    AuthorizationAttributeSourceAdvisor aasa = new AuthorizationAttributeSourceAdvisor();
    aasa.setSecurityManager(securityManager);
    return new AuthorizationAttributeSourceAdvisor();
}
```

加上之后可以使用注解了

以下为常用注解

| 注解                    | 说明                               |
| ----------------------- | ---------------------------------- |
| @RequiresAuthentication | 表明当前用户需是经过认证的用户     |
| @ RequiresGuest         | 表明该用户需为”guest”用户          |
| @RequiresPermissions    | 当前用户需拥有指定权限             |
| @RequiresRoles          | 当前用户需拥有指定角色             |
| @ RequiresUser          | 当前用户需为已认证用户或已记住用户 |

```JAVA
@RequiresRoles("admin")
@GetMapping("/say")
public void say(){
    System.out.println("==================");
}
```

## Shiro整合Spring常用注解

以下为常用注解

| 注解                        | 说明                                   |
| --------------------------- | -------------------------------------- |
| **@RequiresAuthentication** | **表明当前用户需是经过认证的用户**     |
| **@ RequiresGuest**         | **表明该用户需为”guest”用户**          |
| **@RequiresPermissions**    | **当前用户需拥有指定权限**             |
| **@RequiresRoles**          | **当前用户需拥有指定角色**             |
| **@ RequiresUser**          | **当前用户需为已认证用户或已记住用户** |

## Realm采用Redis作为缓存

### Spring容器工具类

```JAVA
@Component
public class ApplicationContextUtils implements ApplicationContextAware{
    //放置在获取bean的时候提示空指针，将其定义为静态变量
    private static ApplicationContext context;

   //类初始化完成之后调用setApplicationContext()方法进行操作
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ApplicationContextUtils.context = applicationContext;
    }

   public static ApplicationContext geteContext(){
        return context;
    }

   public static Object getBean(String beanName){
        //在这一步的时候一定要注意，此时可调用这个方法的时候
        //context可能为空，会提示空指针异常，需要将其定义成静态的，这样类加载的时候
        //context就已经存在了
        return context.getBean(beanName);
    }
}
```

**通过这个工具类我们可以获取到容器中的Bean 下面我们会用到**

在认证和授权的时候，程序需要频繁的访问数据库，这样对于数据库的压力可想而知，那我们怎么处理呢？

我们可以使用Redis作为缓存来减轻数据库的压力。

Redis的配置不多做介绍

1. 导入`spring-boot-starter-data-redis`

```XML
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-pool2</artifactId>
   </dependency>
```

1. `RedisConfig`Redis配置类配置

 这个类里我们实现了设置缓存、获取缓存、移除缓存、情况缓存等方法。
其中我们开启缓存后，用户登录成功就会将缓存放入到redis中，使用退出功能，就会清楚当前登录的缓存信息，授权信息也是一样，只要使用退出功能就会清空当前的缓存信息。但是这里并没有设计过期时间的处理。所以真实场景下，我们还需要考虑过期时间的设置。这里显然我们用到了RedisTemplate模板，这个模板我们一般也是自己定义，不过也可以直接使用SpringBoot默认提供的。这里我们采用自己定义RedisTemplate的方式。

```JAVA
@Bean("redisTemplate")
public StringRedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory){
        StringRedisTemplate redisTemplate = new StringRedisTemplate(redisConnectionFactory);

        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        JdkSerializationRedisSerializer jdkSerializationRedisSerializer = new JdkSerializationRedisSerializer();
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        redisTemplate.setKeySerializer(stringRedisSerializer);              //键值序列化方式
        redisTemplate.setValueSerializer(jdkSerializationRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);          //绑定hash的序列化方式
        redisTemplate.setHashValueSerializer(jdkSerializationRedisSerializer);
//		redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;

    }
	
	//redisson配置类
	@Bean
    public RedissonClient redissonClient() {
        Config config = new Config();

        //指定编码，默认编码为org.redisson.codec.JsonJacksonCodec
        //config.setCodec(new org.redisson.client.codec.StringCodec());
        config.useSingleServer()
                .setAddress("redis://124.222.35.20:6666")
                .setPassword("dyw20020304")
                .setConnectionPoolSize(50)
                .setIdleConnectionTimeout(10000)
                .setConnectTimeout(3000)
                .setTimeout(3000)
                .setDatabase(1);

        return Redisson.create(config);
    }
```

1. 提供自定义缓存管理器 `RedisCacheManager`

 只要加入了缓存管理器，配置了缓存管理类，系统就会默认在查询完认证和授权后将信息放入到缓存中且下次需要认证和授权时，都是优先去查询缓存中的内容，查询不到，才会去查询数据库，这里也验证了这一点，与之前的画的加入缓存后的授权信息的获取图是一样的。

```JAVA
public class RedisCacheManager implements CacheManager {
    @Override
    public <K, V> Cache<K, V> getCache(String s) throws CacheException {
        System.out.println("进入到了自定义缓存管理器,传入参数cacheName："+ s);
        return new RedisCache<K,V>(s);
    }
}
```

1. 设计自己的缓存管理类 实现shiro的Cache<K,V>接口

 所有的缓存管理类的实例都应该是Cache的实现类。所以我们自己定义redis的缓存管理类应该也必须去实现这个Cache类。实现如下：

```JAVA
@NoArgsConstructor
public class RedisCache<k,V> implements Cache<k,V> {

    private String cacheName;

    public RedisCache (String cacheName){
        this.cacheName = cacheName;
    }

    //获取redis操作对象
    public RedisTemplate getRedisTemplate(){
        RedisTemplate redisTemplate = (RedisTemplate) ApplicationContextUtils.getBean("redisTemplate");
        return redisTemplate;
    }
    @Override
    public V get(k k) throws CacheException {
        System.out.println(cacheName+":获取缓存方法，传入参数：" + k+",此时的redisTemplate:"+getRedisTemplate());
        return (V) getRedisTemplate().opsForHash().get(cacheName,k.toString());
    }

    @Override
    public V put(k k, V v) throws CacheException {
        System.out.println("加入缓存方法，传入参数 K:" + k+",V:"+v);
        //放入redis中的值，一定要是序列化的对象
        getRedisTemplate().opsForHash().put(cacheName.toString(),k.toString(),v);
        return null;
    }

    @Override
    public V remove(k k) throws CacheException {
        System.out.println("调用了remove方法,传入参数："+k.toString());
        getRedisTemplate().opsForHash().delete(cacheName,k.toString());
        return null;
    }

    @Override
    public void clear() throws CacheException {
        System.out.println("调用了clear方法");
        getRedisTemplate().opsForHash().delete(cacheName);
    }

    @Override
    public int size() {
        return getRedisTemplate().opsForHash().size(cacheName).intValue();
    }

    @Override
    public Set<k> keys() {
        return getRedisTemplate().opsForHash().keys(cacheName);
    }

    @Override
    public Collection<V> values() {
        return getRedisTemplate().opsForHash().values(cacheName);
    }
}
```

5.

 开启Realm的缓存

```JAVA
@Bean
public UserRealm userRealm() {
       UserRealm userRealm = new UserRealm();
       userRealm.setCredentialsMatcher(new HashedCredentialsMatcher("MD5"));
       //开启缓存
       userRealm.setAuthenticationCachingEnabled(true);
       userRealm.setAuthorizationCacheName("authenticationCache"); //设置缓存名称--认证
       userRealm.setAuthenticationCacheName("authorizationCache"); //设置缓存名称--授权
       userRealm.setCacheManager(new RedisCacheManager());//设置为我们刚刚自定义的RedisCacheManager
       return userRealm;
   }
```

经过验证发现,在用户登录过后访问需要权限的资源后,确实不再走数据库,而是从Redis缓存中查找信息。

## Shiro共享Session

### 会话的问题

[![1581846589128](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/1581846589128.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/1581846589128.png)

### 分布式会话实现思路

所有服务器的session信息都存储到了同一个Redis集群中，即所有的服务都将 Session 的信息存储到 Redis 集群中，无论是对 Session 的注销、更新都会同步到集群中，达到了 Session 共享的目的。

[![1581846769926](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/1581846769926.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/1581846769926.png)

 Cookie 保存在客户端浏览器中，而 Session 保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上，这就是 Session。客户端浏览器再次访问时只需要从该 Session 中查找该客户的状态就可以了。

 在实际工作中我们建议使用外部的缓存设备(包括Redis)来共享 Session，避免单个服务器节点挂掉而影响服务，共享数据都会放到外部缓存容器中

### 实现

1. 编写RedisSessionDao继承AbstractSessionDAO，重写了会话的创建、读取、修改等操作，全部缓存与redis中。

   ```JAVA
   @Slf4j
   @Component
   public class RedisSessionDAO extends AbstractSessionDAO {
       @Autowired
       private RedisTemplate redisTemplate;
       @Override
       protected Serializable doCreate(Session session) {
           Serializable sessionId = generateSessionId(session);
           assignSessionId(session,sessionId);
   
           redisTemplate.opsForValue().set(session.getId().toString(),session,60, TimeUnit.MINUTES);
           return sessionId;
       }
   
       @Override
       protected Session doReadSession(Serializable sessionId) {
   
           return sessionId == null ? null : (Session) redisTemplate.opsForValue().get(sessionId.toString());
       }
   
       @Override
       public void update(Session session) throws UnknownSessionException {
           if (session!=null&&session.getId()!=null){
               session.setTimeout(3600 * 1000);
           }
           redisTemplate.opsForValue().set(session.getId().toString(),session,60,TimeUnit.MINUTES);
       }
   
       @Override
       public void delete(Session session) {
           if (session!=null&session.getId()!=null){
               redisTemplate.opsForValue().getOperations().delete(session.getId().toString());
           }
       }
   
       @Override
       public Collection<Session> getActiveSessions() {
           return redisTemplate.keys("*");
       }
   }
   ```

   可以根据需求做更多灵活性的改动。

2. ShiroConfig将RedisSessionDao注入SessionManager中

   ```JAVA
   @Bean
   public DefaultWebSessionManager defaultWebSessionManager(RedisSessionDAO redisSessionDAO) {
          DefaultWebSessionManager defaultWebSessionManager = new DefaultWebSessionManager();
          defaultWebSessionManager.setGlobalSessionTimeout(3600 * 1000);
          defaultWebSessionManager.setDeleteInvalidSessions(true);
          defaultWebSessionManager.setSessionDAO(redisSessionDAO);
          defaultWebSessionManager.setSessionValidationSchedulerEnabled(true);
          /**
           * 修改Cookie中的SessionId的key，默认为JSESSIONID，自定义名称
           */
          defaultWebSessionManager.setSessionIdCookie(new SimpleCookie("JSESSIONID"));
          return defaultWebSessionManager;
      }
   ```

3. 将SessionManager注入SecurityManager中

   ```JAVA
   @Bean(name = "securityManager")
   public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm, RedisSessionDAO redisSessionDao) {
       DefaultWebSecurityManager securityManager = new
               DefaultWebSecurityManager();
       //关联Realm
       securityManager.setRealm(userRealm);
      
       // 取消Cookie中的RememberMe参数
       securityManager.setRememberMeManager(null);
       securityManager.setSessionManager(defaultWebSessionManager(redisSessionDao));
       return securityManager;
   }
   ```

4. 做完上述操作在进行访问我们可以看到我们的session已经存入了redis中

## 限制密码重试次数

### 实现原理

保证原子性：

 单系统：AtomicLong计数

 集群系统：使用Redis提供的RAtomicLong计数

```TEX
1、获取系统中是否已有登录次数缓存,缓存对象结构预期为："用户名--登录次数"。

2、如果之前没有登录缓存，则创建一个登录次数缓存。

3、如果缓存次数已经超过限制，则驳回本次登录请求。

4、将缓存记录的登录次数加1,设置指定时间内有效

5、验证用户本次输入的帐号密码，如果登录登录成功，则清除掉登录次数的缓存
```

计数缓存的部分主要是在登录验证密码的时候。所以我们需要自定义一个密码比较器。

### 实现

1. 自定义密码比较器RetryLimitCredentialsMatcher继承HashedCredentialsMatcher重写doCredentialsMatch方法

```JAVA
@NoArgsConstructor
@Slf4j
@Component
public class RetryLimitCredentialsMatcher extends HashedCredentialsMatcher {

    private static Long RETRY_LIMIT_NUM = 4L;

    public RedisTemplate getRedisTemplate(){
        RedisTemplate redisTemplate = (RedisTemplate) ApplicationContextUtils.getBean("redisTemplate");
        return redisTemplate;
    }
    
    public RetryLimitCredentialsMatcher(String hashAlgorithmName){
        super(hashAlgorithmName);
    }

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        log.info("进入自定义密码比较器");
        //获得用户名
        String loginName = (String) token.getPrincipal();
        //获得缓存
        log.info(loginName);
        RedisAtomicLong atomicLong = new RedisAtomicLong(loginName, getRedisTemplate().getConnectionFactory());
        long retryFlag = atomicLong.get();

        //判断次数
        if (retryFlag>RETRY_LIMIT_NUM){
            //超过次数设计10分钟后重试
            atomicLong.expire(10, TimeUnit.MINUTES);
            log.error("密码错误5次，请10分钟以后再试");
            throw new ExcessiveAttemptsException("密码错误5次，请10分钟以后再试");
        }
        //没有超过
        //累加次数
        atomicLong.incrementAndGet();
        atomicLong.expire(10,TimeUnit.MINUTES);
        //密码校验
        boolean flag = super.doCredentialsMatch(token, info);
        log.info("{}",atomicLong.get());
        //如果登录成功 清除缓存
        if (flag){
            atomicLong.expire(0,TimeUnit.MILLISECONDS);
        }
        return flag;
    }


}
```

编写完成后记得将我们一开始设置的Shiro自带的密码比较器更换为我们自定义的。

```JAVA
userRealm.setCredentialsMatcher(new RetryLimitCredentialsMatcher("MD5"));
```

## 在线并发登录人数控制

### 原理

在实际开发中，我们可能会遇到这样的需求，一个账号只允许同时一个在线，当账号在其他地方登陆的时候，会踢出前面登陆的账号，那我们怎么实现

- 自定义过滤器:继承AccessControlFilter
- 使用redis队列控制账号在线数目

实现步骤：

```TEX
1、只针对登录用户处理，首先判断是否登录
2、使用RedissionClien创建队列
3、判断当前sessionId是否存在于此用户的队列=key:登录名 value：多个sessionId
4、不存在则放入队列尾端==>存入sessionId
5、判断当前队列大小是否超过限定此账号的可在线人数
6、超过：
	*从队列头部拿到用户sessionId
	*从sessionManger根据sessionId拿到session
	*从sessionDao中移除session会话
7、未超过：放过操作
```

### 实现

`AccessControlFilter：`控制对资源的访问的任何过滤器的超类，如果用户未通过身份验证，则可以将用户重定向到登录页面。这个超类提供了方法，借助它可以在用户超出登录最大在线人数后将其的session删除并且重定向到登录页面。

```JAVA
@Component
@Slf4j
public class KickedOutAuthorizationFilter extends AccessControlFilter {
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return false;
    }

    @Resource
    private RedissonClient redissonClient;
    @Resource
    private DefaultWebSessionManager sessionManager;
    @Resource
    private RedisSessionDAO redisSessionDAO;

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {

        Subject subject = getSubject(request, response);
        if (!subject.isAuthenticated()) {
            //如果没有登录，直接进行之后的流程
            return true;
        }
        //存放session对象进入队列
        String sessionId = (String) subject.getSession().getId();
        String LoginName = (String) subject.getPrincipal();
        RDeque<String> queue = redissonClient.getDeque("KickedOutAuthorizationFilter:"+LoginName);
        //判断sessionId是否存在于此用户的队列中
        boolean flag = queue.contains(sessionId);
        if (!flag) {
            queue.addLast(sessionId);
        }
        //如果此时队列大于1，则开始踢人
        if (queue.size() > 1) {
            sessionId = queue.getFirst();
            queue.removeFirst();
            Session session = null;
            try {
                session = sessionManager.getSession(new DefaultSessionKey(sessionId));
            }catch (UnknownSessionException ex){
                log.info("session已经失效");
            }catch (ExpiredSessionException expiredSessionException){
                log.info("session已经过期");
            }
            if (session!=null){
                redisSessionDAO.delete(session);
            }
        }
        return true;
    }
}
```

1. 修改ShiroConfig将我们自定义的过滤器添加进ShiroFilterFactoryBean中

   ```JAVA
   @Resource
   private KickedOutAuthorizationFilter kickedOutAuthorizationFilter;
   /**
     * @Description 自定义过滤器定义
     */
   private Map<String, Filter> filters() {
       HashMap<String, Filter> map = new HashMap<>();
       map.put("kickedOut",kickedOutAuthorizationFilter);
       return map;
   }
   ```

   1. 拦截所有需要登录的路径

2. 测试有效

## SpringBoot+Shiro+Jwt整合

### 问题追踪

 前面我们实现分布式的会话缓存，但是我们发现此功能的实现是基于浏览的cookie机制，也就是说用户禁用cookie后，我们的系统会就会产生会话不同的问题

### 解决方案

 我们的前端可能是web、Android、ios等应用，同时我们每一个接口都提供了无状态的应答方式，这里我们提供了基于JWT的token生成方案

```TEX
1、用户登陆之后，获得此时会话的sessionId,使用JWT根据sessionId颁发签名并设置过期时间(与session过期时间相同)返回token

2、将token保存到客户端本地，并且每次发送请求时都在header上携带JwtToken

3、ShiroSessionManager继承DefaultWebSessionManager，重写getSessionId方法，从header上检测是否携带JwtToken，如果携带，则进行解码JwtToken，使用JwtToken中的jti作为SessionId。

4、重写shiro的默认过滤器，使其支持jwtToken有效期校验、及对JSON的返回支持
	JwtAuthcFilter:实现是否需要登录的过滤，拒绝时如果header上携带JwtToken,则返回对应json
	JwtPermsFilter:实现是否有对应资源的过滤，拒绝时如果header上携带JwtToken,则返回对应json
	JwtRolesFilter:实现是否有对应角色的过滤，拒绝时如果header上携带JwtToken,则返回对应json
```

### JWT概述

JWT（JSON WEB TOKEN）：JSON网络令牌，JWT是一个轻便的安全跨平台传输格式，定义了一个紧凑的自包含的方式在不同实体之间安全传输信息（JSON格式）。它是在Web环境下两个实体之间传输数据的一项标准。实际上传输的就是一个字符串。

- 广义上：JWT是一个标准的名称；
- 狭义上：JWT指的就是用来传递的那个token字符串

JWT由三部分构成：header（头部）、payload（载荷）和signature（签名）。

1. Header

   存储两个变量

   1. 秘钥（可以用来比对）
   2. 算法（也就是下面将Header和payload加密成Signature）

2. payload

   存储很多东西，基础信息有如下几个

   1. 签发人，也就是这个“令牌”归属于哪个用户。一般是`userId`
   2. 创建时间，也就是这个令牌是什么时候创建的
   3. 失效时间，也就是这个令牌什么时候失效(session的失效时间)
   4. 唯一标识，一般可以使用算法生成一个唯一标识（jti==>sessionId）

3. Signature

   这个是上面两个经过Header中的算法加密生成的，用于比对信息，防止篡改Header和payload

然后将这三个部分的信息经过加密生成一个`JwtToken`的字符串，发送给客户端，客户端保存在本地。当客户端发起请求的时候携带这个到服务端(可以是在`cookie`，可以是在`header`)，在服务端进行验证，我们需要解密对于的payload的内容

### 代码实现

1. 首先导入Jwt的依赖

   ```XML
   <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt-api</artifactId>
       <version>0.11.5</version>
   </dependency>
   <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt-impl</artifactId>
       <version>0.11.5</version>
   </dependency>
   ```

2. 编写一个Jwt的工具类用于签发令牌,验证令牌等。

   这个可以根据实际项目情况进行自定义更改（例如JWT密钥的指定）

   ```JAVA
   public class JwtTokenUtil {
   
       /**
        * jwt加密密钥 
        */
       private static final String JWT_SECRET = "aPbOBbnH4gnZBzIYEY7mxWNu49kYljNPMeva9Fjrwwqzw0bFlO0kPXZTCGaVcw0j";
   
       /**
        * @Description 签发令牌
        *      jwt字符串包括三个部分
        *        1. header
        *            -当前字符串的类型，一般都是“JWT”
        *            -哪种算法加密，“HS256”或者其他的加密算法
        *            所以一般都是固定的，没有什么变化
        *        2. payload
        *            一般有四个最常见的标准字段（下面有）
        *            iat：签发时间，也就是这个jwt什么时候生成的
        *            jti：JWT的唯一标识
        *            iss：签发人，一般都是username或者userId
        *            exp：过期时间
        * @param iss 签发人
        * @param ttlMillis 有效时间
        * @param id jwt中存储的用户id
        * @return jws
        */
       public static String IssuedToken(String iss, long ttlMillis,String sessionId, Integer id) {
           HashMap<String, Object> claims = new HashMap<>();
           if (id != null) {
               claims.put("userId",id);
           }
           SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
   
           long nowMillis = System.currentTimeMillis();
           SecretKey secretKey = generalKey();
   
           JwtBuilder builder = Jwts.builder()
                   .setClaims(claims)
                   .setId(sessionId)//2. 这个是JWT的唯一标识，一般设置成唯一的，这个方法可以生成唯一标识,此时存储的为sessionId,登录成功后回写
                   .setIssuedAt(new Date(nowMillis))//1. 这个地方就是以毫秒为单位，换算当前系统时间生成的iat
                   .setSubject(iss)//3. 签发人，也就是JWT是给谁的（逻辑上一般都是username或者userId）
                   .signWith(signatureAlgorithm, secretKey);//这个地方是生成jwt使用的算法和秘钥
           if (ttlMillis >= 0) {
               long expMillis = nowMillis + ttlMillis;
               Date exp = new Date(expMillis);//4. 过期时间，这个也是使用毫秒生成的，使用当前时间+前面传入的持续时间生成
               builder.setExpiration(exp);
           }
           return builder.compact();
       }
   
       /**
        * @Description 解析令牌
        * @param jwtToken 令牌
        * @return
        */
       public static Claims decodeToken(String jwtToken) {
   
           SecretKey secretKey = generalKey();
   
           // 得到 DefaultJwtParser
           return Jwts.parser()
                   // 设置签名的秘钥
                   .setSigningKey(secretKey)
                   // 设置需要解析的 jwt
                   .parseClaimsJws(jwtToken)
                   .getBody();
       }
   
       /**
        * 由字符串生成加密key
        *
        * @return SecretKey
        */
       public static SecretKey generalKey() {
           String stringKey = JWT_SECRET;
           byte[] encodedKey = BaseEncoding.base64().decode(stringKey);
           SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "HmacSHA256");
           return key;
       }
   
   }
   ```

3. 继承`DefaultWebSessionManager`重写`getSessionId`方法

   `getSessionId()`方法就是Shiro用于获取SessionId的方法，默认是从cookie中获取，但是我们采用JWT了，就得从Jwt中获取所以我们对该方法进行了修改。

   ```JAVA
   @Slf4j
   public class ShiroSessionManager extends DefaultWebSessionManager {
   
       private static final String AUTHORIZATION = "token";
   
       private static final String REFERENCED_SESSION_ID_SOURCE = "Stateless request";
   
       public ShiroSessionManager(){
           super();
       }
   
   
       /**
        * 这个方法主要是让我们自定义获取sessionId的方法 在将sessionId返回让后续的流程去获取session 然后鉴权
        * @param request
        * @param response
        * @return
        */
       @Override
       protected Serializable getSessionId(ServletRequest request, ServletResponse response){
           String jwtToken = WebUtils.toHttp(request).getHeader(AUTHORIZATION);
           if(StringUtils.isEmpty(jwtToken)){
               //如果没有携带id参数则按照父类的方式在cookie进行获取
               return super.getSessionId(request, response);
           }else{
               //如果请求头中有 authToken 则其值为jwtToken，然后解析出会话id sessionId
   					                                                  request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE,REFERENCED_SESSION_ID_SOURCE);
               Claims decode = JwtTokenUtil.decodeToken(jwtToken);
               //获取我们创建jwt时填入的sessionId
               String id = (String) decode.get("jti");
               log.info(id); request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID,id);
               request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID,Boolean.TRUE);
               return id;
           }
       }
   
   }
   ```

   **注意：我们这里获取到了sessionId后 Shiro框架会将SessionId传入SessionDAO中去获取Session 再通过Session中存储的用户登录权限信息进行鉴权**。

4. 自定义重写三类Shiro过滤器

   之所以要重写是为了更好的适配前后端分离的项目，因为Shiro默认过滤器没有通过是会返回一个页面的，但是前后端分离的项目不需要你去返回页面了，所以我们需要重写我们要使用的过滤器使其返回前后端规定好的对象返回。

   重写`authc`过滤器

   ```JAVA
   public class JwtAuthcFilter extends FormAuthenticationFilter {
   
   
       /**
        * @Description 是否允许访问
        */
       @Override
       protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
           //判断当前请求头中是否带有jwtToken的字符串
           String jwtToken = WebUtils.toHttp(request).getHeader("token");
           //如果有：走jwt校验
           if (!StringUtils.isEmpty(jwtToken)){
               Claims claims = JwtTokenUtil.decodeToken(jwtToken);
               if (!claims.isEmpty()){
                   return super.isAccessAllowed(request, response, mappedValue);
               }else {
                   return false;
               }
           }
           //没有携带token：走原始校验
           return super.isAccessAllowed(request, response, mappedValue);
       }
   
       /**
        * @Description 访问拒绝时调用
        */
       @Override
       protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
           //判断当前请求头中是否带有jwtToken的字符串
           String jwtToken = WebUtils.toHttp(request).getHeader("token");
           //如果有：返回json的应答
           if (!StringUtils.isEmpty(jwtToken)){
               Result<String> result = ResultUtil.fail("没有登录");
               response.setCharacterEncoding("UTF-8");
               response.setContentType("application/json; charset=utf-8");
               response.getWriter().write(JSONObject.toJSONString(result));
               return false;
           }
           //如果没有：走原始方式
           return super.onAccessDenied(request, response);
       }
   }
   ```

   重写`perms`过滤器

   ```JAVA
   public class JwtPermsFilter extends PermissionsAuthorizationFilter {
   
       /**
        * @Description 访问拒绝时调用
        */
       @Override
       protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws IOException {
           //判断当前请求头中是否带有jwtToken的字符串
           String jwtToken = WebUtils.toHttp(request).getHeader("token");
           //如果有：返回json的应答
           if (!StringUtils.isEmpty(jwtToken)){
               Result<String> result = ResultUtil.fail("没有权限");
               response.setCharacterEncoding("UTF-8");
               response.setContentType("application/json; charset=utf-8");
               response.getWriter().write(JSONObject.toJSONString(result));
               return false;
           }
           //如果没有：走原始方式
           return super.onAccessDenied(request, response);
       }
   }
   ```

   重写`roles`过滤器

   ```JAVA
   public class JwtRolesFilter extends RolesAuthorizationFilter {
   
       /**
        * @Description 访问拒绝时调用
        */
       @Override
       protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws IOException {
           //判断当前请求头中是否带有jwtToken的字符串
           String jwtToken = WebUtils.toHttp(request).getHeader("token");
           //如果有：返回json的应答
           if (!StringUtils.isEmpty(jwtToken)){
               Result<String> result = ResultUtil.fail("没有角色");
               response.setCharacterEncoding("UTF-8");
               response.setContentType("application/json; charset=utf-8");
               response.getWriter().write(JSONObject.toJSONString(result));
               return false;
           }
           //如果没有：走原始方式
           return super.onAccessDenied(request, response);
       }
   }
   ```

5. 将新增配置添加进ShiroConfig配置类

   ```JAVA
   /**
    * @Description 会话管理器
    * 将刚刚创建好的会话管理器添加进配置类中
    */
   @Bean(name="sessionManager")
   public ShiroSessionManager shiroSessionManager(RedisSessionDAO redisSessionDAO){
       ShiroSessionManager sessionManager = new ShiroSessionManager();
       sessionManager.setSessionDAO(redisSessionDAO);
       sessionManager.setSessionValidationSchedulerEnabled(false);
       sessionManager.setSessionIdCookieEnabled(true);
       sessionManager.setSessionIdCookie(simpleCookie());
       //设置超时
       sessionManager.setGlobalSessionTimeout(3600*10000);
       return sessionManager;
   }
   /**
    * @Description 创建cookie对象
    */
   @Bean(name="sessionIdCookie")
   public SimpleCookie simpleCookie(){
      SimpleCookie simpleCookie = new SimpleCookie();
      simpleCookie.setName("ShiroSession");
      return simpleCookie;
   }
   /**
    * @Description 自定义过滤器定义
    * 将我们重写三个过滤器添加进来 为了后面添加进过滤器工厂做准备
    */
   private Map<String, Filter> filters() {
       Map<String, Filter> map = new HashMap<String, Filter>();
       map.put("jwt-authc", new JwtAuthcFilter());
       map.put("jwt-perms", new JwtPermsFilter());
       map.put("jwt-roles", new JwtRolesFilter());
       return map;
   }
   
   /**
    * 将自定义的会话管理器添加进securityManager中
    *
   */
   @Bean(name = "securityManager")
   public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm, RedisSessionDAO redisSessionDAO) {
       DefaultWebSecurityManager securityManager = new
               DefaultWebSecurityManager();
       //关联Realm
       securityManager.setRememberMeManager(null);
       securityManager.setSessionManager(shiroSessionManager(redisSessionDAO)); //这里添加自定义会话管理器
       securityManager.setRealm(userRealm);
       return securityManager;
   }
   
   
   //将刚刚三个自定义的过滤器添加进过滤器工厂中。
       @Bean("shiroFilterFactoryBean")
       public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager securityManager) {
           ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
           //设置安全管理器
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           shiroFilterFactoryBean.setFilters(filters()); //将刚刚三个自定义的过滤器添加进过滤器工厂中。
           
           ......
   
           return shiroFilterFactoryBean;
       }
   ```

   如果要使用刚刚三个自定义的过滤器进行过滤器拦截的话 就需要使用那个三个自定义的关键词来配置过滤。

6. `LoginService`中的`Login`方法

   ```JAVA
   @Override
   public Result loginForJwt(LoginDTO loginDTO) {
           Map<String, String> map = new HashMap<>();
           String jwtToken = null;
           try {
               UsernamePasswordToken token = new UsernamePasswordToken(loginDTO.getUsername(), loginDTO.getPassword());
               Subject subject = SecurityUtils.getSubject();
               subject.login(token);
               String shiroSessionId = (String) subject.getSession().getId();
               //登录后颁发的令牌
               LambdaQueryWrapper<User> userLambdaQueryWrapper = new LambdaQueryWrapper<>();
               userLambdaQueryWrapper.eq(User::getUsername,loginDTO.getUsername());
               User shiroUser = userMapper.selectOne(userLambdaQueryWrapper);
               jwtToken = JwtTokenUtil.IssuedToken("system", subject.getSession().getTimeout(),shiroSessionId,shiroUser.getId());
               map.put("jwtToken",jwtToken );
               log.info("jwtToken:{}",map.toString());
               //创建缓存
   //            this.loadAuthorityToCache();
           } catch (Exception ex) {
               return ResultUtil.fail("登录失败");
           }
   
           return ResultUtil.succeed(map);
       }
   ```

   主旨就是使用Shiro的login方法进行登录，登录如果没有抛出异常则说明登录成功，登录成功后将sessionId封装成JWS返回到前端，

   后续的前端通过在请求头部添加token的方式将token发送到后端，后端通过自定义的会话管理器（`ShiroSessionManager`）的`getSessionId`的方法解码`token`获得其中的`sessionId`， 再由后续的`SessionDAO`通过`SessionId`获得`Session`进行登录鉴权。

------

## 注意

### 盐序列化

当我们的密码有加密盐的时候若想使用`Redis`作为`Realm`的缓存,需要重写实现`ByteSource`接口，因为`Redis`存储信息的时候需要对信息进行序列化后才能进行存储，但是`ByteSource`的实现类`SimpleByteSource`并没有实现序列化(即没有实现`Serializable`接口)，同时由于`SimpleByteSource`没有无参构造导致无法反序列化。

重写后的`MyByteSource`

```JAVA
/**
 * @author ： Donald
 * @date ： 2020/10/18 17:41
 * @description： 自定义salt的实现 主要巍峨salt的序列化
 * 采用redis缓存shiro的认证信息，并且要对这些信息进行序列化后再存储，但是序列化的时候，SimpleByteSource类没有实现Serializable接口，导致序列化失败
 * SimpleByteSource没有默认构造方法，导致反序列化的时候失败
 *
 */
public class MyByteSource implements ByteSource, Serializable {



    /*这里将final去掉了,去掉后要在后面用getter和setter赋、取值*/
    private byte[] bytes;
    private String cachedHex;
    private String cachedBase64;

    /*添加了一个无参构造方法*/
    public MyByteSource(){}

    public MyByteSource(byte[] bytes) {
        this.bytes = bytes;
    }

    public MyByteSource(char[] chars) {
        this.bytes = CodecSupport.toBytes(chars);
    }

    public MyByteSource(String string) {
        this.bytes = CodecSupport.toBytes(string);
    }

    public MyByteSource(ByteSource source) {
        this.bytes = source.getBytes();
    }

    public MyByteSource(File file) {
        this.bytes = (new MyByteSource.BytesHelper()).getBytes(file);
    }

    public MyByteSource(InputStream stream) {
        this.bytes = (new MyByteSource.BytesHelper()).getBytes(stream);
    }

    public static boolean isCompatible(Object o) {
        return o instanceof byte[] || o instanceof char[] || o instanceof String || o instanceof ByteSource || o instanceof File || o instanceof InputStream;
    }


    /*这里加了getter和setter*/
    public void setBytes(byte[] bytes) {
        this.bytes = bytes;
    }

    public byte[] getBytes() {
        return this.bytes;
    }

    public boolean isEmpty() {
        return this.bytes == null || this.bytes.length == 0;
    }

    public String toHex() {
        if (this.cachedHex == null) {
            this.cachedHex = Hex.encodeToString(this.getBytes());
        }

        return this.cachedHex;
    }

    public String toBase64() {
        if (this.cachedBase64 == null) {
            this.cachedBase64 = Base64.encodeToString(this.getBytes());
        }

        return this.cachedBase64;
    }

    public String toString() {
        return this.toBase64();
    }

    public int hashCode() {
        return this.bytes != null && this.bytes.length != 0 ? Arrays.hashCode(this.bytes) : 0;
    }

    public boolean equals(Object o) {
        if (o == this) {
            return true;
        } else if (o instanceof ByteSource) {
            ByteSource bs = (ByteSource)o;
            return Arrays.equals(this.getBytes(), bs.getBytes());
        } else {
            return false;
        }
    }

    private static final class BytesHelper extends CodecSupport {
        private BytesHelper() {
        }

        public byte[] getBytes(File file) {
            return this.toBytes(file);
        }

        public byte[] getBytes(InputStream stream) {
            return this.toBytes(stream);
        }
    }

    /*取代原先加盐的工具类*/
    public static class Util{
        public static ByteSource bytes(byte[] bytes){
            return new MyByteSource(bytes);
        }

        public static ByteSource bytes(String arg0){
            return new MyByteSource(arg0);
        }
    }
}
```

自定义`UserRealm`

```JAVA
return new SimpleAuthenticationInfo(username, user.getPassword(), MyByteSource.Util.bytes(user.getSalt()), this.getName());
```

### 过滤器

`AccessControlFilter`过滤器中有两个方法 分别为`isAccessAllowed`和`onAccessDenied`

> isAccessAllowed：表示是否允许访问；mappedValue就是[urls]配置中拦截器参数部分，如果允许访问返回true，否则false；
>
> onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回true表示需要继续处理；如果返回false表示该拦截器实例已经处理了，将直接返回即可。

------

## 分布式网关

**未完待续。。。**