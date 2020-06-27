# Maven

项目构建，依赖管理。

## 基本使用

### 创建新项目

![image-20200611001320021](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611001320021.png)

### 依赖管理

- Scope：依赖范围![image-20200611004135974](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611004135974.png)
- 依赖传递
- 依赖传递导致依赖冲突
  - 自己调解
    - 先声明优先
    - 短路原则，最短路径原则，直接定义比传递的路径短
  - 依赖排除
    - exclusions：exclusion
  - 版本锁定
    - dependencyManagement：dependencies：dependency

### 生命周期与插件

Maven生命周期：包含在一个项目构建中的一系列有序的阶段。

Maven有三套生命周期：

- 清理：clean

  - pre-clean：执行清理前工作
  - clean：清理上一次构件生产的文件
  - post-clean：执行清理后工作

- 构建：default

  - compile
  - test
  - package
  - install
  - 。。。

- 生成项目站点

  - pre-site
  - site：生成项目的站点文档
  - post-site
  - site-deploy：发布生成的站点到服务器上

  ![image-20200611005411971](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611005411971.png)

#### 演示jetty插件

![image-20200611005527006](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611005527006.png)

### 聚合和继承管理

#### 聚合

使用一条命令，构建多个模块。

![image-20200611010102920](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010102920.png)

#### 继承

消除重复，将相同的配置提取出来，例如groupId，version等。

![image-20200611010051906](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010051906.png)

![image-20200611010132001](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010132001.png)

### Maven私服搭建

![image-20200611010201521](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010201521.png)

![image-20200611010208173](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010208173.png)

#### 安装

![image-20200611010314593](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010314593.png)

![image-20200611010321398](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010321398.png)

### 脚手架开发

![image-20200611010956162](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611010956162.png)

#### 开发

![image-20200611011052047](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611011052047.png)

![image-20200611011100301](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611011100301.png)





# git

![image-20200611012607507](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012607507.png)

![image-20200611012700220](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012700220.png)

![image-20200611012737839](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012737839.png)

![image-20200611012812902](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012812902.png)

![image-20200611012842985](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012842985.png)

![image-20200611012905256](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012905256.png)

![image-20200611012928592](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012928592.png)

![image-20200611012940945](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012940945.png)

![image-20200611012947746](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012947746.png)

![image-20200611012954431](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611012954431.png)

![image-20200611013406609](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611013406609.png)

![image-20200611013847004](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611013847004.png)

![image-20200611013929110](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611013929110.png)

![image-20200611014048687](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611014048687.png)

![image-20200611014119586](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611014119586.png)

![image-20200611014249804](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611014249804.png)

## github



![image-20200611014348569](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611014348569.png)

![image-20200611014941424](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611014941424.png)

![image-20200611015131498](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015131498.png)

![image-20200611015157170](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015157170.png)

# Jenkins

![image-20200611015505099](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015505099.png)

![image-20200611015523673](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015523673.png)

![image-20200611015612937](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015612937.png)

![image-20200611015645223](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015645223.png)

# sonarQube

![image-20200611015718844](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015718844.png)

![image-20200611015737623](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015737623.png)

![image-20200611015746878](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015746878.png)

![image-20200611015801301](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015801301.png)

# 云课堂

微服务架构dubbo+springcloud

- dubbo：config、rpc解决方案
- springcloud

![image-20200611015940073](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200611015940073.png)