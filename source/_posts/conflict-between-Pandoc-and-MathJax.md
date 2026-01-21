---
title: 越俎代庖的公式渲染问题
date: 2026-01-21
tags: [others]
categories: [misc]
math: true
---


解决了基于 Hexo + Fluid 的环境下，使用 Pandoc 作为渲染引擎时，部分数学公式无法正常渲染的情况。

<!--more-->

# 现象描述

Pandoc 渲染器与 MathJax 脚本之间出现了冲突：部分简单的行内公式字体异常，且**右键点击无法弹出 MathJax 菜单**。而另一部分复杂的公式渲染正常。问题同时出现在行内公式与行间公式中。这类异常的渲染虽然不会改变公式内容的正确性，但是看起来既不美观也不规范。

举个例子，当按下 F12 检查网页源代码的时候，渲染异常的公式可能是这样的：

``` html
𝒜 = {<strong>a</strong>|<strong>a</strong> ∈ {0, 1}<sup><em>N</em></sup>, ∥<strong>a</strong>∥<sub>1</sub> ≤ <em>m</em>}
```

而正常渲染的公式一般是这样的：
``` html
<mjx-container class="MathJax CtxtMenu_Attached_0" jax="CHTML" style="font-size: 113.1%; position: relative;" tabindex="0" ctxtmenu_counter="11"><mjx-math class="MJX-TEX" aria-hidden="true"><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D445 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-mi class="mjx-i" size="s"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi></mjx-script></mjx-msub><mjx-mo class="mjx-n" space="4"><mjx-c class="mjx-c225C TEX-A"></mjx-c></mjx-mo><mjx-munderover space="4" limits="false"><mjx-mo class="mjx-sop"><mjx-c class="mjx-c2211 TEX-S1"></mjx-c></mjx-mo><mjx-script style="vertical-align: -0.285em; margin-left: 0px;"><mjx-mi class="mjx-i" size="s"><mjx-c class="mjx-c1D441 TEX-I"></mjx-c></mjx-mi><mjx-spacer style="margin-top: 0.291em;"></mjx-spacer><mjx-texatom size="s" texclass="ORD"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D456 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c3D"></mjx-c></mjx-mo><mjx-mn class="mjx-n"><mjx-c class="mjx-c31"></mjx-c></mjx-mn></mjx-texatom></mjx-script></mjx-munderover><mjx-msub space="2"><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D453 TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em; margin-left: -0.06em;"><mjx-mi class="mjx-i" size="s"><mjx-c class="mjx-c1D456 TEX-I"></mjx-c></mjx-mi></mjx-script></mjx-msub><mjx-mo class="mjx-n"><mjx-c class="mjx-c28"></mjx-c></mjx-mo><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c29"></mjx-c></mjx-mo><mjx-msub><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D44E TEX-I"></mjx-c></mjx-mi><mjx-script style="vertical-align: -0.15em;"><mjx-mi class="mjx-i" size="s"><mjx-c class="mjx-c1D456 TEX-I"></mjx-c></mjx-mi></mjx-script></mjx-msub><mjx-mo class="mjx-n"><mjx-c class="mjx-c28"></mjx-c></mjx-mo><mjx-mi class="mjx-i"><mjx-c class="mjx-c1D461 TEX-I"></mjx-c></mjx-mi><mjx-mo class="mjx-n"><mjx-c class="mjx-c29"></mjx-c></mjx-mo></mjx-math><mjx-assistive-mml unselectable="on" display="inline"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mi>R</mi><mi>t</mi></msub><mo>≜</mo><munderover><mo data-mjx-texclass="OP">∑</mo><mrow data-mjx-texclass="ORD"><mi>i</mi><mo>=</mo><mn>1</mn></mrow><mi>N</mi></munderover><msub><mi>f</mi><mi>i</mi></msub><mo stretchy="false">(</mo><mi>t</mi><mo stretchy="false">)</mo><msub><mi>a</mi><mi>i</mi></msub><mo stretchy="false">(</mo><mi>t</mi><mo stretchy="false">)</mo></math></mjx-assistive-mml></mjx-container>
```

显然，在异常的情况中，公式根本没有经过 Mathjax 的处理，而是直接由普通的字符组成，所以字体和格式都和 $\TeX$ 公式完全不同了。

# 原因

问题的核心在于 Pandoc 的默认预处理行为。Pandoc 在将 Markdown 转为 HTML 时，如果发现公式比较简单（例如只有上下标、加减法），它会尝试直接用原生的 HTML 标签（如 `<sup>`, `<sub>`, `<strong>`）和 Unicode 字符来模拟公式。被 Pandoc “预处理”过的公式，其原本的 TeX 定界符（如 `$` 或 `$$`）会被抹除。之后，MathJax 脚本在网页加载后会扫描特定的定界符。由于 Pandoc 已经把公式变成了普通的 HTML 代码，MathJax 认为那是普通文本，从而**直接跳过**，导致公式无法被正确渲染。

只有那些 Pandoc 无法转化的复杂公式，才会被保留为原始 TeX 代码，交给 MathJax 处理。因此，一些复杂的公式反而看起来是正常的。

# 解决方案

在 `_config.yml` 中配置 Pandoc 插件，通过添加 `--mathjax` 参数，强制要求 Pandoc 保持公式原样，留给前端处理。

``` yaml
pandoc:
  args: ['--mathjax']
```

只要修改这一处即可，不需要改 Mathjax 的配置，更不需要在前端注入任何东西。

这个问题在 Gemini 的辅助下得到了解决，一开始尝试过修改 CSS 和 Inject 等操作，但是没有任何作用。本质上，在 Hexo + Pandoc + MathJax 的链条中，存在一个**渲染权**的问题：

- **Pandoc 的默认逻辑**：它认为自己是一个“全能转换器”，看到简单的 `$a^2$`，它觉得没必要动用沉重的 MathJax 脚本，直接用 `a<sup>2</sup>` 就能搞定。
- **MathJax 的逻辑**：它只负责渲染带有特定“通行证”（如 `$` 或 `\( \)`）的 TeX 代码。
- **冲突点**：由于 Pandoc 提前把简单公式“翻译”成了 HTML，MathJax 在页面上找不到“通行证”，导致这些公式沦为了普通文本，失去了 LaTeX 字体和右键菜单。

`args: ['--mathjax']` 这个选项的作用是**“剥夺 Pandoc 对数学公式的解释权”**。

- 它强制 Pandoc 在输出 HTML 时，将所有的数学公式包裹在标准的 `\( ... \)` (行内) 和 `\[ ... \]` (行间) 中。
- 这相当于给所有公式贴上了 MathJax 认可的“统一标签”。
- 既然公式都是由 MathJax 统一渲染生成的，它们自然就获得了统一的 LaTeX 字体、正确的求和符号布局以及右键菜单。

最后的最后，Gemini 指出：


> **“不要在前端去修补后端弄坏的东西。”** 
>
> 很多时候我们看到的显示异常，其根源并不在 CSS 样式表里，而是在 HTML 结构生成的那个瞬间。



