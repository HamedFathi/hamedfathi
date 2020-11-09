---
title: Write a custom plugin with Aurelia 2 and Lerna
date: November 09 2020
category: aurelia
tags:
    - aurelia2
    - plugin
    - lerna
    - bootstrap5
---

In this article, We will be able to write a simple `Bootstrap v5` plugin for `Aurelia 2` and manage our packages via `Lerna` structure.

<!-- more -->

With help of my good friend, [Sayan](https://github.com/Sayan751), I want to discuss about writing a custom plugin. At the end, you will have a good knowledge to write your own plugins, so stay tuned.

### What is Bootstrap?

> Bootstrap is the most popular CSS Framework for developing responsive and mobile-first websites.

The main purpose of this article is to create a component of [Bootstrap 5](https://v5.getbootstrap.com/) as a plugin for Aurelia 2.

### What is Lerna?

> A tool for managing JavaScript projects with multiple packages.
> Splitting up large codebases into separate independently versioned packages is extremely useful for code sharing. However, making changes across many repositories is messy and difficult to track, and testing across repositories becomes complicated very quickly.
> To solve these (and many other) problems, some projects will organize their codebases into multi-package repositories (sometimes called monorepos).

To achieve the ultimate goal of this article, we will create our project in the form of monorepos.

### What is a plugin?

> In computing, a plug-in (or plugin, add-in, addin, add-on, or addon) is a software component that adds a specific feature to an existing computer program. When a program supports plug-ins, it enables customization.

I want the user to be able to customize their requirements while using this plugin so in the following we will examine how to add config to our plugin.

### Aurelia 1 vs Aurelia 2