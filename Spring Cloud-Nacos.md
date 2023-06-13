# Spring Cloud-Nacos

## 认识Nacos

`Nacos` 是阿里巴巴的产品，现在是 `SpringCloud` 中的一个组件。相比 `Eureka` 功能更加丰富，在国内受欢迎程度较高。

[![image-20220812144921010](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121449674.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121449674.png)

`Nacos /nɑ:kəʊs/` 是 Dynamic Naming and Configuration Service的首字母简称，**一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。**

服务（Service）是 Nacos 世界的一等公民。Nacos 支持几乎所有猪流类型的“服务”的发现、配置、管理：

- [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [gRPC](https://grpc.io/docs/guides/concepts.html#service-definition) & [Dubbo RPC Service](https://dubbo.incubator.apache.org/)
- [Spring Cloud RESTful Service](https://spring.io/projects/spring-restdocs)

**Nacos 的关键特性包括:**

- **服务发现和服务健康监测**

  > **Nacos 支持基于 DNS 和基于 RPC 的服务发现。**服务提供者使用 [原生SDK](https://nacos.io/zh-cn/docs/sdk.html)、[OpenAPI](https://nacos.io/zh-cn/docs/open-api.html)、或一个[独立的Agent TODO](https://nacos.io/zh-cn/docs/other-language.html)注册 Service 后，服务消费者可以使用[DNS TODO](https://nacos.io/zh-cn/docs/xx) 或[HTTP&API](https://nacos.io/zh-cn/docs/open-api.html)查找和发现服务。
  >
  > **Nacos 提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求**。Nacos 支持传输层 (PING 或 TCP)和应用层 (如 HTTP、MySQL、用户自定义）的健康检查。 对于复杂的云环境和网络拓扑环境中（如 VPC、边缘网络等）服务的健康检查，Nacos 提供了 agent 上报模式和服务端主动检测2种健康检查模式。Nacos 还提供了统一的健康检查仪表盘，帮助您根据健康状态管理服务的可用性及流量。

- **动态配置服务**

  > **动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置。**
  >
  > 动态配置消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效和敏捷。
  >
  > 配置中心化管理让实现无状态服务变得更简单，让服务按需弹性扩展变得更容易。
  >
  > Nacos 提供了一个简洁易用的UI ([控制台样例 Demo](http://console.nacos.io/nacos/index.html)) 帮助您管理所有的服务和应用的配置。Nacos 还提供包括配置版本跟踪、金丝雀发布、一键回滚配置以及客户端配置更新状态跟踪在内的一系列开箱即用的配置管理特性，帮助您更安全地在生产环境中管理配置变更和降低配置变更带来的风险。

- **动态 DNS 服务**

  > **动态 DNS 服务支持权重路由，让您更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单DNS解析服务。**动态DNS服务还能让您更容易地实现以 DNS 协议为基础的服务发现，以帮助您消除耦合到厂商私有服务发现 API 上的风险。
  >
  > Nacos 提供了一些简单的 [DNS APIs TODO](https://nacos.io/zh-cn/docs/xx) 帮助您管理服务的关联域名和可用的 IP:PORT 列表.

- **服务及其元数据管理**

  > **Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的 SLA 以及最首要的 metrics 统计数据。**

> Nacos官方文档：[home (nacos.io)](https://nacos.io/zh-cn/)

## Nacos 地图

一图看懂 Nacos

[![nacosMap](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121813662.jpg)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121813662.jpg)

## Nacos生态

[![nacos_landscape.png](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121813862.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121813862.png)

如 Nacos 全景图所示，Nacos 无缝支持一些主流的开源生态，例如

- [Spring Cloud](https://nacos.io/en-us/docs/quick-start-spring-cloud.html)
- [Apache Dubbo and Dubbo Mesh](https://nacos.io/zh-cn/docs/use-nacos-with-dubbo.html)
- [Kubernetes and CNCF](https://nacos.io/zh-cn/docs/use-nacos-with-kubernetes.html)。

使用 Nacos 简化服务发现、配置管理、服务治理及管理的解决方案，让微服务的发现、管理、共享、组合更加容易。

## Nacos安装-单机

### 前置环境

有一个能够运行 **docker** 和 **mysql**，可以参考[Docker中运行一个mysql](https://developer.aliyun.com/article/871775?spm=a2c6h.12873639.article-detail.6.b10d6b470PITWC)

### 选择拉取镜像

> https://hub.docker.com/r/nacos/nacos-server/tags

这里选择了**`2.1.0`**版本

[![image.png](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121825334.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121825334.png)

```
SH
docker pull nacos/nacos-server:v2.1.0
```

### 创建nacos数据库

> 将nacos持久化到mysql数据库中
>
> 新建nacos数据库

从[Nacos数据库配置](https://github.com/alibaba/nacos/blob/develop/distribution/conf/nacos-mysql.sql)下载建表语句。也可以将下列语句粘贴执行

```
SQL
/*
 * Copyright 1999-2018 Alibaba Group Holding Ltd.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(20) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(50) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

### 运行容器

#### 获取容器内的配置文件

先直接部署一个容器

> 是为了拿到`application.properties`等配置文件

```
SH
docker run -d \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=localhost \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=root \
-e MYSQL_SERVICE_DB_NAME=nacos \
-p 8848:8848 \
--name nacos \
--restart=always \
nacos/nacos-server:v2.1.0 
```

> 参数说明
>
> - MODE=standalone 单节点模式（开发阶段单机模式即可）
> - SPRING_DATASOURCE_PLATFORM=mysql 使用mysql数据库连接方式
> - MYSQL_SERVICE_HOST=192.168.120.1 指定数据库地址
> - MYSQL_SERVICE_PORT 数据库端口
> - MYSQL_SERVICE_USER 数据库用户名
> - MYSQL_SERVICE_PASSWORD 数据库密码
> - MYSQL_SERVICE_DB_NAME 数据库名称
> - -p 8848:8848 端口映射
> - –name nacos 容器命名
> - –restart=always 任意时候重启容器，开机就能自动启动容器(需设置docker为开机自启)

**Ncaos Docker支持的参数有：**

**Common property configuration**

| 属性名称                                | 描述                                               | 选项                                                         |
| --------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| MODE                                    | 系统启动方式: 集群/单机                            | cluster/standalone默认 **cluster**                           |
| NACOS_SERVERS                           | 集群地址                                           | p1:port1空格ip2:port2 空格ip3:port3                          |
| PREFER_HOST_MODE                        | 支持IP还是域名模式                                 | hostname/ip 默认 **ip**                                      |
| NACOS_SERVER_PORT                       | Nacos 运行端口                                     | 默认 **8848**                                                |
| NACOS_SERVER_IP                         | 多网卡模式下可以指定IP                             |                                                              |
| SPRING_DATASOURCE_PLATFORM              | 单机模式下支持MYSQL数据库                          | mysql / 空 默认:空                                           |
| MYSQL_SERVICE_HOST                      | 数据库 连接地址                                    |                                                              |
| MYSQL_SERVICE_PORT                      | 数据库端口                                         | 默认 : **3306**                                              |
| MYSQL_SERVICE_DB_NAME                   | 数据库库名                                         |                                                              |
| MYSQL_SERVICE_USER                      | 数据库用户名                                       |                                                              |
| MYSQL_SERVICE_PASSWORD                  | 数据库用户密码                                     |                                                              |
| MYSQL_SERVICE_DB_PARAM                  | 数据库连接参数                                     | default : **characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false** |
| MYSQL_DATABASE_NUM                      | 数据库编号                                         | 默认 :**1**                                                  |
| JVM_XMS                                 | -Xms                                               | 默认 :1g                                                     |
| JVM_XMX                                 | -Xmx                                               | 默认 :1g                                                     |
| JVM_XMN                                 | -Xmn                                               | 默认 :512m                                                   |
| JVM_MS                                  | -XX:MetaspaceSize                                  | 默认 :128m                                                   |
| JVM_MMS                                 | -XX:MaxMetaspaceSize                               | 默认 :320m                                                   |
| NACOS_DEBUG                             | 是否开启远程DEBUG                                  | y/n 默认 :n                                                  |
| TOMCAT_ACCESSLOG_ENABLED                | server.tomcat.accesslog.enabled                    | 默认 :false                                                  |
| NACOS_AUTH_SYSTEM_TYPE                  | 权限系统类型选择,目前只支持nacos类型               | 默认 :nacos                                                  |
| NACOS_AUTH_ENABLE                       | 是否开启权限系统                                   | 默认 :false                                                  |
| NACOS_AUTH_TOKEN_EXPIRE_SECONDS         | token 失效时间                                     | 默认 :18000                                                  |
| NACOS_AUTH_TOKEN                        | token                                              | 默认 :SecretKey012345678901234567890123456789012345678901234567890123456789 |
| NACOS_AUTH_CACHE_ENABLE                 | 权限缓存开关 ,开启后权限缓存的更新默认有15秒的延迟 | 默认 : false                                                 |
| MEMBER_LIST                             | 通过环境变量的方式设置集群地址                     | 例子:192.168.16.101:8847?raft_port=8807,192.168.16.101?raft_port=8808,192.168.16.101:8849?raft_port=8809 |
| EMBEDDED_STORAGE                        | 是否开启集群嵌入式存储模式                         | `embedded` 默认 : none                                       |
| NACOS_AUTH_CACHE_ENABLE                 | nacos.core.auth.caching.enabled                    | default : false                                              |
| NACOS_AUTH_USER_AGENT_AUTH_WHITE_ENABLE | nacos.core.auth.enable.userAgentAuthWhite          | default : false                                              |
| NACOS_AUTH_IDENTITY_KEY                 | nacos.core.auth.server.identity.key                | default : serverIdentity                                     |
| NACOS_AUTH_IDENTITY_VALUE               | nacos.core.auth.server.identity.value              | default : security                                           |
| NACOS_SECURITY_IGNORE_URLS              | nacos.security.ignore.urls                         | default : `/,/error,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/p` |

### 宿主机配置文件映射

##### 拷贝配置文件

```
SH
docker cp nacos:/home/nacos/conf/application.properties /home/docker/nacos/config/
```

##### 拷贝logback日志配置文件

```
SH
docker cp nacos:/home/nacos/conf/nacos-logback.xml /home/docker/nacos/config/
```

##### 修改application.properties的配置

```
PROPERTIES
# spring
server.servlet.contextPath=${SERVER_SERVLET_CONTEXTPATH:/nacos}
server.contextPath=/nacos
server.port=${NACOS_APPLICATION_PORT:8848}

# 修改此行,将SPRING_DATASOURCE_PLATFORM的默认值""改为mysql
spring.datasource.platform=${SPRING_DATASOURCE_PLATFORM:mysql}
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
db.num=${MYSQL_DATABASE_NUM:1}

# 修改此行,添加MYSQL_SERVICE_HOST的默认值为192.168.120.1,MYSQL_SERVICE_DB_NAME的默认值为nacos
db.url.0=jdbc:mysql://${MYSQL_SERVICE_HOST:124.222.35.20}:${MYSQL_SERVICE_PORT:3319}/${MYSQL_SERVICE_DB_NAME:nacos_config}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}

# 修改此行,添加MYSQL_SERVICE_HOST的默认值为192.168.120.1,MYSQL_SERVICE_DB_NAME的默认值为nacos
db.url.1=jdbc:mysql://${MYSQL_SERVICE_HOST:124.222.35.20}:${MYSQL_SERVICE_PORT:3319}/${MYSQL_SERVICE_DB_NAME:nacos_config}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}

# 修改此行,添加MYSQL_SERVICE_USER的默认值为root
db.user=${MYSQL_SERVICE_USER:ding}

# 修改此行,添加MYSQL_SERVICE_PASSWORD的默认值为root
db.password=${MYSQL_SERVICE_PASSWORD:dyw20020304}
### The auth system to use, currently only 'nacos' is supported:
nacos.core.auth.system.type=${NACOS_AUTH_SYSTEM_TYPE:nacos}


### The token expiration in seconds:
nacos.core.auth.default.token.expire.seconds=${NACOS_AUTH_TOKEN_EXPIRE_SECONDS:18000}

### The default token:
nacos.core.auth.default.token.secret.key=${NACOS_AUTH_TOKEN:SecretKey012345678901234567890123456789012345678901234567890123456789}

### Turn on/off caching of auth information. By turning on this switch, the update of auth information would have a 15 seconds delay.
nacos.core.auth.caching.enabled=${NACOS_AUTH_CACHE_ENABLE:false}
nacos.core.auth.enable.userAgentAuthWhite=${NACOS_AUTH_USER_AGENT_AUTH_WHITE_ENABLE:false}
nacos.core.auth.server.identity.key=${NACOS_AUTH_IDENTITY_KEY:serverIdentity}
nacos.core.auth.server.identity.value=${NACOS_AUTH_IDENTITY_VALUE:security}
server.tomcat.accesslog.enabled=${TOMCAT_ACCESSLOG_ENABLED:false}
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D
# default current work dir
server.tomcat.basedir=
## spring security config
### turn off security
nacos.security.ignore.urls=${NACOS_SECURITY_IGNORE_URLS:/,/error,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/**,/v1/console/health/**,/actuator/**,/v1/console/server/**}
```

采用添加默认值的方式，这样不会影响指定命令行的参数

#### 运行容器

运行之前先删除之前启动的容器

```
SH
docker stop nacos
docker rm nacos
```

重新运行容器

```
SH
docker run -d \
-e MODE=standalone \
-p 8848:8848 \
-v /home/docker/nacos/config:/home/nacos/conf \
-v /home/docker/nacos/logs:/home/nacos/logs \
-v /home/docker/nacos/data:/home/nacos/data \
--name nacos \
--restart=always \
nacos/nacos-server:v2.1.0
```

访问：**`ip:8848/nacos`** 进入 `Nacos` 图形化控制台

输入账户密码 默认账户密码都为 nacos

[![image-20220812194232088](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121942195.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121942195.png)

进入如下界面表示登录成功。

## 服务注册/服务发现

### 入门案例

将服务注册到Nacos上，完成服务调用。

由于 Spring Cloud Commons 定义了Spring Cloud 的一组抽象类和接口，所以Spring Cloud 各类组件的行为都是一致的，例如Spring Cloud 规定了 服务发现接口和 服务注册的接口，Spring Cloud 中的服务治理组件都是实现了这两个接口并衍生的，所以相同类别的组件切换只需要更换依赖和修改配置即可。其他操作与原来一致（例如服务的注册，服务的发现，服务的调用等）

例如这里我们使用 nacos 代替 eureka 作为注册中心，只需要修改依赖和添加nacos的服务器地址即可，其他行为与原eureka相同，不需要做其他的改变。

> **Spring Cloud 2021.0.1** 新版本使用 **Spring Cloud Loadbalancer** 做负载均衡，没有默认集成 **Ribbon** 了，在进行服务消费者开发的项目中需要引入 **Loadbalancer** 依赖，这一点需要注意一下。从2021版本开始 Nacos已经不再支持Ribbon了。所以推荐使用Spring Cloud LoadBalaner

#### 父工程

在父工程中添加 `spring-cloud-alibaba-dependencies`得管理依赖

```
XML
<dependency>    
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

完整pom.xml依赖

```
XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.8</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.devildyw</groupId>
    <artifactId>Cloud-10-Nacos</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Cloud-10-Nacos</name>
    <description>Cloud-10-Nacos</description>

    <packaging>pom</packaging>
    <modules>
        <module>Cloud-nacos-consumer-order80</module>
        <module>Cloud-nacos-provider-payment8001</module>
        <module>Cloud-nacos-provider-payment8002</module>
    </modules>


    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>4.12</junit.version>
        <lombok.version>1.18.10</lombok.version>
        <log4j.version>1.2.17</log4j.version>
        <mysql.version>8.0.28</mysql.version>
        <druid.version>1.2.11</druid.version>
        <mybatis.spring.boot.version>2.1.1</mybatis.spring.boot.version>
        <mybatis-plus>3.5.2</mybatis-plus>
    </properties>

    <!--子模块继承之后，提供作用：锁定版本+子module不用谢groupId和version-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-project-info-reports-plugin</artifactId>
                <version>3.2.2</version>
            </dependency>
            <!--spring cloud Hoxton.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2021.0.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud 阿里巴巴-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2021.1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--mysql-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
                <scope>runtime</scope>
            </dependency>
            <!-- druid-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <!--mybatis-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.version}</version>
            </dependency>
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus}</version>
            </dependency>
            <!--junit-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>2.6.8</version>
            </dependency>
            <!--log4j-->
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
        </dependencies>

    </dependencyManagement>

    <build>
        <finalName>SpringCloud-Hello-01</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

------

#### 消费者

将`Cloud-eureka-consumer-order80`工程复制到该父工程的子模块下，改名为`Cloud-nacos-consumer-order80`

新增pom.xml依赖，并且把原来的eureka的依赖删除或者注释掉。

```
XML
<!-- nacos客户端依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

完整`pom.xml`依赖

```
XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>top.devildyw</groupId>
        <artifactId>Cloud-10-Nacos</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>Cloud-nacos-consumer-order80</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
<!--        <dependency>-->
<!--            <groupId>org.springframework.cloud</groupId>-->
<!--            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>-->
<!--        </dependency>-->

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>com.dyw</groupId>
            <artifactId>Cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

删除掉原来与`eureka`有关的所有配置，`application.yml`和代码中的部分

`application.yml`新增nacos配置

```
YML
cloud:
  nacos:
    server-addr: ip:8848 # 注册中心地址
    username: nacos # nacos用户名
    password: nacos # nacos密码
```

------

#### 生产者

将`Cloud-eureka-provider-payment8001` 和 `Cloud-eureka-provider-payment8002`工程复制到该父工程的子模块下，改名为`Cloud-nacos-provider-payment8001` 和 `Cloud-nacos-provider-payment8002`

新增pom.xml依赖，并且把原来的eureka的依赖删除或者注释掉。

```
XML
<!-- nacos客户端依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

完整`pom.xml`依赖

```
XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>top.devildyw</groupId>
        <artifactId>Cloud-10-Nacos</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>Cloud-nacos-provider-payment8002</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
<!--        <dependency>-->
<!--            <groupId>org.springframework.cloud</groupId>-->
<!--            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>-->
<!--        </dependency>-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.dyw</groupId>
            <artifactId>Cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

删除掉原来与`eureka`有关的所有配置，`application.yml`和代码中的部分

`application.yml`新增nacos配置

```
YML
cloud:
  nacos:
    server-addr: ip:8848 # 注册中心地址
    username: nacos # nacos用户名
    password: nacos # nacos密码
```

#### 测试

启动生产者集群和消费者。

启动完成后可以在Nacos的控制界面中看到，这跟eureka也是一致的。

[![image-20220812231435085](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208122314253.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208122314253.png)

点击详情可以看到服务实例节点的ip、端口等信息。

[![image-20220812231544441](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208122315536.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208122315536.png)

调用`get: http://localhost:80/consumer/payment/get/1547503738317369346`

[![image-20220812232655486](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208122326542.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208122326542.png)

调用成功。

## Nacos服务分级存储模型

[![image-20220813111902398](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131119550.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131119550.png)

一个服务可以包含多个服务集群，集群又可以包含多个服务实例。这就是Nacos的分级存储模型。

**服务跨集群调用问题**

服务调用尽可能选择本地集群的服务，，跨集群调用延迟较高

本地集群不可访问时，再去访问其他集群。

[![image-20220813112131302](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131121375.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131121375.png)

没有设置集群是，会显示DEFAULT

[![image-20220813112654815](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131126861.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131126861.png)

我们可以通过修改`application.yml`文件来修改配置集群名称

```
YML
cloud:
  nacos:
    server-addr: ip:8848 # 注册中心地址
    username: nacos # nacos用户名
    password: nacos # nacos密码
    discovery:
      cluster-name: CD #集群名称
```

修改后

[![image-20220813112952725](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131129777.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131129777.png)

**总结**：

1. Nacos服务分级存储模型
   1. 一级是服务，例如userservice
   2. 二级是集群，例如杭州或上海
   3. 三级是实例，例如杭州机房的某台部署了userservice的服务器
2. 如何设置实例的集群属性
   1. 修改`application.yml`文件，添加集群名称即可。

## NacosRule 负载均衡

Nacos + Ribbon特有的策略

> Spring Cloud Nacos 2021 版本开始已经禁止使用Ribbon做负载均衡了，而使用LoadBalancer有没有许多支持的策略，所以这里我们选择老版本的Spring Cloud Nacos 做演示。

```
XML
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  <version>2.2.4.RELEASE</version>
</dependency>
```

消费者添加如上依赖后，修改`application.yml`配置

```
YML
cloud-payment-service:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则
```

为 `cloud-payment-service` 服务配置Nacos的集群负载均衡策略。

修改后消费者就会优先调用同一个集群内的服务提供者。如果同一集群内的服务提供者宕机或者发生网络波动断开了与注册中心的连接，消费者就会向其他集群内的提供者发起调用，并且会在控制台提醒发生了跨集群调用。

[![image-20220813130918232](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131309334.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131309334.png)

**总结：**

1. NacosRule负载均衡策略
   1. 优先选择同集群服务实例列表
   2. 本地集群找不到提供者，才去其他集群寻找，并且会报警
   3. **确定了可用实例列表后，再采用随机负载均衡挑选实例。**

### 根据权重负载均衡

实际部署中会出现这样的场景：

- 服务器设备性能由差异，部分实例所在机器性能比较好，另一些较差，我没希望性能好的机器承担更多的用户请求。

Nacos提供了权重配置来控制访问频率，权重越大则访问频率越高。

在Nacos控制台可以设置实例的权重值，首先选中实例后面的编辑按钮

[![image-20220813145655114](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131456215.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131456215.png)

将权重设置为0.1，测试可以发现8001被访问到的频率大大降低

[![image-20220813145757129](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131457175.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131457175.png)

**用途**

除了希望将性能更好的机器承担更多的用户请求外，还有一点；当我们想要将某一服务实例做维护时，可以将该服务实例的权重调为0，慢慢的请求将不会再发送到该实例上，我们就可以将其下线进行维护，等维护完毕，想要上线时，将其的权重调小一点，放部分请求测试服务是否可用后，再将其的权重调为正常值。**在此期间用户时完全没有感知的**。

**总结**

1. 实例的权重控制
   1. Nacos控制台可用设置实例的权重值，0~1之间
   2. 统计群内的多个实例，权重越高被访问的频率越高
   3. 权重为0则完全不会被访问

## 环境隔离 - NameSpace

Nacos中服务存储和数据存储的最外层都是一个名为 namespace 的东西，用来做最外层隔离。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131508378.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131508378.png)

用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

Nacos 数据模型 Key 由三元组唯一确认。

- **作为注册中心时，Namespace + Group + Service**
- **作为配置中心时，Namespace + Group + DataId**

在Nacos控制台可用创建 namespace，用来隔离环境

[![image-20220813151321008](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131513130.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131513130.png)

然后填写一个新的命名空间信息：

[![image-20220813151432517](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131514573.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131514573.png)

保存后会在控制台看到这个命名空间的id：

[![image-20220813151506147](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131515237.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131515237.png)

而我们创建的服务注册到 Nacos 上默认都是在 public 这个命名空间下，想要修改就需要在`application.yml` 中添加如下配置。

```
YML
cloud:
  nacos:
    discovery:
      namespace: 6f5658db-97ef-45e7-b48c-d3a8309275a3 #namespace的id
```

配置好后 在dev的命名空间下就可以看到我们配置的服务。

[![image-20220813151841255](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131518316.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131518316.png)

此时访问消费者调用提供者的接口，因为namespace不同，会导致找不到userservice，控制台会报错：

[![image-20220813151944162](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131519203.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131519203.png)

**总结**

1. Nacos环境隔离
   1. 每个namespace都有唯一id
   2. 服务设置namespace时要写id而不是名称
   3. 不同namespace下的服务互相不可见

## 配置分组-Group

Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：**不同的应用或组件使用了相同的配置类型，如 database_url 配置和 MQ_topic 配置**。

## Nacos 与 Eureka 对比

nacos注册中心细节分析

[![image-20220813152243270](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131522369.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208131522369.png)

消费者并不是每一次都向注册中心拉取一次服务列表，而是有一个缓存，nacos客户端会对服务列表做一个服务列表缓存，默认每隔30秒刷新一次。这一点与Eureka一致；**但是当发现有服务变更，Nacos会主动推送变更消息到消费者。**

Nacos对于临时实例采用心跳检测（即服务每个一段时间向 Nacos 发起一次心跳），如果超过一段时间后 Nacos 接收不到服务的心跳了，就会把这个临时实例剔除；

但是对于非临时实例 服务实例不再需要向 Nacos 发送心跳，而是Nacos主动询问节点是否存活。并且服务挂掉了也不会将其直接剔除，而是标记为不健康的实例。

```
YML
cloud:
  nacos:
    discovery:
      ephemeral: false #修改为非临时实例
```

**总结：**

1. Nacos 与 Eureka 的共同点
   1. 都支持服务注册和服务拉取
   2. 都支持服务提供者心跳方式做健康检测
2. Nacos 与 Eureka 的区别
   1. Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式
   2. 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
   3. Nacos 支持服务列表变更的消息推送模式，服务列表更新更及时。
   4. Nacos 集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式。

------

## 配置管理

前面我们学过 Spring Cloud Config。这里我们学习 Nacos 的配置管理，这也是非常流行的方式。

- 配置更改热更新

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132156351.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132156351.png)

Spring Cloud Alibaba Nacos 即有服务注册/发现的功能，又有配置管理的功能。

### 添加配置

在Nacos控制台上添加配置

[![image-20220813215901633](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132159724.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132159724.png)

在弹出表单中填写配置信息：

[![image-20220813215938772](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132159829.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132159829.png)

### 获取配置

配置获取的步骤如下：

[![image-20220813220331379](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132203450.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208132203450.png)

1. 引入Nacos的配置管理客户端依赖

```
XML
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2021.1</version>
</dependency>
```

1. 在resource目录添加一个bootstrap.yml文件，**这个文件是引导文件，优先级高于application.yml**，用来在引导配置管理中心的的信息，来获取配置：

   ```
   YML
   spring:
     application:
       name: cloud-order-service #服务名称
     profiles:
       active: dev #开发环境，这里是dev
     cloud:
       nacos:
         server-addr: 36.137.128.27:8848
         username: nacos
         password: nacos
         config:
           file-extension: yaml #文件名后缀
           enabled: true
           group: DEFAULT_GROUP
       
   
   management:
     endpoints:
       web:
         exposure:
           include:
             - '*'
   ```

   配置后可以将Application.yml中的nacos相关配置给删除，但还是按需配置。

2. 业务类

   ```
   JAVA
   @Value("${pattern.dateformat}")
      private String dateFormat;
      
      @GetMapping("now")
      public String now(){
          return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateFormat));
      }	
   ```

3. 启动后 访问对于接口

   ```
   JSON
   2022-08-14 11:56:48
   ```

**总结：**

将配置交给Nacos管理的步骤

1. 在Nacos中添加配置文件。
2. 在微服务中引入nacos的config依赖。
3. 在微服务中添加bootstrap.yml，配置nacos地址、当前环境、服务名称、文件后缀名。这些决定了程序启动时去nacos读取那个文件。

### 配置自动刷新

Nacos中的配置文件变更后，微服务无需重启就可以感知。**与Spring Cloud Config 不同的是Nacos配置文件发送变动后会主动推送给订阅者。**不过需要通过下面两种配置实现：

**方式一：**

在 @Value 注入的变量所在类上添加注解 @RefreshScope

```
JAVA
@Slf4j
@RestController
@RefreshScope
@RequestMapping("consumer")
public class OrderController {
    @Value("${pattern.dateformat}")
    private String dateFormat;

    @GetMapping("now")
    public String now(){
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateFormat));
    } 
}
```

**方式二：**

使用@ConfigurationProperties注解

```
JAVA
@Component
@Data
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
}
JAVA
@Resource
PatternProperties properties;

@GetMapping("now")
public String now(){
    return LocalDateTime.now().format(DateTimeFormatter.ofPattern(properties.getDateformat()));
}
```

**总结：**

Nacos配置更改后，微服务可以实现热更新，方式：

1. 通过@Value注解注入，结合@RefreshScope来刷新
2. 通过@ConfigurationProperties注入，自动刷新

注意事项：

- 不是所有的配置都适合放到配置中心，维护起来比较麻烦
- 建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置

### 多环境配置共享

微服务启动时会从nacos读取多个配置文件：

- [spring.application.name]-[spring.profiles.active].yaml，例如：userservice-dev.yaml
- [spring.application.name].yaml，例如：userservice.yaml无论profile

如何变化，[spring.application.name].yaml这个文件一定会加载，因此多环境共享配置可以写入这个文件

[![image-20220814133233859](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141332991.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141332991.png)

新增配置文件`cloud-order-service.yaml`

[![image-20220814133314239](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141333328.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141333328.png)

**测试**

配置类

```
JAVA
@Component
@Data
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;

    private String envSharedValue;
}
```

业务类

```
JAVA
@Resource
PatternProperties properties;


@GetMapping("prop")
public PatternProperties properties(){
    return properties;
}
```

结果

```
JSON
{
    "dateformat": "yyyy-MM-dd  HH:mm:ss",
    "envSharedValue": "环境共享属性值"
}
```

发现确实访问到了未带有属性的配置文件。

**多环境配置覆盖优先级**

**服务名-profile.yaml > 服务名称.yaml > 本地配置**

[![image-20220814134350338](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141343402.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141343402.png)

**总结：**

微服务会从nacos读取的配置文件：

1. [服务名]-[spring.profile.active].yaml，环境配置
2. [服务名].yaml，默认配置，多环境共享

优先级：

- [服务名]-[环境].yaml >[服务名].yaml > 本地配置

### 多服务共享配置

不同微服务之间可以共享配置文件，通过下面的两种方式来指定：

首先创建一个用来共享的配置文件。**即除了配置中心独有特质的配置文件外，还想配置中心拉取以创建的共享的配置文件**

[![image-20220814135415706](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141354786.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141354786.png)

**方式一：**

```
YML
spring:
  application:
    name: cloud-order-service #服务名称
  profiles:
    active: dev #开发环境，这里是dev
  cloud:
    nacos:
      server-addr: 36.137.128.27:8848
      username: nacos
      password: nacos
      config:
        file-extension: yaml #文件名后缀
        enabled: true
        group: DEFAULT_GROUP
        # 多微服务共享配置文件 配置
        shared-configs: # 多微服务间共享的配置列表
          - data-id: common.yaml # 要共享的配置文件id
```

**方式二：**

```
YML
spring:
  application:
    name: cloud-order-service #服务名称
  profiles:
    active: dev #开发环境，这里是dev
  cloud:
    nacos:
      server-addr: 36.137.128.27:8848
      username: nacos
      password: nacos
      config:
        file-extension: yaml #文件名后缀
        enabled: true
        group: DEFAULT_GROUP
        # 多微服务共享配置文件 配置
        extension-configs: # 多微服务间共享的配置列表
          - data-id: common.yaml # 要共享的配置文件id
```

测试：

```
JSON
{
    "dateformat": "yyyy-MM-dd HH:mm:ss",
    "envSharedValue": "环境共享属性值",
    "name": "共享配置文件"
}
```

**多种配置的覆盖优先级：**

**服务名-profile.yaml >服务名称.yaml > extension-config > shared-config > 本地配置**

[![image-20220814135520212](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141355287.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208141355287.png)

**总结**

微服务默认读取的配置文件：

1. [服务名]-[spring.profile.active].yaml，默认配置
2. [服务名].yaml，多环境共享

不同微服务共享的配置文件：

1. 通过shared-configs指定
2. 通过extension-configs指定

优先级：

- 环境配置 >服务名.yaml > extension-config > extension-configs > shared-configs > 本地配置

> 实践：
>
> 项目中的使用：每个微服务创建自己的命名空间，使用配置分组区分环境，dev、test、prod
>
> 同时加载多个配置集
>
> 1. 微服务任何配置信息，任何配置文件都可以放在配置中心中
> 2. 只需要在 bootstrap.yml 中说明加载配置中心那些配置文件即可

## Nacos安装-集群

docker network create nacos-net

## Open-API

> Nacos open-API官网地址：https://nacos.io/zh-cn/docs/open-api.html
>
> 包含了查询服务实例、注销服务实例等功能。