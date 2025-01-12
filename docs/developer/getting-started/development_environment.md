# Development Environment

This is part of the developer [getting started guide](./README.md).

## Stack

See [README#what-is-it](../../../README.md#what-is-it).

A good base knowledge of Vue, Vuex and Nuxt should be reached before going through the code. Looking through `nuxt.config.js` is a good way to understand how the Dashboard is glued together, importantly how plugins are brought in and how the frontend proxies requests to Rancher's APIs.


### Helpful links
Description | Link
-----| ---
Core Vue Docs | https://vuejs.org/v2/guide
Vue Template/Directive Shorthands | https://vuejs.org/v2/guide/syntax.html
Vue Conditional rendering | https://vuejs.org/v2/guide/conditional.html
Vuex Core Docs | https://vuex.vuejs.org/
Nuxt Get Started | https://nuxtjs.org/docs/2.x/get-started/installation
Nuxt Structure | https://nuxtjs.org/docs/2.x/directory-structure
Axios (HTTP Requests) | https://axios.nuxtjs.org/options
HTTP Proxy middleware | https://github.com/nuxt-community/proxy-module (https://github.com/chimurai/http-proxy-middleware)

## Platform

The Dashboard is shipped with the Rancher package which contains the Rancher API. When developing locally the Dashboard must point to an instance of the Rancher API.

### Installing Rancher
See https://rancher.com/docs/rancher/v2.x/en/installation/. This covers two methods confirmed to work with the Dashboard
- [Single Docker Container](https://rancher.com/docs/rancher/v2.x/en/installation/other-installation-methods/single-node-docker/)
- [Kube Cluster (via Helm)](https://rancher.com/docs/rancher/v2.x/en/installation/install-rancher-on-k8s/)

Also for consideration
- [RKE in a binary (rancherd)](https://rancher.com/docs/rancher/v2.x/en/installation/install-rancher-on-linux/)

You should be able to reach the older Ember UI by navigating to the Rancher API url. This same API Url will be used later when starting up the Dashboard.

### Uninstalling Rancher
- Docker - This should be a simple `docker stop` & `docker rm`
- Kube Cluster -  Use `helm delete` as usual and then the `remove` command from [System Tools](https://rancher.com/docs/rancher/v2.x/en/system-tools/) client 


## Environment

Developers are free to use the IDE and modern browser of their choosing. Here's some tips on some in particular

### VS Code
- Install the `vetur` extension. This contains syntax highlighting, IntelliSense, snippets, formatting, etc)

### Chrome
- Install the Chrome `vue-devtools` extension to view the Vuex store.
  
  > This can consume a lot of the host's resources. It's recommended to pause Vuex history (nav to Vue tab in DevTools and toggle the `Recording` button top right of the history section)

## Running / Debugging Dashboard

### Running the Dashboard

See the [Running For Development](../../../README.md#running-for-development) section on how to bring up the Dashboard locally

> Troubleshooting: Multiple `Could not freeze errors` in `yarn dev` terminal
>
> This is most probably due to a correct cache in `/node_modules/.cache`. Exit out of `yarn run` and run `yarn run clean` and then try again.

### Debugging the Dashboard

#### SSR vs SPA
It's important to understand the difference between SSR and SPA modes described in the [Server-Side-Rendering (SSR)](../../../README.md#server-side-rendering-ssr) section. When running in the default SSR mode you will not be able to step through some methods such as Vue component's `async fetch`. It is therefore advised to switch to SPA mode before attempting to step through the code (see linked guide.. either start with `--spa` or load page with url parm `?spa` / `&spa`).

SSR adds additional complexity for the store and some component functions. More on this can be found in the `Resource` section of Development guide.

#### Breakpoints
Finding the correct file in Dev Tools and reliably setting a breakpoint can be hit and miss, even in SPA mode. It is advised to manually add a `debugger` statement in code instead. 

#### Examining the contents of a Resource
Due to the way Dashboard resources are constructed examining the contents of one can sometimes provide unexpected results. It's recommended to read the sections covering resource proxy and resource instance before continuing.

- When viewing the object via template `{{ resource }}` the `resource-instance.js` `toString` method will print out a basic interpretation
- When printing the object via console the resource's [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy) will be displayed. To make this simpler to view use `JSON.parse(JSON.stringify(<resource>))`.

> Q Why are my resource's `nameDisplay`, `description`, etc missing?
>
> A These are part of the common underlying `resource-instance.js` or, if the resource type has it, the type's own `model`.

### Exploring the API
The API serves up an interface to [browse both Norman and Steve APIs](https://github.com/rancher/api-ui). Both will list supported schemas and allow the user to fetch individual or collections of resources. The schemas will describe the actions executable against individual or collections of resource. For Norman it will also show fields that can be filtered on.

The dashboard will proxy requests to the API, so the interfaces are available via `<Dashboard URL>/v3` (Norman) and `<Dashboard URL>/v1` (Steve)


