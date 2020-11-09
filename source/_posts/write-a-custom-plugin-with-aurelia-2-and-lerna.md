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

**Aurelia 1**

```js
// src/main(.js|.ts)

export function configure(aurelia: Aurelia): void {

aurelia.use.plugin(PLATFORM.moduleName('aurelia-toolbelt'));
// OR
// aurelia.use.plugin(PLATFORM.moduleName('aurelia-toolbelt/components/bootstrap/button'));

  aurelia.start()
         .then(() => aurelia.setRoot(PLATFORM.moduleName('app')));
}
```

**Aurelia 2**

```js
// main.ts

import Aurelia from 'aurelia';
import { App } from './app';
import * as atComponents from 'aurelia-toolbelt';

Aurelia
  .register(
    atComponents // This globalizes all the exports of our registry.
  )
  .app(App)
  .start();
```

### The structure

* core
* bootstrap-v5-core
* bootstrap-v5
* demo

### Lerna configuration

### Plugin configuration

```js
export enum Size {
    ExtraSmall = 'xs',
    Small = 'sm',
    Medium = 'md',
    Large = 'lg',
    ExtraLarge = 'xl',
}

export interface IBootstrapV5Options {
    enableRippleEffect?: boolean;
    defaultSize?: Size;
}

const defaultOptions: IBootstrapV5Options = {
    enableRippleEffect: false,
    defaultSize: Size.Medium
};

export const IBootstrapV5Options = DI.createInterface<IBootstrapV5Options>('IBootstrapV5Options').noDefault();

function createIBootstrapV5Configuration(optionsProvider: (options: IBootstrapV5Options) => void) {
    return {
        optionsProvider,
        register(container: IContainer) {
            optionsProvider(defaultOptions); // <-- this is your hook to add the customizations 
            return container.register(Registration.instance(IBootstrapV5Options, defaultOptions))
        },
        customize(cb?: (options: IBootstrapV5Options) => void) { //<-- provide delgate to the users so that they can mutate the provided (via the cb arg) options object
            return createIBootstrapV5Configuration(cb ?? optionsProvider);
        },
    };
}

// How to use?
// container.register(BootstrapV5Configuration) or container.register(BootstrapV5Configuration.customize((options) => { options.enableRippleEffect = true; })).
export const BootstrapV5Configuration = createIBootstrapV5Configuration(() => { /* This is noop, as by default you don't make any change to the default options. */ });

```