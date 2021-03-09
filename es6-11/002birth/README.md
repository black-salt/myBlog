# 一、JavaScript的诞生

<motto></motto>

1995年2月Netscape的 `Brendan Eic布兰登.艾奇` 开发了针对网景公司的 `Netscape Navigator` 浏览器的脚本语言LiveScript。Brendan Eich 有很强的函数式编程背景，希望以 Scheme 语言（函数式语言鼻祖 LISP 语言的一种方言）为蓝本，实现这种新语言。之后Netscape与Sun公司联盟后LiveScript更名为JavaScript。

之所以起这个名字，并不是因为 JavaScript 本身与 Java 语言有多么深的关系（事实上，两者关系并不深，详见下节），而是因为 Netscape 公司已经决定，使用 Java 语言开发网络应用程序，JavaScript 可以像胶水一样，将各个部分连接起来。当然，后来的历史是 Java 语言的浏览器插件失败了，JavaScript 反而发扬光大。

1995年12月4日，Netscape 公司与 Sun 公司联合发布了 JavaScript 语言，对外宣传 JavaScript 是 Java 的补充，属于轻量级的 Java，专门用来操作网页。

1996年3月，Navigator 2.0 浏览器正式内置了 JavaScript 脚本语言。

## 谁设计了 ECMAScript ?

答案是：TC39（技术委员会39）。

TC39，是一个推动 JavaScript 发展的技术委员会。其成员是公司（其中包括所有主要浏览器厂商）。TC39 定期举行会议，其会议是由成员公司派代表和特邀专家出席。会议纪要可在线获取，以便于我们更好的了解 TC39 工作流程。

偶尔（甚至在这本书中），你会看到 TC39 成员指的是某一个人。 它的意思是：由 TC39 成员公司派出的代表。

有趣的是，TC39 以协商一致方式运作：决定要求大多数人同意，没有人强烈反对以致否决。 对于许多成员来说，一致通过的决议将会产生真正的义务（他们将必须实现这些达成一致的功能等）。

## ECMAScript是如何设计的?

### 问题: ECMAScript 2015 （ES6）是一个包含了过多内容的发布

最近发布的 ECMAScript ， ECMAScript 2015（ES6）包含功能过多，在 ES5（2009年12月 – 2015年6月）之后差不多 6 年才被标准化。 两个版本发布相隔这么长时间，有两个主要问题：

* 对于稍早已经实现的功能必须等到整个版本完成后才能发布。
* 这些经过漫长等待的功能会在压力下打包，因为错过了这次发布意味着又一次的漫长等待才能发布。而且这些功能也有可能导致本次发布延迟。

因此，从 ECMAScript 2016（ES7）开始，发布将更频繁，因此内容会更小。 每年将发布一个版本，它将包含在截止日期之前完成的所有功能。

### 解决方案：TC39 过程

ECMAScript 功能的每个提案都将经过以下几个阶段逐渐成熟，从阶段 0 开始。从一个阶段到下一个阶段的推进必须由 TC39 批准。

#### 阶段0: 稻草人（头脑风暴式的提案）

**它是什么？** 一种自由形式的提交推进 ECMAScript 的想法。 提案意见书必须由TC39的成员或者 [已经注册成TC39贡献者](http://www.ecma-international.org/memento/contribute_TC39_Royalty_Free_Task_Group.php) 的非成员提出。

**需要什么？** 该意见书必须在 TC39 会议进行审查（[来源](https://github.com/tc39/ecma262/blob/master/FAQ.md)），然后将其添加到阶段 0 的[提案页面中](https://github.com/tc39/ecma262/blob/master/stage0.md)。

#### 阶段1: 建议

**这是什么？**功能正式的提案。

**需要什么？**必须确定所谓的提案负责人。该负责人或者协同负责人必须是TC39成员 ([来源](https://github.com/tc39/ecma262/blob/master/FAQ.md))。该提案所解决的问题必须说明清楚。解决方案必须通过实例描述，一个 API 和一个有关语义和算法的讨论。最后，必须明确该提案的潜在障碍，比如和其它功能的交互或者实现挑战。建议的实现方式、 polyfills 和演示也是必须的。

**下一步是什么？** 通过了阶段 1，才意味着 TC39 愿意来进行审议，讨论并为该提案付诸努力。在往下，将预计该提案所带来的主要变化。

#### 阶段2: 草案

**这是什么？** 第一个版本将在规范中。这个版本也可能是该特性纳入标准的最终版本。

**需要什么？** 该提案现在必须有该特性的语法和语义的形式描述（使用 ECMAScript 规范的形式语言）。描述应尽可能完整，但可以包含 todos 和 占位符 。该功能必须要有两个实验实现，但其中的一个可以通过transpiler（转译器）实现，比如 Babel。

**下一步是什么？** 从现在开始，一般只会有一些增量变化。

#### 阶段3: 候选

**这是什么？** 该提案已经基本完成，现在需要实现和用户的反馈，以便进一步推进。

**需要什么？** 该规范文本必须是完整的。指定评审员（由TC39任命，而不是负责人）和 ECMAScript 的规范编写人必须在规范文本上签字。必须有至少两个符合规范的实现（默认情况下不必启用）。

**下一步是什么？** 从此以后，只有在实现或其应用引发重大问题时，才会对该提案做出修改。

#### 阶段4: 完成

**这是什么？** 该提案已准备好纳入标准。

**需要什么？** 在提案可以达到这个阶段之前，需要做以下事情：

* [测试262](https://github.com/tc39/test262) 验收测试（大致为语言特性的单元测试，用JavaScript编写）；
* 两个符合规范的实现通过测试；
* 对于该实现的显著实践；
* ECMAScript的规范编写人必须在规范文本签字

**下一步是什么？** 该提案将尽快纳入 ECMAScript 规范。 当规范以年度批准为标准的时候，这个提案就被批准为一部分。

### 不要叫他们的 ECMAScript 20XX 特性

正如你所看到的，一旦提案进入阶段4，你也只能确保该提案将被包含在标准。而且，将其列入下一版本的 ECMAScript 是可能的，但也不是100％确定（可能需要更长的时间）。因此，你不能叫该提案：“ES7特性” 或 “ES2016特性” 了。因此，我的两个写标题的文章和博客的最喜欢的方法是：

* “ECMAScript 提案: the XXX feature”。该提案的阶段在本文开头提到。
* “ES. 阶段2: the XXX feature”

如果一个提案是已经进入阶段4，我可以叫它 ES20xx 特性，但最安全的还是要等到规范编写人证实了它将被包含在哪个发布中后。 `Object.observe` 是一个例子，它进入阶段 2 后，却最终却被撤回了。