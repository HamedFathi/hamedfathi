---
title: Write a custom plugin with Aurelia 2 and Lerna
date: November 09 2020
category: aurelia
tags:
    - aurelia2
    - plugin
    - lerna
    - bootstrap5
    - monorepository
---

In this article, We will be able to write a simple [Bootstrap 5](https://v5.getbootstrap.com/) plugin for `Aurelia 2` and manage our packages via [Lerna](https://lerna.js.org/).

<!-- more -->

With help of my good friend, [Sayan](https://github.com/Sayan751), I want to discuss about writing a custom plugin. At the end, you will have a good knowledge to write your own plugins, so stay tuned.

### What is Bootstrap?

> Bootstrap is the most popular CSS Framework for developing responsive and mobile-first websites.

The main purpose of this article is to create a component of `Bootstrap 5` as a plugin for Aurelia 2.

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

We want to separate our plugin in three packages.

* **bootstrap-v5-core**

We will add the Bootstrap 5 confiurations in this package.

* **bootstrap-v5**

Our Bootstrap 5 components will define in this package. `bootstrap-v5` depends on `core` and `bootstrap-v5-core` packages.

* **demo**

We will use our plugin in this package as a demo. `demo` depends on `bootstrap-v5`.

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

The result should contain

* `packages`: The folder you will create your repositories there.
* `lerna.json`: Lerna's configuration file.
* `package.json`: Node's configuration file.

Open your `packages` folder and install the projects inside it.

```bash
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

**version**: the current version of the repository.

**npmClient**: an option to specify a specific client to run commands with (this can also be specified on a per command basis). Change to "yarn" to run all commands with yarn. Defaults to "npm".

**command.bootstrap.hoist**: Common dependencies will be installed only to the top-level `node_modules`, and omitted from individual package `node_modules`.

**packages**: Array of globs to use as package locations.

## Dependencies

As described in the structure section defined packages depend on each other. So, we link them together and add the other prerequisites for each.

* **bootstrap-v5-core**

This package has no dependency.

* **bootstrap-v5**

Go to `package.json` and add the following dependencies:

```js
// bootstrap-v5/package.json
"dependencies": {	
    "aurelia": "dev",
    "bootstrap": "^5.0.0-alpha2",	
    "@aurelia-toolbelt/bootstrap-v5-core": "0.1.0"
},
```

* **demo**

Go to `package.json` and add the following dependencies:

```js
// demo/package.json
"dependencies": {	
    "aurelia": "dev",	
    "@aurelia-toolbelt/bootstrap-v5": "0.1.0",
},
```

**Note**: To refer to the `Lerna` packages you should use package name with an `@` at the start.

**Note**: All created packages have `0.1.0` version so pay attention if the version changes, update it in other packages.

Finally, run the blow command to install packages.

```bash
lerna bootstrap
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
    defaultSize?: Size;
}

const defaultOptions: IBootstrapV5Options = {
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

### Plugin implementation



### Plugin usage


