---
title: 企业人员的臃肿是怎么来的
date: 2023-12-02 23:20:04
author: repoog
excerpt: 一家企业的成长过程中，随着员工人数的增加，有的人会发现企业人员越来越臃肿，甚至人员增加了几倍，产出效率却依然保持不变。本文从笔者亲历的一件事情分析为什么企业人员会随着发展越来越臃肿。
comments: true
tags:
  - 人员管理
  - 企业管理
  - 信息化
  - 负担转移
categories:
  - 管理经验
---

在上一家的公司中，笔者的职位是CIO（首席信息官），负责公司的信息化、IT和信息安全工作，同时也负责处理了公司内外部流程中的诸多事件，梳理内部管理和运营流程，提升公司的运作效率和协作效率。这其中，见证了公司的人员规模近四倍的增长，但有人会发现，虽然人数规模上涨很多，但真正干活的依旧是原来能干活的人，真正有价值的也依然是那些少数的人。类似的现象在诸多公司都常见，即工作的产出和效率并未和员工规模成正比。

在本文中，笔者以亲历的一件事情作为示例，解释这个现象是怎样发生的，问题的根源在哪里，以及如何解决这样的问题。

当时公司的业务是SaaS产品，但在客户的使用过程中总可能会遇到不同的问题，包括新的需求、功能反馈、产品缺陷等等，也会在产品销售过程中，随着客户对于产品的了解深入，提出系统对接等定制开发类的需求。需求和缺陷的第一手接触人员通常会是销售人员或客户成功，而负责开发需求和解决缺陷的团队自然而言是产品团队，其中从成本节约的角度，定制开发的工作选择了由第三方的外包开发团队负责。

因此，当销售人员在商务洽谈阶段遇到客户的定制开发（通常是系统对接开发）需求时，需要将需求最终同步给外包团队，完成开发后再由客户做结果和质量确认。

在初始情况下，上述流程中涉及到的公司人员只是销售人员，但客户定制开发的具体需求和场景各有不同，销售人员的能力无法真正理解和传递客户的真实需求，对于外包团队而言容易造成开发结果或质量的偏差，也因此容易造成开发成本的偏差，或是偏高，或是偏低，对于公司而言会支付不必要的成本。

为了解决这个问题，公司专门成立了解决方案部门，由销售人员和解决方案部门对接，同步客户的需求，因而增加了解决方案专家理解客户场景和客户需求，再通过解决方案专家和外包团队沟通需求，确定开发需求和开发费用。这时，在销售人员的基础上又增加了解决方案专家的人员。

但定制需求的来源每个月或每个阶段是不同的，有时客户需求多，有时客户需求少，单靠解决方案专家协同销售、客户和外包团队忙不过来，且缺乏项目跟进，因此，解决方案部门又额外引入了项目经理岗位，此时的解决方案部门人数便轻松的突破了5人。

当定制需求来临时，整体协作流程如下：

1. 销售人员收到客户的定制开发需求；

2. 销售人员通过系统提交定制开发需求申请；

3. 解决方案部门负责人审批需求申请，如审批通过，则分配项目经理；

4. 项目经理与销售人员沟通、确认定制开发需求；

5. 明确需求后，项目经理通过系统向外包团队提出开发申请；

6. 外包团队负责人根据开发内容描述评估开发周期、开发费用；

7. 收到开发周期、开发成本后，由解决方案部门负责人确认；

8. 确认通过后，开发周期、开发成本通过系统同步销售人员；

9. 销售人员和客户沟通、确认开发周期和开发成本。

当某个时期面临的定制需求增加时，为了解决项目跟进和解决方案制定的问题，解决方案部门的方案专家和项目经理人数便也会增加，一度人数突破10人，如果加上外包团队的人数，至少也会在20人以上规模。

上述流程的弊端也是显而易见的，主要有以下问题：

1. 解决方案部门的效率会成为客户与产品之间需求和反馈的瓶颈；

