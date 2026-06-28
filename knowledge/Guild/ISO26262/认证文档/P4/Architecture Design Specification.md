
## # 1.            Overiew

## 1.1         Purpose

目的是满足功能安全需求的架构设计
# 2.            Scope & Objective

## 2.1.        Scope

声明认证范围

# 3.            Reference Materials


# 4.            Definitions, Acronyms and Abbreviations

# 5.            Resource Limitation

资源消耗 -- ASPICE才要用到

|   |   |   |   |   |
|---|---|---|---|---|
|**Module**|**RAM Data Size(bytes)**|**ROM Data Size(bytes)**|**STACK Data Size(bytes)**|**CPU** **Load(%)**|
# 6.            Hardware Support

# 7.            MCAL Software Architecture safety concept overview

## 7.1.        Software Architecture Overview


概述软件架构，软件包含哪些模块，要做到ASIL几

说明模块之前的独立性，如果有模块之间的交互也应说明清楚

对于每个模块都概述一下（包内不涉及认证的模块，以及涉及认证的都简单说明一下）

包括底层的Common部分


## 7.2.        Tool Interaction Overview

---- --

# 8.            Software Functional Design

## 8.1.        Function Description
表明将使用安全的需求，满足26262流程的开发，并继承软件需求来指导模块设计实现

说明软件层级

概述一下各模块的功能，定下架构ID，如果有与其他模块的调用关系，需要说明（此为及静态设计）
附上模块之间的调用图（此为动态设计）

## 8.2.        Safety mechanisms for MCAL drivers

描述安全机制，什么检查，报错之类的，
安全状态机（如果有），没有的话也要说明

# 9.            Initiate process design

不同类型的初始化说明
分类，一类是怎么样，一类是怎么样

# 10.            Application scheduling design

调度是怎么样的，有则写，无则声明

# 11.            Compilation Guideline

引用