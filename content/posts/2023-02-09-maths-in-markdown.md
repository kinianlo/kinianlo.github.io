---
layout: post
title: Writing LaTex maths in markdown
date: 2023-02-09 23:10:42
description: This article records the pain I went through to write maths in markdown
tags: 
    - latex
    - markdown
    - maths
categories: blogging
comment: false
---

Since I started blogging, I have been using markdown to write my posts. Due to the nature of the topics I write about, I often need to write maths using LaTex.

Initially, I had been using [kramdown](kramdown.gettalong.org) as the markdown parser since it has a built-in math syntax. Essentially, kramdown recognises anything in between `$$` and `$$` as LaTex code and not markdown. That means `1,y` in `$$(x_1, y_1)$$` is not interpreted as italics or bold.
If you put the maths within a paragraph, then the maths is considered inline. If you put the maths in its own paragraph, then the maths is considered display.

It was all fine until I decided to use the built-in markdown previewer in vscode, which uses [CommonMark](https://commonmark.org/) as its markdown parser (or rather its markdown standard). Yes, there are (too) many markdown standards out there.
The problem is that in CommonMark, anything in between two `$$`'s is interpreted as a display math block, regardless of whether it is in its own paragraph or not. If you need an inline math block, you need to use a single `$` instead.
kramdown refuses to use a single `$` for inline maths for it is not unusual to have a pair of dollar signs in a sentence, e.g. "The price is $10 a discount of $2.".

Luckily, there is a vscode extension called [Markdown + Math](https://marketplace.visualstudio.com/items?itemName=goessner.mdmath) that allows you to use kramdown's math syntax in the vscode markdown previewer.

#### How does LaTex maths usually work in markdown?
First of all, maths support is almost always considered an extension to basically every major markdown standard. 
For example, there is not a single mention of the word "math" in the [CommonMark Spec](https://spec.commonmark.org) (version 0.30) nor the [GitHub Flavored Markdown Spec](https://github.github.com/gfm/) (version 0.29-gfm).
That makes every markdown parser that follows these standards not recognise any maths syntax by default.
So the first step to getting maths working in markdown is to make your markdown parser recognises a certain maths syntax, usually in the form of delimiters. So that the parser would not interpret the LaTex code as markdown and perform rendering on the maths.


#### List of markdown libraries
 - [kramdown](https://kramdown.gettalong.org) is the markdown parser used by Jekyll as default. The double dollar signs syntax is used for both inline and display maths. There is no obvious way to change the maths delimiters.
 That said, kramdown does not do any LaTex parsing by itself. 
 By default, it just changes the maths delimiters to those recognised by [MathJax](https://www.mathjax.org/), which is a strictly client-side parser.
 However, you can configure kramdown to use other maths engines. For example, you can use [KaTex](https://katex.org/) which allows rendering LaTex to MathML both on the server side and the client side.
 - [MathJax](https://www.mathjax.org/) is a javascript library that renders LaTex code in the browser. It is solely a client-side parser. In MathJax version 3, the delimiters recognised are `\[` and `\]` for display maths and `\(` and `\)` for inline maths. In MathJax version 2, the delimiters are HTML tags `<script type="math/tex">` and `<script type="math/tex; mode=display">` for display maths and `<script type="math/tex; mode=inline">` for inline maths.
 _Note: I have been encountering rendering issues with MathJax. Sometimes I would get a rectangular block of white covering part of the rendered maths._
 - [KaTex](https://katex.org/) is a fast math typesetting library for the web. The core of KaTex is used to render LaTex code with web technologies, i.e. javascript and css. Markdown is not involved at all. However, there is an extension called [Auto-render](https://katex.org/docs/autorender.html) of KaTex that searches for maths in any text and renders it. You can choose what delimiters to use for maths.
 - [markdown-it](https://markdown-it.github.io) is a markdown parser following the CommonMark Spec. There is no maths support by itself, which is a good thing, believe it or not. There is a plugin called [markdown-it-texmath](https://github.com/goessner/markdown-it-texmath) that allows configuring the maths delimiters. The default delimiters are `$$` for display maths and `$` for inline maths. Importantly, it supports the use of kramdown-style maths delimiters.
 

#### Test area
$$ $$
$$a + b$$ $$a _ b$$.
$$ \lambda * 5 $$
$ \lambda * 5 $
$ a+b $
This $$1 $$ $$ 2$$.

$$
1 + 2
$$

This is a $ a + b $ c $ 
$$
a + b
$$