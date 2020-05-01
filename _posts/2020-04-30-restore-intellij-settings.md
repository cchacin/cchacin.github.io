---
layout: post
title:  🔌 Restore IntelliJ Idea settings ⛽
description: After several installations and reinstallations of IDEs and different Java/JDK versions (8, 9, 11, 14), the IDE was not able to import my maven projects. The IntelliJ IDEA was acting as a simple text editor at this time 😟.
lang: en-us
tags: java ide IntelliJ idea
cover_image: https://carloschac.in/public/images/intellij-restore-settings.png
---

![intellij-restore-settings](https://carloschac.in/public/images/intellij-restore-settings.png)

After a long day trying to figure out how to reset all my IntelliJ settings, I decided to write this to document the obvious solution that was not that obvious to me.

I was using the 🔥 JetBrains Toolbox App 🔥 for a while to manage my IntelliJ Idea Ultimate installation along with other tools like Rider, WebStorm, and the Early Access Preview for IntelliJ Community Edition.

![toolboxapp](https://carloschac.in/public/images/toolbox-app.png)

After several installations and reinstallations of IDEs and different Java/JDK versions (8, 9, 11, 14), the IDE was not able to import my maven projects. The IntelliJ IDEA was acting as a simple text editor at this time 😟.

<!-- more -->

## 💀 Failed attempt to solve it 🔇

1) Uninstall IntelliJ IDEA Ultimate using the Toolbox App
2) Uninstall the Toolbox App
3) Delete the following directories:

```bash
$ rm -rf ~/Library/Application\ Support/JetBrains/
```

```bash
$ rm -rf ~/Library/Caches/JetBrains/IntelliJIdea2020.1
```
4) Reinstall IntelliJ IDEA without using Toolbox
    
```bash
$ brew cask install intellij
```

## 💊 The obvious and simple solution 🍏

![intellij-restore-settings-gif](https://carloschac.in/public/images/intellij-restore-settings.gif)

Go to:

`Configure` -> `Restore Default Settings` -> `Restore and Restart`