2. 流程中的销售人员、项目经理、外包团队之间需求沟通、确认等过程会成为效率瓶颈；

3. 流程中的审批环节过多，会增加定制需求评估、费用确认的时间成本；

4. 外包团队在同类需求或同类功能开发报价时并不会降低报价，因此间接增加客户和公司成本；

5. 外包团队研发人员的能力和时间不可控，且研发代码不完全会交付公司，会增加质量风险和维护成本；

期间发生一次销售投诉事件，事件中销售人员正在极力争取客户的订单，客户于周五向各个供应商提出定制需求，并要求第二天反馈需求定制的费用，其他供应商在第二天甚至当天便反馈了评估结果，而销售人员虽然在系统中提交了定制需求申请，但基于以往解决方案部门反馈速度，销售人员等不及反馈，便依据历史项目的经验给客户做了答复。此举无疑是绕开了公司的流程，可能会造成公司在该项目上不必要的成本投入。

此事件的分歧在于，从销售角度为了获得订单必须尽快反馈客户结果（疑惑为何公司评估速度这么慢，并因此已经造成订单丢失），从公司的角度，为了获得定制需求的准确评估该流程是必要的，且销售人员没有遵守内部流程。

事后联合解决方案部门、销售管理部门、销售部门调研，以及平台数据统计分析，发现：

1. 解决方案部门对于定制需求的的评估周期平均是4-5天，最长达8天；

2. 销售人员至项目经理再到外包团队的沟通，缺乏结构化的协作内容，依赖各个人员的沟通和表达能力；

在这个案例中，公司人员增加了一个部门，但效率和结果并未因此而得到同比的改善或增加，甚至对于沟通能力强的销售怀念早前自己直接和外包团队沟通的日子。当销售管理总监和笔者提到当年定制需求的费用相比前一年同比增加了许多时，笔者反倒认为这是坏事，即公司陷入了“负担转移”陷阱。

负担转移陷阱指的是，为纠正问题而使用短期缓解办法，看似立即奏效，但随着方法的反复运用，更根本的长期纠正办法越来越被忽视，最终的结果是开发根本解决方法的能力萎缩或消失，导致对短期缓解办法的严重依赖。

![负担转移模型](images/2023/12/strain-transfer.png)

当公司看到定制需求的费用增加，费用评估的结果精准（公司认为只要评估精准，周期增加不是问题）时，会很容易认为是由于新部门成立的结果，但却忽视了上文中销售反馈的副作用，即在未建立绝对的产品优势时，响应速度便是客户判断和选择供应商的依据之一。

在和销售管理部门沟通后，笔者提出的最终解决方案如下：

1. 取消解决方案部门审批和项目经理跟进的工作内容；

2. 构建3-4人左右的专门负责定制开发的开发团队，每年支付的外包开发费用远超过该规模的人员成本；

3. 根据历史定制需求的内容，设计、制作需求评估调查表，用于销售人员和客户进行需求基础沟通；

4. 销售人员根据需求评估调查表与定制开发团队沟通，确定开发周期；

5. 半年或年度复盘定制开发需求，同步共性需求和开发成果至产品团队，作为产品功能或插件之一；

通过上面的案例，可以发现企业遇到的问题往往是固定的，区别在于量的不同，但组织结构和工作系统的设计决定了解决问题的效率和质量，不合理的组织结构会影响工作质量（康威定律），不恰当的工作系统会增加企业负担，或者造成潜在的负面影响。与工程实现类似，当设计系统时需要基于数据分析（但不依赖）以及业务逻辑，绘制系统运作的流程，寻找最短的路径，就像产品增加新的功能，对于企业每个新增岗位和部门都需要谨慎考虑：

1. 它的设立会带来什么样的价值；

2. 如果没有这个人会带来什么损失；

毕竟，一堆人做成一点事不算什么本事，一点人搞定一堆事才是本事。