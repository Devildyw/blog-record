# Gitlab持续集成/持续部署

## Java

1. `docker`环境下直接执行这条命令，安装 `Runner`

```BASH
sudo docker run -d --name gitlab-runner --restart always \
-v /home/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
    2. 注册服务 `url `和 `token `在`gitlab`左边的设置展开`runner`即可看到
BASH
docker exec -it gitlab-runner  gitlab-runner register -n \
  --url https://git.pyhub.club/ \
  --registration-token gsh33g31h2Q1E3szh4Jd \
  --description "docker deploy" \
  --docker-privileged=true \
  --docker-pull-policy="if-not-present" \
  --docker-image "docker:latest" \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
  --docker-volumes /root/m2:/root/.m2 \
  --executor docker
```

1. 写配置文件` .gitlab-ci.yml`

```YML
before_script:
  - rm -rf /root/.m2/settings.xml
  - echo -e "<?xml version=\""1.0\"" encoding=\""UTF-8\""?><settings xmlns=\""http://maven.apache.org/SETTINGS/1.0.0\"" xmlns:xsi=\""http://www.w3.org/2001/XMLSchema-instance\"" xsi:schemaLocation=\""http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd\""><mirrors><mirror><id>mirror</id><name>mirror</name><url>https://maven.aliyun.com/nexus/content/groups/public</url><mirrorOf>central,jcenter,!rdc-releases,!rdc-snapshots</mirrorOf></mirror> </mirrors>     <servers>
        <server>
            <id>rdc-releases</id>
            <username>617d503fbc6f250a94c5d6ec</username>
            <password>VDzlsL5jYZot</password>
        </server>
        <server>
            <id>rdc-snapshots</id>
            <username>617d503fbc6f250a94c5d6ec</username>
            <password>VDzlsL5jYZot</password>
        </server>
    </servers>
    <profiles>
        <profile>
            <id>rdc</id>
            <properties>
                <altReleaseDeploymentRepository>
                    rdc-releases::default::https://packages.aliyun.com/maven/repository/2150952-release-4Nd0Uf/
                </altReleaseDeploymentRepository>
                <altSnapshotDeploymentRepository>
                    rdc-snapshots::default::https://packages.aliyun.com/maven/repository/2150952-snapshot-LmgYUo/
                </altSnapshotDeploymentRepository>
            </properties>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>https://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>snapshots</id>
                    <url>https://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>rdc-releases</id>
                    <url>https://packages.aliyun.com/maven/repository/2150952-release-4Nd0Uf/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>rdc-snapshots</id>
                    <url>https://packages.aliyun.com/maven/repository/2150952-snapshot-LmgYUo/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>https://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>snapshots</id>
                    <url>https://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>rdc-releases</id>
                    <url>https://packages.aliyun.com/maven/repository/2150952-release-4Nd0Uf/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>rdc-snapshots</id>
                    <url>https://packages.aliyun.com/maven/repository/2150952-snapshot-LmgYUo/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>rdc</activeProfile>
    </activeProfiles>
    </settings>" > /root/.m2/settings.xml



# 定义一些变量, 下面各阶段会使用
variables:
  jar_name: fileservice-0.0.1-SNAPSHOT.jar
  java_path: /usr/local/jdk/jdk1.8.0_321/bin
  TAG: file-service:v1.0  # 镜像名称
  CONTAINER_NAME: file-service
  PORT: 8900
  DOCKER_DRIVER: overlay2

# 定义执行的各个阶段及顺序
stages:
  - build
  - deploy

# 使用 maven 镜像打包项目
maven-build:
  stage: build
  image: maven:3.5.0-jdk-8
  script:
    - cd fileservice
    - mvn package -B -Dmaven.test.skip=true
  cache:
    key: m2-repo
    paths:
      - .m2/repository
  artifacts:
    paths:
      - fileservice/target/$jar_name


build-master: # 定义的 Jobs 之一，用于构建 Docker 镜像。负责执行 deploy 这一流程。具体执行 build 和 run。
  stage: deploy
  script:
    - docker rmi -f $TAG
    - docker build -t $TAG . # 构件镜像
    - docker rm -f $CONTAINER_NAME || true # 删除容器
    - docker run -d --restart=always --name $CONTAINER_NAME --net=host $TAG # 运行容器
  only: # 指定哪些branch的push commit会触发执行该job，本例子指定只有master才会执行deploy这个job
    - main
```

注: [pymjl大佬的- gitlab持续集成/持续部署](http://www.pymjl.com/articles/67)