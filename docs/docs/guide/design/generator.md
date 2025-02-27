---
title: 出码模块设计
sidebar_position: 5
---
本篇主要讲解了出码模块实现的基本思路与一些概念。如需接入出码和定制出码方案，可以参考《[使用出码功能](https://www.yuque.com/lce/doc/cplfv0)》一节。
# npm 包与仓库信息
| **NPM 包** | **代码仓库** | **说明** |
| --- | --- | --- |
| [@alilc/lowcode-code-generator](https://www.npmjs.com/package/@alilc/lowcode-code-generator) | [alibaba/lowcode-engine](https://github.com/alibaba/lowcode-engine)
(子目录：modules/code-generator) | 出码模块核心库，支持在 node 环境下运行，也提供了浏览器下运行的 standalone 模式 |
| [@alilc/lowcode-plugin-code-generator](https://www.npmjs.com/package/@alilc/lowcode-plugin-code-generator) | [alibaba/lowcode-code-generator-demo](https://github.com/alibaba/lowcode-code-generator-demo) | 出码示例 -- 浏览器端出码插件 |

# 出码模块原理

出码模块的输入和输出很简单：
![](https://cdn.nlark.com/yuque/0/2022/jpeg/263300/1644825891969-1777dbe4-5ffc-4c94-a022-3aba0c116021.jpeg)

这里有几个概念：

- schema: 搭建协议内容，指符合《阿里巴巴中后台前端搭建协议规范》的 schema
- solution：出码方案，指具体的项目框架（如 Rax，Ice.js)
- Source Codes：生成的源代码，以目录树的形式进行描述

可以看出，这是一个与用户基本没有交互，通过既定的流程完成整个功能链路的模块。其核心暴露的是一个将搭建协议 schema 按既定的 solution 转换为代码的函数。对于使用者来说就是一个输入输出都确定的黑盒系统。

## 出码流程概述
出码模块和编译器很类似，都是将代码的一种表现形式转换成另一种表现形式，如：

**编译器流程**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/242652/1644403720547-e5021c3c-0c63-47ea-b8f0-1af79bbae3ef.png#clientId=u70bc6751-fff0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=246&id=u3b3e3126&margin=%5Bobject%20Object%5D&name=image.png&originHeight=492&originWidth=3228&originalType=binary&ratio=1&rotation=0&showTitle=false&size=110319&status=done&style=none&taskId=u32f55257-a18c-4caa-9334-0b3ca5b0bc0&title=&width=1614)

**出码模块流程**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/242652/1644402753768-02d402da-dd1a-4783-9021-606e276d4c68.png#clientId=u70bc6751-fff0-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=182&id=GlAz4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=182&originWidth=1536&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23743&status=done&style=none&taskId=ubbea9289-cb03-4fcf-9ce7-22da1ce077b&title=&width=1536)

## 出码流程详解
### 协议解析
协议解析主要是将输入的 schema 解析成更适合出码模块内部使用的数据结构的过程。这样在后面的代码生成过程中就可以直接用这些数据，不必重复解析了。
![](https://intranetproxy.alipay.com/skylark/lark/0/2021/jpeg/154771/1636960093808-624b5e50-18d6-476d-a951-556912832cdb.jpeg)
主要步骤如下：

- 解析三方组件依赖
- 分析 ref API 的使用情况
- 建立容器之间的依赖关系索引
- 分析容器内的组件依赖关系
- 分析路由配置
- 分析 utils 和 NPM 包依赖关系
- 其他兼容处理

### 前置优化
前置优化是计划基于策略对 schema 做一些优化。
主要逻辑分为分析、规则和优化三个部分，组合为一个支持通过配置进行一定程度定制化的策略包。每个策略包会先执行分析器，对输入进行特征提取，然后通过规则对特征进行判断，决定是否执行优化动作：
![](https://intranetproxy.alipay.com/skylark/lark/0/2021/jpeg/154771/1636960832211-0db6925b-c4b5-4be4-a883-fdd52a47e19a.jpeg)

### 代码生成
代码生成的流程如下：
![](https://intranetproxy.alipay.com/skylark/lark/0/2021/jpeg/154771/1636975834791-e3fe89f9-cd2d-446f-aa9f-fca0bb1d56c3.jpeg)
如果简单粗暴地拼字符串生成源代码将难以扩展和维护，因此出码模块在代码生成过程中将代码进行了一些抽象化。
日常开发中，我们常常是基于某一个特定的项目框架，将一些配置、UI 代码、逻辑代码放到他们应该在的地方，最终形成一套可以 run 起来的业务系统。那么其实对于出码这件事，我们也可以层层拆解，**项目 -> 插槽 -> 模块 -> 文件 -> 代码块**（代码片段）。这样就能将复杂的项目产出问题，拆分为一个个相对专注且单一的代码块产出问题，同时也支持组合复用。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/242652/1644402753919-1f063db3-5ad9-406b-9b45-6eeef79ea38b.png#clientId=u70bc6751-fff0-4&crop=0&crop=0&crop=1&crop=1&height=305&id=LgGUo&margin=%5Bobject%20Object%5D&name=image.png&originHeight=454&originWidth=892&originalType=binary&ratio=1&rotation=0&showTitle=false&size=230321&status=done&style=none&taskId=u50d38d84-3a3e-4c77-af05-8bc5f794643&title=&width=600)
注：中间表达结构即为对 Schema 解析后的结构化产物

**插槽**
首先来看下插槽，插槽描述了对应模块在项目中相对路径，并且可以对模块做固定的命名。每个插槽都有一系列插件来完成代码产出工作。生成的一个或多个文件，最终会依照插槽的描述放入项目中。
```typescript
// 项目模版
export interface IProjectTemplate {
  slots: Record<string, IProjectSlot>;
}

// 插槽
interface IProjectSlot {
  path: string[];
  fileName?: string;
}

// 插槽出码插件配置
interface IProjectPlugins {
  [slotName: string]: BuilderComponentPlugin[];
}
```
**代码块**
代码块是出码产物的最小单元，由出码模块插件产出，多个代码块最后会被组装为代码文件。每个代码块通过 name 描述自己，再通过 linkAfter 描述应该跟在哪些 name 的代码块后面。
```typescript
interface ICodeChunk {
  type: ChunkType; // 处理类型 ast | string | json
  fileType: string; // 文件类型 js | css | ts ...
  name: string; // 代码块名称，与 linkAfter 相关
  subModule?: string; // 模块内文件名，默认是 index
  content: ChunkContent; // 代码块内容，数据格式与 type 相关
  linkAfter: string[]; //
}
```

### 后置优化
后置优化分为文件级别和项目级别两种：

- 文件级别：在生成完一个文件后进行处理
- 项目级别：在所有文件都生成完了之后进行处理

文件级别的后置优化目前主要是有 prettier 这个代码格式化工具。
