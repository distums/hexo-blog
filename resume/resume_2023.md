# 个人信息

- 涂明晟/男/1989
- 本科/数学与应用数学/华南理工大学
- 工作年限：11年
- Github：[https://github.com/distums](https://github.com/distums)
- 期望职位：前端开发
- 期望城市：广州 / 深圳

---

# 联系方式

- 手机：15710608830
- Email：[distu@sina.com](mailto:distu@sina.com)

---

# 个人优势

编程技能：熟悉 js/css/html 以及各种原生 web api，熟练使用 react/vue 进行 pc/h5 页面开发，能基于 webpack 以及 babel 等搭建前端构建体系。后端方面基于 koa/midway 做过后端服务以及函数式应用程序的开发，在桌面端领域基于 electron 开发了饿了么商家端应用程序；

团队合作：擅长与他人合作，能够有效地和关联方进行沟通以及协调参与成员工作，共同保障项目顺利进行；

解决问题的能力：善于分析和解决问题，当面临问题时，能够清晰地思考并利用各种调试工具或者方法论去找到解决方案；

快速学习能力：始终保持学习的状态，积极关注前端领域的最新技术和趋势，包括但不限于新的浏览器 api 或者框架的最近进展；

---

# 工作经历

## 饿了么（2018年7月 ~ 至今）
先后负责了多个重要业务（比如订单）的前端开发工作，承担需求的评审、系统分析以及协调其他同学参与开发工作。同时作为团队内前端研发工程领域负责人，承担了团队脚手架以及构建器的功能设计和维护工作。

### 商家端订单页面
项目描述:  
饿了么 b 端订单页面，用于商家进行接单、打印小票、出餐以及跟进订单状态，是商家每天打开电脑必定会使用的页面，日 uv 20 万左右。

我的贡献和职责:  
负责商家端订单页面的日常需求开发和功能维护；  
在接手项目之初，详细梳理了订单业务，并制定了基于微应用技术的重构方案，对订单页面进行了重构；  
我进行了打印模块的调研，并制定了升级方案，将小票打印从本地打印迁移到云打印模式；

项目成果和亮点:  
在订单页面的重构发布过程中，保持了稳定性，没有出现故障；  
通过重构，页面加载时长从原来的7.1秒降低到了3.1秒，大幅度提升了页面的加载速度和用户体验；  
成功完成了小票模块对 XP 系统电脑的适配，确保了商家在不同操作系统上都能正常打印小票；

### 统一商运平台
项目描述:  
饿了么 b 端商户运营平台，用于商户运营全场景全流程的异常问题进行监控及治理。比如围绕平台商品质量不高问题，可以圈选出有问题的商家下发相应的整改动作。核心是流程图编辑器以及规则节点的表单设置。

我的贡献和职责:  
负责需求的评审、方案的制定、协调各参与同学的任务以及跟进需求的进度；  
基于 antv6 完成了流程图编辑器的开发，以及 antv6 数据格式到 bpmn 格式的转换；

项目成果和亮点:  
高质量完成了需求开发，顺利地完成了功能的测试和上线；  
沉淀了 b 端运营平台类应用开发标准方案，降低后续同学的开发成本；

### 巨鳄平台
项目描述:  
饿了么 C 端选品及投放平台，用于根据指定规则圈选一批商家/商品，配合搭建平台搭建好页面后，指定页面在饿了么 app 里某个资源位根据特定的规则（时间、区域、人群等）透出页面。

我的贡献和职责:  
负责前后端一体化应用的开发和搭建，使用 React 和 Ant Design 构建前端界面，保证用户界面友好且具有良好的用户体验；  
设计并实现了基于 JSONSchema 和 react-jsonschema-form 的表单渲染方案，简化了新投放场景的接入流程（大约 200 行的配置代码，耗时仅需 10-30 分钟即可完成接入），大大提高了团队的工作效率；  
使用 midway 搭建接口服务，并与后端团队实现接口对接，保证前后端的无缝集成和数据的正确传输；

项目成果和亮点:  
成功搭建和维护了巨鳄投放平台、用户运营平台和任务平台，为公司提供了强大的运营和管理工具；  
设计的表单渲染方案大大简化了新投放场景的接入流程，节省了团队的时间和精力，并在一年内将投放场景从接手时的 25 个拓展到了 200+，产生了超过 9 万次的运营配置数据；  
设计了自助接入流程以及问题排查工具，进一步优化接入以及运营排查问题效率；

### electron 客户端
项目描述:  
饿了么商家端 pc 版本，商家通常通过这个客户端来访问 b 端的页面。最核心的能力是小票打印，因为目前没有 web api 可以用来获取系统连接的打印机。

我的贡献和职责:  
参与桌面端从 nw.js 到 electron 的改造工作，负责了安装以及卸载脚本的编写、自动升级功能的设计以及开发和打印模块的迁移工作；  
负责后续新客户端的维护工作，包括功能的优化，商户反馈问题的排查和修复等；

项目成果和亮点:  
通过客户端升级，将 Chrome 内核从 v50 升级到 v96，带来约 20% 的性能提升，提升了使用新客户端的用户体验；  
借助公司网关平台设计的自动升级功能，可以方便地进行客户端的灰度更新；  
新客户端建立了完善的错误和崩溃日志上报机制，解决了旧客户端中难以处理的打印问题；

## 陆金所 （ 2016年6月 ~ 2018年7月 ）

### 陆金所APP

陆金所APP内嵌了很多h5页面，用于理财产品的展示。任职期间，主要工作内容是：

- 根据设计师给出的交互及视觉稿，完成新h5页面/功能的开发；
- 承接部分基础架构需求的开发，主要有基础组件的开发、打包过程的维护和优化；
- 应后台团队请求，负责对其团队pc端客户IM系统进行重写，并且针对前后端项目一体的特性自定义了一些webpack插件，简化了打包步骤，改造后开发维护成本大大降低，后续交由外包同学专职维护；

## 多点生活中国网络科技有限公司 （ 2015年8月 ~ 2016年5月 ）

### AMS活动管理系统

专门为运营人员打造的活动页搭建系统，系统里提供了多种标准化的组件，运营人员按需拖拽组合不同的组件，然后编辑相应的属性，即可短时间内完成各种打折/秒杀/促销活动页的发布上线，解决了以往运营人员每要上一个新活动都要开发介入的问题。其中我主导了AMS应用的技术选型以及整体项目的搭建，并且开发了系统所需的各个组件以及最终活动页面的生成。

## 厦门煦逸信息科技有限公司 （ 2014年9月 ～ 2015年6月 ）

### 65代购主站、ezbuy站点

跨境电商平台，主要为东南亚用户提供淘宝代购服务，65代购主站供新加坡和马来西亚用户使用，而ezbuy则是给澳洲用户使用。任职期间主要工作是：

- 站点日常需求开发，包含前端页面以及后端接口实现（.NET）；
- 参与站点的重构以及性能优化，代表有站点的首页，将原来后端直出的模式修改为异步加载，提升了网站首页首屏渲染速度；
- 站点活动页面的开发，代表有公司5周年活动65eday活动页面；

## 深圳英华达科技有限公司 （ 2014年7月 ～ 2014年9月 ）

任职期间作为公司 NodeJs 团队开发人员，参与公司创新业务：团队协作系统的开发。

## 厦门至正信息技术有限公司 （ 2012年8月 ～ 2014年6月 ）

作为公司软件研发人员，使用 ASP.NET/jQuery/kendoJs 等技术参与公司机队管理系统和飞机飞行译码平台的研发。

---

# 致谢

感谢您花时间阅读我的简历，期待能有机会和您共事。
