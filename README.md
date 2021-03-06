# vue-lazy-hydration

[![Patreon](https://img.shields.io/badge/patreon-donate-blue.svg)](https://www.patreon.com/maoberlehner)
[![Donate](https://img.shields.io/badge/Donate-PayPal-blue.svg)](https://paypal.me/maoberlehner)
[![Build Status](https://travis-ci.org/maoberlehner/vue-lazy-hydration.svg?branch=master)](https://travis-ci.org/maoberlehner/vue-lazy-hydration)
[![GitHub stars](https://img.shields.io/github/stars/maoberlehner/vue-lazy-hydration.svg?style=social&label=Star)](https://github.com/maoberlehner/vue-lazy-hydration)

> Lazy hydration of server-side rendered Vue.js components.

## WARNING: Alpha stage

This plugin is currently in an early alpha stage. Although everything seems to work fine in combination with the applications I tested it with, there might be certain cases I haven't considered. I'd be very happy if you test it with your own projects and report back if it works for you.

**Use at your own risk.**

## Motivation

`vue-lazy-hydration` is a Vue.js plugin and Nuxt.js module to **improve Estimated Input Latency and Time to Interactive** of server-side rendered Vue.js applications. This can be achieved **by using lazy hydration to delay the hydration of pre-rendered HTML**. Additionally, **code splitting is used to delay the loading of the JavaScript code of components** which are marked for lazy hydration.

## Caveats

Because of how this plugin works, **lazy loaded components will not automatically rerender when properties change**. This is by design. The basic idea is to **use this plugin for mostly static sites** or for **mostly static components** of a otherwise dynamic application.

**This plugin will not work as advertised if you're not using it in combination with SSR.** Although it should work with every pre-rendering approach (like [Prerender SPA Plugin](https://github.com/chrisvfritz/prerender-spa-plugin), [Gridsome](https://gridsome.org/), ...) I've only tested it with [Nuxt.js](https://nuxtjs.org) so far.

## Install

```bash
npm install vue-lazy-hydration
```

### Nuxt.js

Add `vue-lazy-hydration/nuxt` to modules section of `nuxt.config.js`.

```js
export default {
  // ...
  modules: ['vue-lazy-hydration/nuxt'],
  // ...
};
```

### Vue.js (with SSR)

```js
import { VueLazyHydration } from 'vue-lazy-hydration';
import Vue from 'vue';

Vue.use(VueLazyHydration);
```

### Basic example

In the example below you can see the three `load` modi in action.

1. The `ArticleContent` component is only loaded in SSR mode, which means it never gets hydrated in the browser, which also means it will never be interactive (static content only).
2. Next we can see the `AdSlider` beneath the article content, this component will most likely not be visible initially so we can delay hydration until the point it becomes visible.
3. At the very bottom of the page we want to render a `CommentForm` but because most people only read the article and don't leave a comment, we can save resources by only hydrating the component whenever it actually receives focus.

```html
<template>
  <div class="ArticlePage">
    <ArticleContent :content="article.content"/>
    <AdSlider/>
    <CommentForm :article-id="article.id"/>
  </div>
</template>

<script>
import {
  loadOnInteraction,
  loadSsrOnly,
  loadWhenVisible,
} from 'vue-lazy-hydration';

export default {
  components: {
    AdSlider: loadWhenVisible(
      () => import('./AdSlider.vue'),
      { selector: '.AdSlider' },
    ),
    ArticleContent: loadSsrOnly(() => import('./ArticleContent.vue')),
    CommentForm: loadOnInteraction(
      () => import('./CommentForm.vue'),
      {
        event: ['click', 'focusin'],
        selector: '.CommentForm',
      },
    ),
  },
  // ...
};
</script>
```

### Advanced

#### Manually load and hydrate component

Sometimes you might want to prevent a component from loading initially but you want to activate it on demand if a certain action is triggered. You can do this by manually resolving the component like you can see in the following example.

```html
<template>
  <div class="MyComponent">
    <button @click="editModeActive = true">
      Activate edit mode
    </button>
    <UserSettingsForm :editable="editModeActive"/>
  </div>
</template>

<script>
import { loadSsrOnly } from 'vue-lazy-hydration';

const ResolvableUserSettingsForm = loadSsrOnly(() => import('./UserSettingsForm.vue'));

export default {
  components: {
    UserSettingsForm: ResolvableUserSettingsForm,
  },
  data() {
    return {
      editModeActive: false,
    };
  },
  watch: {
    // As soon as the value of `editModeActive` changes the
    // `UserSettingsForm` component is loaded and hydrated
    // so it becomes fully interactive.
    editModeActive() {
      if (ResolvableUserSettingsForm.lazyHydration.resolve) {
        ResolvableUserSettingsForm.lazyHydration.resolve();
      }
    },
  },
  // ...
};
</script>
```

## Benchmarks

### Without lazy hydration

![Without lazy hydration.](https://res.cloudinary.com/maoberlehner/image/upload/c_scale,f_auto,q_auto,w_600/v1532158513/github/vue-lazy-hydration/no-lazy-hydration-demo-benchmark)

### With lazy hydration

![With lazy hydration.](https://res.cloudinary.com/maoberlehner/image/upload/c_scale,f_auto,q_auto,w_600/v1532158513/github/vue-lazy-hydration/lazy-hydration-demo-benchmark)

## Articles

- [How to Drastically Reduce Estimated Input Latency and Time to Interactive of SSR Vue.js Applications](https://markus.oberlehner.net/blog/how-to-drastically-reduce-estimated-input-latency-and-time-to-interactive-of-ssr-vue-applications/)

## About

### Author

Markus Oberlehner  
Website: https://markus.oberlehner.net  
Twitter: https://twitter.com/MaOberlehner  
PayPal.me: https://paypal.me/maoberlehner  
Patreon: https://www.patreon.com/maoberlehner

### License

MIT
