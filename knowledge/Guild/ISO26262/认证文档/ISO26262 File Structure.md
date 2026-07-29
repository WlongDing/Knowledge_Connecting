

## Confirmation Management
Configuration Management Process ： 配置管理流程
Change Management Process : 变更管理流程
Quality Assurance Plan : 质量管控计划

ASPICE 流程中这一块尤为具体且重要

## Confirmation Review

	Safety Plan
	Safety Case
	Safety Analysis

## Verification Review

Verification Review Checklist
	[[Safety Assumption]]
	safety Reqirement
	[[Architecture Design Specification]]
	[[Safety Analysis FEMA]]

FMEA (Failure Mode and Effects Analysis)则是针对技术风险，是对产品开发和生产流程中进行预防性[质量管理](https://zhida.zhihu.com/search?content_id=159347334&content_type=Article&match_order=1&q=%E8%B4%A8%E9%87%8F%E7%AE%A1%E7%90%86&zhida_source=entity)的一种分析方法
按照AIAG&VDA标准的分类，如果将FMEA用于从进货到递交给客户的生产环节中，那么称为[PFMEA](https://zhida.zhihu.com/search?content_id=159347334&content_type=Article&match_order=1&q=PFMEA&zhida_source=entity)（Process FMEA），由工厂管理相关工程师负责；如果将FMEA分析方法用作产品开发环节中，那么称为[DFMEA](https://zhida.zhihu.com/search?content_id=159347334&content_type=Article&match_order=1&q=DFMEA&zhida_source=entity)（Design FMEA）

FMEDA（失效模式、影响及诊断分析）、FTA（故障树分析）、FMEA（失效模式与影响分析）、DFA（设计故障分析）这四种典型的安全分析方法

FMEDA 主要用于分析硬件系统中各组件的失效模式，评估其对系统功能安全的影响，并计算诊断覆盖率、失效率等量化指标。它通常应用于硬件开发阶段，在完成硬件架构设计后，对硬件组件进行深入分析，以确定系统是否满足功能安全要求。

FTA 是一种自上而下的演绎式故障分析方法，从系统的顶事件（不希望发生的故障事件）出发，通过逻辑门（如与门、或门等）逐步分析导致顶事件发生的各种可能的中间事件和底事件。适用于系统开发的多个阶段，在概念阶段可识别系统级潜在风险，在设计和开发阶段可深入分析系统故障原因和传播路径。

FMEA 是一种自下而上的归纳式分析方法，从系统的组件或零部件入手，分析每个组件可能出现的失效模式，以及这些失效模式对系统功能、性能和安全的影响。在系统开发的设计、生产、售后等各个阶段均广泛应用 。

DFA 主要关注系统设计过程中的潜在故障，通过对设计方案的审查和分析，识别设计缺陷、不合理的设计逻辑以及不符合安全规范的设计内容。适用于系统设计的早期阶段，如概念设计和详细设计阶段，可及时评估和优化设计方案，避免设计缺陷在后续开发中带来高额成本和风险。
## ISO26262 Part

-----
4-6 - [[Safety Assumption]] ： 功能安全假想

6-6 - safety Reqirement：需求

6-7 - [[Architecture Design Specification]] ：架构设计
- Coding Guildline
- DocumentTrackLink : DoorSLink
- Safety Analysis FEMA : 在架构这个阶段对模块的失效性分

6-8 Software Design：软件设计

6-9 Unit Test：单元测试  R0，R1，MC/DC

6-10 Integration Test：集成测试   函数覆盖度和函数分支覆盖度

6-11 Function Test：功能测试  基于需求测试

4-7 System Test：系统测试   

------
Safety Case：自己证明自己符合功能安全开发流程的文档，于各个Part中论证自己哪一部分的文档符合了这一part，或者说实现了这一part中提到的内容，并把相关evidence贴上去。

[[Safety Plan]]：该项目功能安全开发的生命周期，计划

Tool_Qualification_Report : 工具置信度报告



