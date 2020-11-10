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

In this article, We will be able to write a simple `Bootstrap 5` plugin for `Aurelia 2` and manage our packages via `Lerna`.

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

To config your monorepos, you should do as following:

Install `Lerna` as a global tool.

```bash
npm i lerna -g
```

Go to a folder that you want to make project and run

```bash
lerna init
```

The result should contains 

* `packages`: The folder you will create your repositories there.
* `lerna.json`: Config your lerna in this file.
* `package.json`: Config your node project in this file.

Open your `packages` folder and install the projects inside it.

```bash
npx makes aurelia core -s typescript
npx makes aurelia bootstrap-v5-core -s typescript
npx makes aurelia bootstrap-v5 -s typescript
npx makes aurelia demo -s typescript
``` 

To continue we need to config `Lerna`, Open your `lerna.json` and paste the followimg code:

```json
{
  "version": "0.1.0",
  "npmClient": "npm",
  "command": {
    "bootstrap": {
      "hoist": "**"
    }
  },
  "packages": ["packages/*"]
}
```





```js
// bootstrap-v5/package.json
"dependencies": {	
    "aurelia": "dev",
    "bootstrap": "^5.0.0-alpha1",	
    "@aurelia-toolbelt/bootstrap-v5-core": "0.1.0",
    "@aurelia-toolbelt/core": "0.1.0"
},

// demo/package.json
"dependencies": {	
    "aurelia": "dev",	
    "@aurelia-toolbelt/bootstrap-v5": "0.1.0",
},
```

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