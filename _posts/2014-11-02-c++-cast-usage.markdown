---
layout: post
title:  C++ 类型转换
date:   2014-11-02 01:41:20

---

写 test 的过程中终于捋开了的事，做个笔记。

## C++ 的类型转换

* static_cast
* const_cast
* reinterpret_cast
* dynamic_cast
* C-style cast

### static_cast：常见类型转换

* 基本被用来做转换 numeric data type 之类的。
* 不能把指针/引用转成非指针/引用。（reinterpret_cast 能做这个）
* upcast（把子类指针 cast 成父类指针），妥妥儿的。
* downcast（把父类指针 cast 成子类指针），不太可靠，因为 Standard C++ 里的它根本不做 run-time 类型检查（不过也就没有这个 overhead），类型不对的时候行为是 undefined。一般要搭配 assert 使用。
* 安全版：[safe_cast](https://msdn.microsoft.com/en-us/library/23b7yy6w.aspx)

### dynamic_cast：安全的指针引用 downcast

* 为了 run-time type check (← overhead) 诞生的 cast！
* 一般用在不知道被转的东西是什么类型的场合。
* upcast，妥妥儿的。
* downcast，会跑 run-time type check 看被转的东西的类型。
	* 转不了的，要转成 pointer 的会返回个 null pointer，转成 referenced 会抛 bad_cast exception。
	* 从继承树里出现一次以上的祖先 cast 下来，或者 cast 成树里出现了一次以上的子孙时会不知道该从哪条路转换而 fail。要从没有 ambiguity 的地方一个一个爬树。
* sidecast，跟 downcast 做的差不多。

### const_cast：用来去掉被转的东西的 const 属性

* 能去掉 const/volatile/__unaligned。
* 临时把一个指针或引用变成非 const 的，以便修改它指的对象的值。
* 去不掉 const variable 的 const。
* 朋友，你知道什么是 [const correctness](https://isocpp.org/wiki/faq/const-correctness) 吗？说不能转就不能转，让转也别转。

### reinterpret_cast：换标签（。
* 功能如字面意思，把被转换的对象直接降到位层面当成新类型来解释，长度塞不下的不让转，其他的随便转。
* 可以在 pointer 和 integral type 之间互转。static_cast 干不了这个。
* 一朵奇葩——“我做完了！虽然不知道对不对”。危险点在于转换得太任性以至于没法保证转出来的东西能用。

### C-style cast：挨个儿试一遍
* 把除了 dynamic_cast 之外的东西都试一遍。一部分人把 static_cast 的普通用法省事儿写成这个。
* 按这个顺序：
	* const_cast
	* static_cast
	* static_cast then const_cast（先转换再去掉 const）
	* reinterpret_cast
	* reinterpret_cast then const_cast
* Don't.
	* 可能会随便去掉 const，所以（。
	* 试 static_cast 的时候会无视 access control，所以（。
	* reinterpret_cast 不安全，这个有可能用到 reinterpret_cast，所以（。

<br />

差不多就是这样吧。
