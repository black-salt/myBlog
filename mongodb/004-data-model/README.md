# 004 - MongoDB 数据模型

<motto></motto>

没有固定的数据格式 != 无需设计数据模型

文档结构 → 数据之间的关系

内嵌式结构 V. S 规范式结构

## 1. 内嵌式结构

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_20-39-34.png" alt="Snipaste_2020-05-29_20-39-34" style="zoom:50%; " />

## 2. 规范式结构

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_20-44-04.png" alt="Snipaste_2020-05-29_20-44-04" style="zoom:50%; " />

## 3. 文档关系：一对一

### 使用内嵌结构

好处：

* 一次查询就可以返回所有数据
* 更具独立性的数据应作为顶层文档
* 补充性数据应作为内嵌文档

## 4. 文档关系：一对多

### 1. 使用内嵌结构

好处：

* 一次查询就可以返回所有数据
* 适合读取频率**远高于**更新频率的数据

坏处：

* 更新内嵌文档的复杂度增高

### 2. 使用规范结构

好处：

* 减少了重复数据
* 降低了文档更新的复杂度

坏处：

* 需要多次读取操作才能得到完整的数据

### 3. 使用数组字段内嵌(需更改顶层文档)

* 适合常常需要返回全部相关文档的查询

* 数组元素较多时，避免使用内嵌文档
* 数组元素极多时，重新设计文档结构

### 4. 使用数组字段+ 内嵌+规范

* 适合常常需要返回全部相关文档的查询

* 数组元素较多时，避免使用内嵌文档
* 数组元素极多时，重新设计文档结构

## 5. 文档关系：树形结构

多见于**静态数据**及**更新频率比较低**的参考数据

### 存父节点：

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_21-07-20.png" alt="Snipaste_2020-05-29_21-07-20" style="zoom:50%; " />

### 存子节点：

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_21-13-33.png" alt="Snipaste_2020-05-29_21-13-33" style="zoom:50%; " />

### 存所有祖先节点：

可跨越层次

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_21-18-19.png" alt="Snipaste_2020-05-29_21-18-19" style="zoom:50%; " />

### 存深度优先的编号

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_21-21-13.png" alt="Snipaste_2020-05-29_21-21-13" style="zoom:50%; " />

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-05-29_21-23-15.png" alt="Snipaste_2020-05-29_21-23-15" style="zoom:50%; " />

