---
title: Enabling .jsx files in React Native
layout: post
date: 2017-06-16 01:00:00
---

React Native is a really cool developing technology, but it forces some strange defaults on its users.
One such choice is the [refusal to support the `.jsx` extension out of the box](https://github.com/facebook/react-native/issues/2303).
While Babel (which is also required by React Native) has no problem handling JSX syntax in `.js` files, many style guides (including [AirBnB's linter rules](https://github.com/airbnb/javascript/issues/982)) require JSX syntax to be isolated to `.jsx` files.

Although the core React Native team doesn't show any signs of reversing their stance, it is possible in recent versions to extend the list of permissible file extensions.
I haven't been able to find any mention of it in the official documentation, but React Native allows customizing [several parts of the packager pipeline](https://github.com/facebook/react-native/blob/29d9c35e122861d86c70b1e27a6e34eda4d82369/local-cli/util/Config.js#L145-L165), including the list of acceptable file extensions for source files.

Configuration options are specified in much the same way as in webpack: you create a Javascript module the exports an object containing all of the fields that you want to override.
It _appears_ that there are two ways to provide this file to React Native, but one of them is a trap.

## How Not To Do It

The React Native scripts provide a `--config` option, so you can supply any file as the configuration module.
However, it has two major flaws.

The first is that the scripts take this value as provided on the commandline and pass it directly to a call to `require` deep in the guts of React Native.
This means that your path must either be relative to wherever the `require` call resides (which is naturally subject to change), or it has to be an absolute path.
Yuck.

The other limitation shows up when you try to run your app in the iOS simulator (and presumably the Android simulator, but I haven't tested this).
When you run `yarn ios` (or `npm run ios`) in a project that includes native components, the React Native script builds your application in one Terminal window while spawning another Terminal process to handle Javascript bundling—and it doesn't forward your configuration file to the child location.

This means that if you want reliable configuration overrides, you have to rely on magic.

## Magic

React Native's configuration loader will search all of its ancestor directories for a file called `rn-cli.config.js`, and it will load the one closest to the root directory.
This means that if you create a file with that name at your project root, it will be loaded and used automatically, no matter which React Native script you're using.
~~It also means that if you want to be really evil to one of your coworkers who left a machine unlocked, you can run `touch /rn-cli.config.js` to completely break any projects using a custom configuration.~~

So how do we use this knowledge to enable `.jsx` files?
We just add this config:

<script src="https://gist.github.com/db396f354a9626abac78ee1d95c74a9f.js"></script>

<noscript>If you had Javascript enabled, you'd see a <a href='https://gist.github.com/db396f354a9626abac78ee1d95c74a9f.js'>gist</a> here.</noscript>

Now you can use `.jsx` files with abandon!
