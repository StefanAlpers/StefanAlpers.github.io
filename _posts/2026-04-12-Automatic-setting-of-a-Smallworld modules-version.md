---
title: "Automatic setting of a Smallworld module's version"
categories:
    - gis
tags:
    - gis
    - smallworld
    - git
    - development
    - devops
    - module
---
- [What is a Smallworld module?](#what-is-a-smallworld-module)
- [Module versions in the past](#module-versions-in-the-past)
- [What has changed?](#what-has-changed)
- [Consequences for deployments](#consequences-for-deployments)
- [How to set a version number?](#how-to-set-a-version-number)
- [A solution using a git pre-commit hook](#a-solution-using-a-git-pre-commit-hook)

## What is a Smallworld module?

A Smallworld module is a collection of Magik code and resources like XML, messages, png. It can be loaded into a Smallworld GIS session.

The minimum a Smallworld module needs is a file called module.def. This defines the module's name and its requirements (other modules) and can contain a description.

```
some_gui

description
    A gui for something
end

requires
    some_logic
end
```

And of course it can contain the version of a module. In this case, no version is given explicitly and therefore it is 1.

## Module versions in the past

Prior to Smallworld GIS 5 setting or even updating a module's version had no real benefit or impact. And honestly, nobody — not even Smallworld itself — cared.

Even as I write this, I can't imagine a use case or realistic scenario for two coexisting module versions in a Smallworld GIS environment.

## What has changed?

Smallworld GIS 5 introduced the possibility, sorry, necessity to compile Magik code into Java byte code stored in JAR files. The naming convention for these files is

product_name.module_name.VERSION(!!!).JAR

## Consequences for deployments

Prior to Smallworld GIS 5, Magik code was compiled into RAM and then a so-called image, a memory dump, was saved. During startup the image unfolded into RAM and had no connection to Magik files.

To introduce new functionality, you modified the Magik code. Until you saved a new image, this new functionality wasn't available.

Have you ever tried to delete or overwrite a JAR file deployed to production? Chances are good that it is linked to a Smallworld session and therefore can't be deleted.

Have you ever tried to merge a git test branch into production branch during working hours? There is a good chance the production environment will be seriously affected. Try changing the module's version the next time.

Do you like to merge changes from test to production after working hours or at the weekend? Try changing the module's version the next time.

If the version is different (or better: higher than before), the name of the JAR file is different and you have no problems. More than one JAR file for a module can coexist in given directory. It is only necessary that the module.def contains a version, too. Then the corresponding JAR is loaded.

(Of course you should delete old versions as soon as possible, since too many JARs can have an impact on startup performance).

## How to set a version number?

Just check which module you made changes in, open its module.def and place an arbitrary number after the module's name. It's that simple.

And it's annoying. I'm not aware of any feature in Emacs, MDT or Visual Studio Code that offers a more compelling approach and supports developers in this — in my view — increasingly important step.

## A solution using a git pre-commit hook

Since in my opinion, working with git is a must, even in a Smallworld GIS environment, I figured out an adequate solution using a git pre-commit hook.

**What are Git hooks?**
Scripts Git executes automatically at specific points in your workflow — before or after committing, pushing, or merging. They live in `.git/hooks` and can be written in any scripting language.

**Client-side:** pre-commit, pre-push, post-merge · **Server-side:** pre-receive, post-receive

**pre-commit** runs before Git asks for a commit message. Exiting non-zero aborts the commit. Bypass with `--no-verify`. Common uses: run linters, check trailing whitespace, inspect staged snapshot.

Further reading: [Git hooks guide](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) · [githooks reference](https://git-scm.com/docs/githooks)
{: .notice--info}

The pre-commit hook checks all staged files and determines the Smallworld modules they belong to. The module's version is then set using the actual date in the format "yyyyMMdd" and changed module.def are added to the commit.

The hook is available [here](https://github.com/stefanalpers/smallworld-git-hooks). It is quite new and has so far only been tested by me. Feel free to use it and I'd welcome your feedback.

Don't forget to recompile after updating the version number in module.def to keep it in sync with the JAR file name. That step could perhaps be automated too — using a post-receive hook?