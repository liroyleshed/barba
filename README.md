# Barba v2

![stability-wip](https://img.shields.io/badge/stability-work_in_progress-lightgrey.svg?style=flat-square)
[![Coverage Status](https://img.shields.io/coveralls/github/epicagency/barba/master.svg?style=flat-square)](https://travis-ci.com/epicagency/barba)
[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg?style=flat-square)](http://commitizen.github.io/cz-cli/)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg?style=flat-square)](https://conventionalcommits.org)
[![License](https://img.shields.io/badge/license-MIT-green.svg?style=flat-square)](https://github.com/barbajs/barba-next/blob/master/LICENSE)
[![Slack channel](https://img.shields.io/badge/slack-channel-purple.svg?style=flat-square&logo=slack)](https://barbajs.slack.com)

> [Invite link to slack channel](https://join.slack.com/t/barbajs/shared_invite/enQtNTI4OTgzNDc0OTYzLTNjMjIyMzI5NWNkNmI5MzNjYTJjNWJjMThiYjIzNjY3Y2IyZTQyNzEyMTEzZWMxMDM5ZTY1ZmJjMDg5MDY1ODE)
>
__Barba__ is a small (8kb minified and gzipped) and easy-to-use library that helps you creating fluid and smooth transitions between your website's pages.

It helps reducing the delay between your pages, minimizing browser HTTP requests and enhancing your user's web experience.

> This is pre-release version. Do not use in production!<br>
> Thanks in advance for reporting bugs. #sharethelove 😊

## What's new?

- Simplified API
- Hook sytem for `transitions` and `views`
- _Transition resolution_: declare your transitions and let Barba pick the right one
- Use of `data-barba` attributes
- Sync mode
- Plugin system
  - `@barba/router`: use of routes for _transition resolution_
  - `@barba/css`: automatic addition of CSS classes
  - `@barba/prefetch`: automatic pages prefetching (and caching), based on viewport
  - `@barba/head`: update your `<head>` _(coming soon)_
  - `@barba/transition`: ready-to-use basic transitions pack (fade, slide, …) _(coming soon)_

## Main changes (TL;DR)

- Barba container and wrapper now use `data-barba` attribute
- Same for namespace via `data-barba-namespace`
- 2 main methods: `barba.init()` and `barba.use()` (for plugins)
- Transitions:
  - are plain JS objects
  - are declared via the `barba.init({ transitions })`
  - use "hooks" corresponding to animation steps
    - `leave` and `enter` hooks are required
    - required hooks can be synchronous or asynchronous (via `this.async()` or promise based)
    - all hooks receive same [`data` argument](#dataargument)
  - use "rules" to select which transition to use
    - default rules are `namespace` and `custom`
    - `@barba/router` adds `route` rule
    - they can be combined within `from` and `to` properties
- Views:
  - are plain JS objects
  - are declared via the `barba.init({ views })`
  - use a subset of animation "hooks":
    - `beforeAppear`, `afterAppear`, `beforeLeave`, `afterLeave`, `beforeEnter`, `afterEnter`
    - receive the same [`data` argument](#dataargument)
- Sync mode will start `leave` and `enter` transitions concurrently

---

## @barba/core

### Usage

```html
<div data-barba="wrapper">
  <div data-barba="container" data-barba-namespace="home">
    page content
  </div>
</div>
```

```js
import barba from '@barba/core';

barba.init(<options>);
```

### `barba.init(<options>)`

| Option       | Type     | Default          | Description                                                    |
| ------------ | -------- | ---------------- | -------------------------------------------------------------- |
| transitions  | Array    | []               | Array of `<transition>` ([see below](#transitionobject))       |
| views        | Array    | []               | Array of `<view>` ([see below](#viewobject))                   |
| debug        | Boolean  | false            | Log extra informations                                         |
| schema       | Object   | `attributSchema` | Data attributes ([see schema.js](packages/core/src/schema.js)) |
| useCache     | Boolean  | true             | Cache your pages                                               |
| usePrefetch  | Boolean  | true             | Prefetch your pages                                            |
| prevent      | Function | (optional)       | Add prevent test                                               |
| requestError | Function | (optional)       | Request error callback                                         |

### `<transition>` object

#### Hooks

> All hooks are methods and receive the same `data` object ([see below](#dataargument))

| order | name         | required | async | description                        |
| ----- | ------------ | -------- | ----- | ---------------------------------- |
| 1     | beforeAppear |          |       | before __appear__ transition       |
| 2     | __appear__   |          | __x__ | current page __appear__ transition |
| 3     | afterAppear  |          |       | before __appear__ transition       |

| order | name        | required | async | description                                               |
| ----- | ----------- | -------- | ----- | --------------------------------------------------------- |
| 1     | before      |          |       | before everything                                         |
| 2     | beforeLeave |          |       | before __leave__ transition                               |
| 3     | __leave__   | __x__    | __x__ | current page __leave__ transition                         |
| 4     | afterLeave  |          |       | after removing __current container__                      |
| 5     | beforeEnter |          |       | before __enter__ transition and adding __next container__ |
| 6     | __enter__   | __x__    | __x__ | next page __enter__ transition                            |
| 7     | afterEnter  |          |       | after removing __next container__                         |
| 8     | after       |          |       | after everything                                          |

##### `data` argument

| property       | detail                  | type/value  | description                        |
| -------------- | ----------------------- | ----------- | ---------------------------------- |
| `data.current` |                         | object      | current page related               |
|                | `container`             | HTMLElement | barba container                    |
|                | `namespace`             | string      | barba namespace                    |
|                | `url`                   | string      | url                                |
|                | `html`                  | string      | HTML of the  page                  |
|                | `route`                 | string      | route name (`@barba/router`)       |
| `data.next`    | same as `data.current`… | object      | next page related                  |
| `data.trigger` | -                       | HTMLElement | link that triggered the transition |
|                | -                       | 'popstate'  | browser back/forward               |
|                | -                       | 'barba'     | programmatic navigation            |

##### Notes

> "async" hooks can be run either synchronously or asynchronously using the common `this.async()` style ([see run-async](https://github.com/sboudrias/run-async#readme)) or returning a promise ([see examples](#examples)).
>
> If you use [`sync: true`](#sync), as __leave__ and __enter__ will be concurrent, order will differ (all before*, then enter/leave, then all after*).
>
> Depending on rules, sync mode or cache availability, some properties can be `undefined`.

#### Rules

Rules define the _transition resolution_ (selecting the right transition to use).<br>
You can combine multiple rules.

| priority | name        | type             | argument | apply when…                                           |
| -------- | ----------- | ---------------- | -------- | ----------------------------------------------------- |
| 1        | `custom`    | function         | data     | return `true`                                         |
| 2        | `route`     | string\|string[] | -        | `current.route` match ([@barba/router](#barbarouter)) |
| 3        | `namespace` | string\|string[] | -        | `current.namespace` match                             |

Any rules can be used within `from` and/or `to` properties [object].

| priority | usage           | apply when…                                 |
| -------- | --------------- | ------------------------------------------- |
| x.1      | `from` AND `to` | rule(s) match for `current` AND `next` data |
| x.2      | `to`            | rule(s) match for `next` data               |
| x.3      | `from`          | rule(s) match for `current` data            |

##### Notes (**important!!!**)

The _transition resolution_ follows this order:

1. Rule(s) priority
2. `from` and/or `to` priority
3. Declaration order

> The common/default behaviour is to start __leave__ as soon as possible.<br>
> __enter__ will be called when __leave__ ends _AND_ next page is "available" (fetched or get from the cache).<br>
> If you use some `to` logic, Barba can not predict and select the right transition until the next page is available.<br>
> This also applies when using `sync: true`.<br>
> But this does not apply if `to` is used with `route` property because "next route" is known when the click occurs…
>
> Bear with this!

Example:

```js
const transition = {
  from: {
    route: '/home',
    custom: ({ trigger }) => trigger.classList && trigger.classList.contains('cta'),
  },
  to: { namespace: '/products' },
  leave() {},
  enter() {},
}
```

#### Sync mode

| Name | Type    | Default | Description                        |
| ---- | ------- | ------- | ---------------------------------- |
| sync | Boolean | false   | __Sync__ mode ([see below](#sync)) |

When set to `true`, __leave__ and __enter__ will "play together".
This involves waiting until the next page is available (fetch or cache).

### `<view>` object

Views allow you to have some logic related to the content of a namespace.
You can see it as "lifecycle" into Barba. It is a good place to init or destroy things…
They use a subset of transition hooks and receive the same [`data` object](#dataargument).

Hooks available are:

- `beforeAppear`
- `afterAppear`
- `beforeLeave`
- `afterLeave`
- `beforeEnter`
- `afterEnter`

### debug

`console.info` about transition used.

### schema

Allows you to override data attributes.

### useCache

Allows Barba to cache your pages.

### usePrefetch

Allows Barba to prefetch your pages (on `mouseover` or `touchstart`).

### prevent

Allows you to add a custom "prevent" test.
If your function returns `true`, Barba will not be actionated.

| Argument | Property | Description     |
| -------- | -------- | --------------- |
| Object   | el       | clicked element |
|          | event    | triggered event |
|          | href     | next page href  |

### requestError

Allows you to catch request errors.
If your function returns `false`, wrong link will not be "force" triggered.

| Argument | Type                  | Description                                                               |
| -------- | --------------------- | ------------------------------------------------------------------------- |
| trigger  | HTMLElement \| string | clicked/hovered element or 'barba' or 'popstate' (see `data.trigger`)     |
| action   | string                | 'enter', 'click' (or 'prefetch' with @barba/prefetch)                     |
| url      | string                | requested URL                                                             |
| response | object                | Fetch error (with `message`) or response (with `status`, `statusText`, …) |

---

## `data-barba`

Everything related to `data-barba` attributes is defined via [`attributeSchema`](packages/core/src/schema.js).
It can be overidden with `barba.init({ schema })`.

### Prefix

Default is `data-barba`.

Usage :

- with values: `data-barba="wrapper"`
- with properties: `data-barba-prevent`

### Values

| Name      | Description       |
| --------- | ----------------- |
| wrapper   | set the wrapper   |
| container | set the container |

### Properties

| Name      | Values         | Description                               | Example                       |
| --------- | -------------- | ----------------------------------------- | ----------------------------- |
| namespace | * (custom)     | set a namespace for transitions and views | `data-barba-namespace="home"` |
| prevent   | self (default) | prevent a link of using barba             | `data-barba-prevent`          |
|           |                |                                           | `data-barba-prevent="self"`   |
|           | all            | prevent all children links                | `data-barba-prevent="all"`    |

---

## @barba/router

Barba router allows you to use __routes__ for page transitions.
It adds a __route__ rule to Barba (less important than `custom()` but more than `namespace`).

```js
import barba from '@barba/core';
import router from '@barba/router';

const routes = [{ path: '/foo', name: 'foo' }, { path: '/bar', name: 'bar' }];

barba.use(router, { routes });
barba.init(<options>);
```

### `barba.use(router, <options>)`

| Option | Type  | Default | Description                                    |
| ------ | ----- | ------- | ---------------------------------------------- |
| routes | array | []      | array of `<route>` ([see below](#routeobject)) |

### `<route>` object

| Property | Type   | Description                                         |
| -------- | ------ | --------------------------------------------------- |
| name     | string | value to be used with `transition[.from|.to].route` |
| path     | string | URL pattern                                         |

> `path` accepts dynamic segments (`/product/:id`) and regular expressions (`/:lang(fr|en)/:post`) ([see path-to-regexp](https://github.com/pillarjs/path-to-regexp))

---

## @barba/css

Barba CSS adds/remove CSS classes during transitions.
By default, it uses `.barba-xxx[-xxx]` but if you add a `name` property to your transition, it uses `.my-transition-xxx[-xxx]` CSS classes.

It is mainly inspired by [Vue.js transitions](https://vuejs.org/v2/guide/transitions.html#Transition-Classes).

CSS classes:

- `appear`
  - `.barba-appear`: __starting state__ for `appear`. Added when appear transition is triggered, removed one frame after.
  - `.barba-appear-active`: __active state__ for `appear`. Applied during the entire appearing phase. Added when appear transition is triggered, removed when transition/animation finishes.
    This class can be used to define the duration, delay and easing curve.
  - `.barba-appear-to`: __ending state__ for `appear`. Added one frame after transition is triggered (at the same time `barba-appear` is removed), removed when transition/animation finishes.
- `leave`
  - `.barba-leave`: __starting state__ for `leave`. Added when leave transition is triggered, removed one frame after.
  - `.barba-leave-active`: __active state__ for `leave`. Applied during the entire leaving phase. Added when leave transition is triggered, removed when transition/animation finishes.
    This class can be used to define the duration, delay and easing curve.
  - `.barba-leave-to`: __ending state__ for `leave`. Added one frame after transition is triggered (at the same time `barba-leave` is removed), removed when transition/animation finishes.
- `enter`
  - `.barba-enter`: __starting state__ for `enter`. Added when enter transition is triggered, removed one frame after next container is inserted.
  - `.barba-enter-active`: __active state__ for `enter`. Applied during the entire leaving phase. Added after next container is inserted, removed when transition/animation finishes.
    This class can be used to define the duration, delay and easing curve.
  - `.barba-enter-to`: __ending state__ for `enter`. Added one frame after next container is inserted (at the same time `barba-enter` is removed), removed when transition/animation finishes.

```js
import barba from '@barba/core';
import css from '@barba/css';

barba.use(css);
barba.init({ transitions: [{
  appear() {}, // Optional
  enter() {},
  leave() {},
  name: 'my-transition',
}]});
```

> You have to declare the main hooks even if they are not used.
> Other synchronous hooks, can be used…

---

## @barba/prefetch (wip)

Barba Prefetch automatically fetch and cache your pages.

It is mainly inspired by [quicklink](https://github.com/GoogleChromeLabs/quicklink).
Based on Intersection Observer (depending on your browser support, you may need a polyfill), it processes all eligible links that enter the viewport.

```js
import barba from '@barba/core';
import css from '@barba/prefetch';

barba.use(prefetch);
barba.init();
```

---

## Examples

### `leave` / `enter`

```js
const t = {
  // Async/await style
  async leave(data) {
    await asyncAnimation(data.current.container);
  },
};
```

```js
const t = {
  // Returning a promise
  leave(data) {
    return asyncAnimation(data.current.container);
  },
};
```

```js
const t = {
  // Returning a promise
  leave(data) {
    return new Promise(resolve => {
      callbackAnimation(data.current.container, {
        onComplete: () => {
          resolve();
        },
      });
    });
  },
};
```

```js
const t = {
  // `this.async()` style
  leave(data) {
    const done = this.async();
    callbackAnimation(data.current.container, {
      onComplete: () => {
        done();
      },
    });
  },
};
```

### `data` argument (`data.current`, `data.next`, `data.trigger`)

```js
const t = {
  // ES6 syntax: `{ current } = data.current`
  leave({ current }) {
    // Also available: current.[namespace, url, route, html…]
    return asyncAnimation(current.container);
  },
  enter({ next }) {
    // Also available: next.[namespace, url, route, html…]
    return asyncAnimation(next.container);
  },
};
```

### `namespace` (idem for `route`)

```js
// If `current.namespace` is "home".
const t = {
  namespace: 'home',
};
```

```js
// If `current.namespace` is "products" or "services".
const t = {
  namespace: ['products', 'services'],
};
```

### `custom(data)`

```js
// If `trigger` element has "barba-FTW" CSS class.
const t = {
  custom({ trigger }) {
    return trigger.classList && trigger.classList.contains('barba-FTW');
  },
};
```

### `from` / `to`

```js
// If `current.namespace` is "home" or "about" AND `next.namespace` is "contact"
const t = {
  from: { namespace: ['home', 'about'] },
  to: { namespace: 'contact' },
};
```

```js
// If `trigger` has [data-cta="product"] AND `current.route` is "home" AND `next.route` is "product-single"
const t = {
  custom: ({ trigger }) => trigger.dataset && trigger.dataset.cta === 'product',
  from: { route: 'home' },
  to: { route: 'product-single' },
};
```

---

## How to contribute

[Follow those instructions](CONTRIBUTING.md)

## Next steps

- [ ] Testing, debugging, fixing, testing…
- [ ] e2e testing suite
- [ ] CI setup (PR, publish, …)
- [ ] Write manual documentation
- [ ] Generate code documentation
- [ ] New website
