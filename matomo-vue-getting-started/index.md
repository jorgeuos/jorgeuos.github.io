# Getting started with Vue in Matomo


# Getting started with Vue in Matomo

At the time of writing, developing with Vue is not so straight forward. The documentation online is very scarse. But I managed to dig out some of it and listed it here:

* Matomo getting started with Vue => [Getting started with Vue](https://developer.matomo.org/guides/vue-getting-started)
* Matomo JavaScript and CSS => [Working with Piwiks UI](https://developer.matomo.org/guides/working-with-piwiks-ui)
* Matomo Vue => [Vue: In Depth](https://developer.matomo.org/guides/in-depth-vue)
* Vue PoC => [POC: Vue.js 3 migration path #17805](https://github.com/matomo-org/matomo/pull/17805)
* Initial commit => [[Vue] Introduce Vue + Workflow commands #17940](https://github.com/matomo-org/matomo/pull/17940)
* Description on assets pipeline => [[Vue] Documentation on the asset pipeline and how Vue/TypeScript are handled #541](https://github.com/matomo-org/developer-documentation/pull/541)


## Building

Matomo Core team has wrapped traditional Vue building scripts into PHP to run with the `./console build:vue` command.

The initial [commit](https://github.com/matomo-org/matomo/pull/17940) into Matomo can give you clues on how they adapted Vue. Which also provided somewhat [documentation](https://github.com/matomo-org/developer-documentation/pull/541).

This post will describe my learnings and I will use some abstractions to simplify explanantions.

## Ok, here's some learnings:

(Yes, I just used the word [learnings](https://en.wiktionary.org/wiki/learnings).)

> Every new Vue component has an angularjs directive adapter that initializes it. This is done for backwards compatibility and as a formal layer for AngularJS to communicate with Vue.

This means that for every `whatevah.vue` file you create, there needs to be a `whatevah.adapter.ts` file, not always, but if you want full potential.

## Directory pattern:
```
vue
└── vue/src
    ├── vue/src/InvalidatedComponent
    │   ├── vue/src/InvalidatedComponent/InvalidatedComponent.adapter.ts
    │   └── vue/src/InvalidatedComponent/InvalidatedComponent.vue
    ├── vue/src/VueComponentDataTable
    │   ├── vue/src/VueComponentDataTable/VueComponentDataTable.adapter.ts
    │   └── vue/src/VueComponentDataTable/VueComponentDataTable.vue
    └── vue/src/index.ts
```

## Setting up your environment

### Description

Some steps for making your life easier as a frontender in Matomo.


### Check out some of our Plugins using Vue

* No public plugins yet.

### Vue folder structure:
```
./plugin/CoolVue/vue/
./plugin/CoolVue/vue/src/
./plugin/CoolVue/vue/src/index.ts
./plugin/CoolVue/vue/src/NameComponent/Namecomponent.vue
./plugin/CoolVue/vue/src/NameComponent/Namecomponent.adater.vue
```

### Check Pre-requisites:
Find recommended `node --version` in:
```s
cat ./plugins/CoreVue/Commands/Build.php | grep 'RECOMMENDED_.*='
    const RECOMMENDED_NODE_VERSION = '16.0.0';
    const RECOMMENDED_NPM_VERSION = '7.0.0';
```

### Here's an easy way to get the correct Node version running
```s
cat ./plugins/CoreVue/Commands/Build.php | grep 'RECOMMENDED_NODE_VERSION.*=' | grep -o "'.*'" | sed "s/'//g" | sed "s/^/v/g" > .nvmrc
nvm use
```

### Run build

The basic way, inside docker:

```sh
./console development:enable
./console vue:build CoolVue
```

If it's nagging about browserlist, you could update the db. But it will nag after every rebuilt of your container.

### Update npx:db

```
npx browserslist@latest --update-db
```

### Webpack

Webpack version is a mystery. If you have it installed globally, it might interfer with your build, if you're outside Docker.

------

## Advanced way 🤓
---------------

The fun stuff:

### Compiling assets outside of docker(faster, harder, SCOOOOOOTER!!! 🚀 🚀 🚀 )

For production files:
```bash
cd ./src/latest/
cat ./plugins/CoreVue/Commands/Build.php | grep 'RECOMMENDED_NODE_VERSION.*=' | grep -o "'.*'" | sed "s/'//g" | sed "s/^/v/g" > .nvmrc
nvm use
FORCE_COLOR=1 MATOMO_CURRENT_PLUGIN=CoolVue ./node_modules/@vue/cli-service/bin/vue-cli-service.js build --target lib --name CoolVue ./plugins/CoolVue/vue/src/index.ts --dest ./plugins/CoolVue/vue/dist
```

### If you wanna keep Vue watch alive

Just add `--watch`, like so:
```bash
FORCE_COLOR=1 MATOMO_CURRENT_PLUGIN=CoolVue ./node_modules/@vue/cli-service/bin/vue-cli-service.js build --watch --target lib --name CoolVue ./plugins/CoolVue/vue/src/index.ts --dest ./plugins/CoolVue/vue/dist
```

For development files:
```bash
FORCE_COLOR=1 MATOMO_CURRENT_PLUGIN=CoolVue ./node_modules/@vue/cli-service/bin/vue-cli-service.js build --mode=development --target lib --name CoolVue --filename=CoolVue.development --no-clean ./plugins/CoolVue/vue/src/index.ts --dest ./plugins/CoolVue/vue/dist --watch
```

