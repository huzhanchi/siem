## 整体介绍
dependency-check 是owsap下对java漏洞组件依赖的检查，不过官方只支持本地模式，即漏洞库本地更新，本地检查。我们做了本地轻量级的plugin，服务端做复杂检查的改进，同时能够阻断包含严重漏洞业务构建流程

* 整体架构

![image](https://github.com/huzhanchi/siem/blob/main/images/siem.png)

* 后台

![image](https://github.com/huzhanchi/siem/blob/main/images/backend.png)