# 使用华为云CloudArtifact Maven 私仓Workflow样例
<sup>私有依赖库（[CloudArtifact](https://support.huaweicloud.com/cloudrelease/index.html)）用于管理私有组件（开发者通俗称之为私服），包括Maven、Npm、Go、PyPI、Rpm等多种仓库格式。</sup>   
使用华为云CloudArtifact Maven 私仓有如下场景：  
1.mvn deploy 推送maven组件到 CloudArtifact Maven 私仓   
2.mvn install 拉取CloudArtifact Maven 私仓的maven组件 

## 前置工作
(1)[新建私有依赖库](https://support.huaweicloud.com/usermanual-releaseman/cloudrelease_01_0008.html)  
(2)[管理用户权限](https://support.huaweicloud.com/usermanual-releaseman/cloudrelease_01_0011.html)  
(3)CloudArtifact Maven 私仓账号信息获取  
[私有依赖库首页](https://devcloud.cn-north-4.huaweicloud.com/cloudartifact/repository)->点击需要的Maven仓库->右上角操作指导->点击下载配置文件->Maven配置在下载的settings.xml文件当中  
![图一](imgs/maven-setting-download.PNG)

## 参数说明
Maven-cloudartifact-action 参数都属于Maven settings.xml文件的元素，详细了解可以到官网[Maven Settings ](http://maven.apache.org/settings.html)。本action四个参数servers, mirrors,repositories,pluginRepositories都是json array的格式。下面给出四个参数的具体的样例。
### `servers`  
```yml
steps:
- uses: huaweicloud/Maven-cloudartifact-action@v1.0.0
  with:
    servers: '[{"id": "serverId", "username": "${{ secrets.MAVEN_USERNAME }}", "password": "${{ secrets.MAVEN_PASSWORD }}"}]'
```
### `mirrors`  
```yml
steps:
- uses: huaweicloud/Maven-cloudartifact-action@v1.0.0
  with:
     mirrors: '[{"id": "mirrorId", "mirrorOf": "mirrorOf", "url": "mirrorUrl"}]'
```
### `repositories`  
```yml
steps:
- uses: huaweicloud/Maven-cloudartifact-action@v1.0.0
  with:
    repositories: '[{ "id": "some-repository", "url": "http://some.repository.url", "releases": { "enabled": "true" }, "snapshots": { "enabled": "false" } }]'
```
### `pluginRepositories`  
```yml
steps:
- uses: huaweicloud/Maven-cloudartifact-action@v1.0.0
  with:
    plugin_repositories: '[{ "id": "some-plugin-repository", "url": "http://some.plugin.repository.url", "releases": { "enabled": "true" }, "snapshots": { "enabled": "false" }}]'
```

## **CloudArtifact Maven 私仓Workflow样例**
### 1.mvn deploy 推送maven组件到 CloudArtifact Maven 私仓 
步骤说明：  
(1)代码检出  
(2)华为云CloudArtifact maven 私仓配置  
(3)maven deploy 推送maven二进制包到华为云CloudArtifact maven 私仓
```yaml
name: Maven Cloudartifact Action Deploy Demo
on:
  push:
    branches:
       master
jobs:
  Publish-to-CloudArtifact:
    runs-on: ubuntu-latest
    steps:
        # 代码检出
      - uses: actions/checkout@v2

        # 华为云CloudArtifact maven 私仓配置 
      - name: Setup HuaweiCloud Maven CloudArtifact
        uses: huaweicloud/Maven-cloudartifact-action@v1.0.0
        with: 
          servers: '[{"id": "release_dfbdbf2e511e40fa88ba1d653358d65c_1_0", "username": "${{ secrets.MAVEN_USERNAME }}", "password": "${{ secrets.MAVEN_PASSWORD }}"}]'
    
        # 推送maven二进制包到华为云CloudArtifact maven 私仓
      - name: deploy artifact 
        run: |
          mvn deploy -e -X
```
详情可参考 ./github/workflow/maven-cloudartifact-action-deploy-demo.yml

### 2.mvn install 拉取CloudArtifact Maven 私仓的maven组件 
步骤说明：  
(1)代码检出  
(2)华为云CloudArtifact maven 私仓配置  
(3)maven install 拉取华为云CloudArtifact maven 私仓二进制包
```yaml
name: Maven Cloudartifact Action Install Demo
on:
  push:
    branches:
       master
jobs:
  Publish-to-CloudArtifact:
    runs-on: ubuntu-latest
    steps:
        # 代码检出
      - uses: actions/checkout@v2

        # 华为云CloudArtifact maven 私仓配置
      - name: Setup HuaweiCloud Maven CloudArtifact
        uses: huaweicloud/Maven-cloudartifact-action@v1.0.0
        with: 
          servers: '[{"id": "release_xxx_1_0", "username": "${{ secrets.MAVEN_USERNAME }}", "password": "${{ secrets.MAVEN_PASSWORD }}"}]'
          repositories: '[{ "id": "central", "url": "https://repo1.maven.org/maven2", "releases": { "enabled": "true" }, "snapshots": { "enabled": "false" } },{ "id": "release_xxx_1_0", "url": "https://devrepo.devcloud.cn-north-4.huaweicloud.com/07/nexus/content/repositories/xxx_1_0/", "releases": { "enabled": "true" }, "snapshots": { "enabled": "false" } }]'

        # 拉取华为云CloudArtifact maven 私仓二进制包
      - name: install artifact
        run: |
          mvn install -e -X
```
详情可参考 ./github/workflow/maven-cloudartifact-action-install-demo.yml