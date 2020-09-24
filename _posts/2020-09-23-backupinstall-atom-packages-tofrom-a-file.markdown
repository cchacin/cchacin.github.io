---
layout: post
title: "🐚 Install all your Atom packages 📦 from a file"
description: Backup/Install all your Atom packages to/from a file that you can store in your dotfiles, Github, Dropbox, etc.
date: "2020-09-23 22:40:59 -0700"
lang: en-us
author: cchacin
tags: atom packages dotfiles
---

![atom](/public/images/atom-packages.png)

Every time I have to set up a new computer, I have to remember the [Atom][1]
packages that I was using before and start installing them one by one.

Recently, I discovered that Atom comes with the Atom Package Manager or [apm][2]
that allows you to install Atom Packages from the terminal:

<!-- more -->

```sh
$ apm install asciidoc-preview
```

But the same tool also allows us to list the currently installed packages:

```sh
$ apm list
```

```sh
Built-in Atom Packages (93)
├── atom-dark-syntax@0.29.1
├── atom-dark-ui@0.53.3
├── atom-light-syntax@0.29.1
├── atom-light-ui@0.46.3
├── base16-tomorrow-dark-theme@1.6.0
├── base16-tomorrow-light-theme@1.6.0
├── one-dark-ui@1.12.5
.
.
.
├── language-xml@0.35.3
└── language-yaml@0.32.0

Community Packages (10) ~/.atom/packages
├── asciidoc-assistant@0.2.3
├── asciidoc-image-helper@1.0.1
├── asciidoc-preview@2.13.1
├── atom-beautify@0.33.4
├── atom-runner@2.7.1
├── autocomplete-asciidoc@0.1.2
├── intellij-idea-keymap@0.2.3
├── jekyll@2.1.0
├── language-asciidoc@1.11.0
└── platformio-ide-terminal@2.10.0
```

The output is vast since it includes all the built-in packages plus the
installed ones. Let's filter only the installed:

```sh
$ apm list --installed
```

```sh
Community Packages (10) ~/.atom/packages
├── asciidoc-assistant@0.2.3
├── asciidoc-image-helper@1.0.1
├── asciidoc-preview@2.13.1
├── atom-beautify@0.33.4
├── atom-runner@2.7.1
├── autocomplete-asciidoc@0.1.2
├── intellij-idea-keymap@0.2.3
├── jekyll@2.1.0
├── language-asciidoc@1.11.0
└── platformio-ide-terminal@2.10.0
```

A lot better, but we will also need to remove the noise from the output:

```sh
$ apm list --installed --bare
```

```sh
asciidoc-assistant@0.2.3
asciidoc-image-helper@1.0.1
asciidoc-preview@2.13.1
atom-beautify@0.33.4
atom-runner@2.7.1
autocomplete-asciidoc@0.1.2
intellij-idea-keymap@0.2.3
jekyll@2.1.0
language-asciidoc@1.11.0
platformio-ide-terminal@2.10.0
```

If we want to remove the specific versions to tell `apm` that we want the latest
versions instead, we can filter the output with `grep`:

```sh
$ apm list --installed --bare | grep '^[^@]\+' -o
```

```sh
asciidoc-assistant
asciidoc-image-helper
asciidoc-preview
atom-beautify
atom-runner
autocomplete-asciidoc
intellij-idea-keymap
jekyll
language-asciidoc
platformio-ide-terminal
```

Now that we have a clean list of our installed packages, we can store it in a
text file:

```sh
$ apm list --installed --bare | grep '^[^@]\+' -o > atom-packages.txt
```

We can store this file in dotfiles, Github, Gists, Gitlab, Dropbox, or even
email and then run the following command to install all the packages i.e., for
new installations:

```sh
$ apm install --packages-file atom-packages.txt
```

[1]: (https://atom.io/)
[2]: (https://github.com/atom/apm)
