# ABACUS开发者文档汇总

## 一、简要说明
本文档供ABACUS（原子算筹）程序的开发者阅读使用。
本文档提供了统一的文档目录，请文档编写者也严格按照目录的架构和格式来添加内容。本文档的阅读者定位为想了解目前程序架构的人，因此尽量客观的描述程序的现状，不建议在文档里讨论对程序修改的建议。

## 二、文件夹模板
每个模块的模板分为四部分：
  1. 功能介绍
  2. 代码介绍
    1. 代码位置
    2. 代码框架图
    3. 模块关系图
  3. 重点功能
  4. 参考文献
  
## 三、文件夹目录
1. 原子算筹
  
  * ABACUS 3.0后整体重构方案
  * 缩写规范
  * 符号规范
  * 全局类和全局变量
  * ABACUS里MPI的并行技术 
  * 自动文档Doxygen使用说明
  * 性能&内存统计规范 
  * ABACUS代码注释添加规范 
  * 测试 

2. 程序介绍
   1. Base模块
   2. IO模块
   3. Cell模块
   4. Basis模块
      1. 平面波模块
      2. 数值原子轨道模块（缺，负责人？）
   5. Psi模块
   6. ElecState模块
   7. Hamiltonian模块
      1. 模守恒赝势（缺，负责人？）
      2. Operator模块
      3. 溶剂算法
      4. DeePKS理论基础
      5. 平面波基组原子受力和晶格应力
      6. 数值原子轨道基组原子受力和晶格应力
      7. 格点积分模块
      8. 格点积分补充材料
      9. +U模块
   8. HSolver模块
   9. ESolver模块
      1. TDDFT功能（缺，负责人李源波）
   10. Relax模块
      1. Relaxation算法补充说明
   11. MD模块
   12. LibRI软件