---
layout: post
tags:
  - cpp
title: 'C++ 函数匹配及实参推断'
category: Program
---
记录一下函数重载、函数匹配、名字查找与实参类型推断的规则。

<!--more-->

## 函数重载

### 函数重载规则

 * 名字相同，形参类型不一样。
 * 不允许两个函数除了返回类型外其他所有的要素都相同。
 * 顶层`const`的形参无法和没有顶层`const`的形参区分。

### 重载与作用域

 * 内层作用域中声明的名字，将隐藏外层作用域中声明的同名实体。
 * 名字查找发生在类型检查之前。

## 名字查找

 * 首先，在名字所在的块中寻找其声明语句，只考虑在名字使用之前出现的声明。
 * 如果没有找到，继续查找外层作用域。
 * 如果最终没有找到匹配的声明，则程序报错。

对于类的定义：

 * 首先，编译成员的声明。
 * 直到类全部可见后才编译函数体。

## 函数匹配

### 函数匹配步骤

* **确定候选函数和可行函数**

 > 选择函数名相同的、声明在调用点可见的函数作用候选函数。
 > 在候选函数中选出形参数量匹配并且类型可转换的函数作用可行函数。

* **寻找最佳匹配**

 > 从可行函数中选择最匹配的函数，如果有多个形参，则最佳匹配条件为：
 >
 > * 该函数每个实参的匹配都不劣于其他可行函数需要的匹配。
 > * 至少有一个实参的匹配优于其他可行函数提供的匹配。
 >
 > 否则，发生二义性调用错误。

## 实参类型转换等级

具体排序如下：

 * 精确匹配：
   * 实参类型和形参类型相同。
   * 实参从数组类型或函数类型转换成对应的指针类型。
   * 向实参添加顶层`const`或者从实参中删除顶层`const`。
 * 通过`const`转换实现的匹配。
 * 通过类型提升实现的匹配。
 * 通过算术类型转换或指针转换实现的匹配。

## 函数指针

函数与函数指针的形参类型必须精确匹配。

## 类类型转换运算符匹配

**如果类类型和目标类型之间存在多种转换的方式，则可能发生二义性错误：**

 * 两个类型提供相同的类型转换。
 * 类定义了多个转换的规则。

当我们使用两个用户定义的类型转换时，如果转换函数之前或之后存在标准类型转换，则标准类型转换将决定最佳匹配到底是哪个。

## 重载函数与转换构造函数

如果两个或多个类型转换都提供了同一种可行的匹配，则这些类型转换一样好。

在调用重载函数时，如果需要额外的标准类型转换，则该转换的级别只有当所有可行函数请求同一个用户定义的类型转换时才有用。如果所需的用户定义的类型转换不止一个，则该调用具有二义性。

## 函数匹配与重载运算符

表达式中运算符的候选函数集既包括成员函数，也应该包括非成员函数。

## 模板实参推断

### 类型转换与模板类型参数

能应用于函数模板的类型转换：

 * 顶层 const 会被忽略。
 * const 转换：非 const 对象的引用（或指针）传递给一个 const 的引用（或指针）的形参。
 * 数组或函数指针的转换。

一个模板类型参数可以用作多个函数形参的类型。由于只允许有限的几种类型转换，因此传递给这些形参的实参必须具有相同的类型。

正常类型转换应用于显示指定的实参。

### 函数指针和实参推断

当参数是一个函数模板实例的地址时，程序上下文必须満足：对每个模板参数，能唯一确定其类型或值。

### 重载与模板

 * 对于一个调用，其候选函数包括所有模板实参推断成功的函数模板实例。
 * 候选的函数模板总是可行的，因为模板实参推断会排除任何不可行的模板。
 * 可行函数按类型转换来排序，如果有一个函数比其他函数匹配更好，则选择此函数。
 * 如果有多个函数提供同样好的匹配则：
   * 如果同样好的函数中 且个是非模板函数，则选择此函数。
   * 如果没有非模板函数，如果其中一个模板函数更特例化，则选择此模板。
   * 否则，此调用有歧义。

## 异常类型与 catch 类型的匹配

异常类型与`catch`类型必须精确匹配：

 * 允许从非常量向常量转换。
 * 允许从派生类向基类转换。
 * 数组与函数转换成指针。