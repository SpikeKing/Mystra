---
title: 在正式提交测试前的代码检查
date: 2016-04-16 11:37:00
categories: [经验]
tags: [经验]
---

正式提交工作, 就意味着写的代码归档, 会影响其他共同开发者, 也会影响项目, 因此, 有些东西就必须要检查, 不要凭着直觉去做. 作为一个优秀的程序员, 最重要的就是仔细, 当然也会适用于各行各业.

<!-- more -->
> 更多: http://www.wangchenlong.org/
> 欢迎Follow我的GitHub: https://github.com/SpikeKing

![Coding](161-check-code/check-code-logo.png)

---

## 功能

1. 检测**功能列表**. 确保所有功能都已经开发, 测试全部**测试用例**通过.

2. 检测**存储类功能**. 在``版本更新``时, 需要确保数据连续. 两种情况, 高版本覆盖低版本, 重新生成高版本; 在``删除首选项``或``清除缓存``时, 确保数据状态正确, 不会造成异常.

3. ``Proguard``. 对于引入**第三方开源库**时, 要注意正确设置Proguard, 通过生成**线上(Release)包**, 验证Proguard的正确性.


## 代码

1. **空指针**: ``Null Pointer``, 最关键. 绝大多数崩溃的元凶, 需要仔细排查, 以方法为单位, 仔细检查入口参数和私有变量, 确保不会发生空指针. 这也是为什么不要把所有变量都写成私有变量的原因之一, ``封装``也是面向对象的三大特性, 确保私有变量的精简. 

2. **Lint**: 如果还不知道或者没有使用过``Lint``, 那么代码一定不优雅. 在``Android Studio``中Lint的位置: ``Analyze -> Inspect Code...``. 使用Lint优化代码, 有些是可读性的, 有些是性能的, 有些是封装的. 确保所有的问题都已知晓, 但**不要照搬全做**, 只选择需要改的地方.

3. **注释**: 为自己也为他人. 为老去的自己写一些东西, 以防止突然的记忆空白. 为他人写一些东西, 让别人更加理解你. 如果是自己的项目, 确保添加``Readme``, 增加项目的可读性, 没有``Readme``的项目, 一文不值.


## 分支

1. **永远**在自己的分支上开发自己的功能. 

2. 在合并时, 把``主分支``合并到``自己的分支``, 处理冲突(Conflict). 完成后, 把自己的分支再合并到主分支, **注意顺序**. 

3. 在提交代码时, 需要合并**前置分支**, 确保以前版本的功能已经加入.

---

开发团队项目需要严谨, 代码写的不漂亮可以, 只要是你的极限就好. 善待你的项目, 善待你的工作, 成为一个优雅的程序员, 与君共勉.

OK, that's all! Enjoy it!

---

> 原始地址: 
> http://www.wangchenlong.org/2016/04/16/1604/161-check-code/
> 欢迎Follow我的[GitHub](https://github.com/SpikeKing), 关注我的[简书](http://www.jianshu.com/users/e2b4dd6d3eb4/latest_articles), [微博](http://weibo.com/u/2852941392), [CSDN](http://blog.csdn.net/caroline_wendy), [掘金](http://gold.xitu.io/#/user/56de98c2f3609a005442ec58). 
> 我已委托“维权骑士”为我的文章进行维权行动. 未经授权, 禁止转载, 授权或合作请留言.

