# jenkins


## 安装

下载 `jenkins.war`，在jekenis文件夹内启动终端，输入命令 `java -jar jenkins.war --httpPort=3344`

打印如下信息
```
Running from: F:\jenkins\jenkins.war
webroot: $user.home/.jenkins
2020-11-29 03:01:46.306+0000 [id=1]     INFO    org.eclipse.jetty.util.log.Log#initialized: Logging initialized @692ms to org.eclipse.jetty.util.log.JavaUtilLog
2020-11-29 03:01:46.450+0000 [id=1]     INFO    winstone.Logger#logInternal: Beginning extraction from war file
2020-11-29 03:01:48.417+0000 [id=1]     WARNING o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
2020-11-29 03:01:48.474+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: jetty-9.4.30.v20200611; built: 2020-06-11T12:34:51.929Z; git: 271836e4c1f4612f12b7bb13ef5a92a927634b0d; jvm 1.8.0_231-b11
2020-11-29 03:01:49.465+0000 [id=1]     INFO    o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
2020-11-29 03:01:49.531+0000 [id=1]     INFO    o.e.j.s.s.DefaultSessionIdManager#doStart: DefaultSessionIdManager workerName=node0
2020-11-29 03:01:49.531+0000 [id=1]     INFO    o.e.j.s.s.DefaultSessionIdManager#doStart: No SessionScavenger set, using defaults
2020-11-29 03:01:49.538+0000 [id=1]     INFO    o.e.j.server.session.HouseKeeper#startScavenging: node0 Scavenging every 660000ms
2020-11-29 03:01:49.952+0000 [id=1]     INFO    hudson.WebAppMain#contextInitialized: Jenkins home directory: C:\Users\USERNAME\.jenkins found at: $user.home/.jenkins
2020-11-29 03:01:50.090+0000 [id=1]     INFO    o.e.j.s.handler.ContextHandler#doStart: Started w.@200a26bc{Jenkins v2.249.3,/,file:///C:/Users/USERNAME/.jenkins/war/,AVAILABLE}{C:\Users\USERNAME\.jenkins\war}
2020-11-29 03:01:50.134+0000 [id=1]     INFO    o.e.j.server.AbstractConnector#doStart: Started ServerConnector@57db2b13{HTTP/1.1, (http/1.1)}{0.0.0.0:3344}
2020-11-29 03:01:50.135+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: Started @4522ms
2020-11-29 03:01:50.135+0000 [id=23]    INFO    winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
2020-11-29 03:01:51.354+0000 [id=30]    INFO    jenkins.InitReactorRunner$1#onAttained: Started initialization
2020-11-29 03:01:51.390+0000 [id=33]    INFO    jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2020-11-29 03:01:52.765+0000 [id=33]    INFO    jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2020-11-29 03:01:52.770+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Started all plugins
2020-11-29 03:01:52.776+0000 [id=39]    INFO    jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2020-11-29 03:01:53.446+0000 [id=28]    INFO    jenkins.InitReactorRunner$1#onAttained: System config loaded
2020-11-29 03:01:53.446+0000 [id=41]    INFO    jenkins.InitReactorRunner$1#onAttained: System config adapted
2020-11-29 03:01:53.447+0000 [id=36]    INFO    jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2020-11-29 03:01:53.447+0000 [id=30]    INFO    jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2020-11-29 03:01:53.456+0000 [id=56]    INFO    hudson.model.AsyncPeriodicWork#lambda$doRun$0: Started Download metadata
2020-11-29 03:01:53.467+0000 [id=56]    INFO    hudson.util.Retrier#start: Attempt #1 to do the action check updates server
2020-11-29 03:01:54.228+0000 [id=41]    INFO    o.s.c.s.AbstractApplicationContext#prepareRefresh: Refreshing org.springframework.web.context.support.StaticWebApplicationContext@3eceddba: display name [Root WebApplicationContext]; startup date [Sun Nov 29 11:01:54 CST 2020]; root of context hierarchy
2020-11-29 03:01:54.229+0000 [id=41]    INFO    o.s.c.s.AbstractApplicationContext#obtainFreshBeanFactory: Bean factory for application context [org.springframework.web.context.support.StaticWebApplicationContext@3eceddba]: org.springframework.beans.factory.support.DefaultListableBeanFactory@53a6b05f
2020-11-29 03:01:54.246+0000 [id=41]    INFO    o.s.b.f.s.DefaultListableBeanFactory#preInstantiateSingletons: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@53a6b05f: defining beans [authenticationManager]; root of factory hierarchy
2020-11-29 03:01:54.464+0000 [id=41]    INFO    o.s.c.s.AbstractApplicationContext#prepareRefresh: Refreshing org.springframework.web.context.support.StaticWebApplicationContext@652991be: display name [Root WebApplicationContext]; startup date [Sun Nov 29 11:01:54 CST 2020]; root of context hierarchy
2020-11-29 03:01:54.464+0000 [id=41]    INFO    o.s.c.s.AbstractApplicationContext#obtainFreshBeanFactory: Bean factory for application context [org.springframework.web.context.support.StaticWebApplicationContext@652991be]: org.springframework.beans.factory.support.DefaultListableBeanFactory@7eb3d0e6
2020-11-29 03:01:54.466+0000 [id=41]    INFO    o.s.b.f.s.DefaultListableBeanFactory#preInstantiateSingletons: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@7eb3d0e6: defining beans [filter,legacy]; root of factory hierarchy
2020-11-29 03:01:54.684+0000 [id=41]    INFO    jenkins.install.SetupWizard#init:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

b097438d3d6e867d8c5573b6296753d3

This may also be found at: C:\Users\USERNAME\.jenkins\secrets\initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2020-11-29 03:05:10.359+0000 [id=56]    INFO    h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2020-11-29 03:05:10.360+0000 [id=56]    INFO    hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
2020-11-29 03:05:10.364+0000 [id=56]    INFO    hudson.model.AsyncPeriodicWork#lambda$doRun$0: Finished Download metadata. 196,907 ms
2020-11-29 03:05:52.342+0000 [id=36]    INFO    jenkins.InitReactorRunner$1#onAttained: Completed initialization
2020-11-29 03:05:52.356+0000 [id=22]    INFO    hudson.WebAppMain$3#run: Jenkins is fully up and running
```

## 进入页面

访问 `http://127.0.0.1:3344`，输入账号密码

密码就是 `b097438d3d6e867d8c5573b6296753d3`

完成配置

## 创建任务

`New Item` --> `Enter an item name` --> 选择一个项目

进入配置任务页面



## 配置任务

- General

　　目前仅需填写描述,其他不做了解

- 源码管理

    源码管理分两种方式，本地和远程(git/svn,两种操作方式类似)

    选择 `git`，填写仓库、需要构建的分支信息等

    本地选无,然后把代码文件拉入jekenis工作空间中(workspace)


- 构建触发器

    - 触发远程构建 (例如,使用脚本):
    - 使用远程脚本出发任务
    - git仓库代码更新后执行


- 构建环境

- 构建

- 发送构建邮件配置

