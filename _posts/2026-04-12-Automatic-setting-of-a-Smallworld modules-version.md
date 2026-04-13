---
title: "Automatic setting of a Smallworld module's version"
categories:
    - gis
tags:
    - gis
    - smallworld
    - git
    - development
---

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

To introduce new functionality, you modified the Magik code. Until you saved a new image, this new functionalities weren't available.

Have you ever tried to delete or overwrite a JAR file deployed to production? Chances are good that it is linked to a Smallworld session and therefore can't be deleted.

Have you ever tried to merge a git test branch into production branch during working hours? Chances are good that the production environment is critically hurt afterwards. Try changing the module's version the next time.

Do you like to merge changes from test to production after working hours or at the weekend? Try changing the module's version the next time.

If the version is different (or better: higher than before), the name of the JAR file is different and you have no problems. More than one JAR file for a module can coexist in given directory. It is only necessary that the module.def contains a version, too. Then the corresponding JAR is loaded.

(Of course you should delete old versions as soon as possible, since too many JARs can have an impact on startup performance).

## How to set a version number?

Just check which module you made changes in, open its module.def and place an arbitrary number after the module's name. It's that simple.

And it's annoying. I'm not aware of any feature in Emacs, MDT or Visual Studio Code that offers a more compelling approach and supports developers in this — in my view — increasingly important step.

## A Solution using git a pre-commit hook

<div style="background:#fff;border:1px solid #e1e4e8;border-radius:8px;padding:1.5rem 1.75rem;margin:1rem 0;font-family:sans-serif;">

  <h2 style="font-size:18px;font-weight:500;margin:0 0 1rem;padding-bottom:1rem;border-bottom:1px solid #e1e4e8;">Git hooks</h2>

  <p style="font-size:15px;line-height:1.7;margin:0 0 1rem;">Git hooks are scripts Git executes automatically at specific points in your workflow — before or after events like committing, pushing, or merging. They live in <code>.git/hooks</code> and can be written in any scripting language.</p>

  <div style="display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:1.25rem;">
    <div style="background:#f6f8fa;border-radius:6px;padding:0.75rem 1rem;">
      <p style="font-size:11px;font-weight:500;color:#57606a;margin:0 0 4px;text-transform:uppercase;letter-spacing:0.04em;">Client-side</p>
      <p style="font-size:13px;margin:0;line-height:1.6;">Run locally on the developer's machine.<br><code>pre-commit</code>, <code>pre-push</code>, <code>post-merge</code></p>
    </div>
    <div style="background:#f6f8fa;border-radius:6px;padding:0.75rem 1rem;">
      <p style="font-size:11px;font-weight:500;color:#57606a;margin:0 0 4px;text-transform:uppercase;letter-spacing:0.04em;">Server-side</p>
      <p style="font-size:13px;margin:0;line-height:1.6;">Run on the Git server, triggered by network operations.<br><code>pre-receive</code>, <code>post-receive</code></p>
    </div>
  </div>

  <div style="border:1px solid #e1e4e8;border-radius:6px;overflow:hidden;margin-bottom:1.25rem;">
    <div style="background:#f6f8fa;padding:0.6rem 1rem;border-bottom:1px solid #e1e4e8;">
      <code style="font-size:14px;font-weight:500;">pre-commit</code>
      <span style="font-size:12px;color:#57606a;margin-left:8px;">— most commonly used</span>
    </div>
    <div style="padding:0.85rem 1rem;">
      <p style="font-size:14px;line-height:1.7;margin:0 0 0.75rem;">Runs before Git asks for a commit message. Exiting with a non-zero status aborts the commit. Can be bypassed with <code>--no-verify</code>.</p>
      <p style="font-size:12px;font-weight:500;color:#57606a;margin:0 0 6px;">Typical uses</p>
      <div style="display:flex;flex-wrap:wrap;gap:6px;">
        <span style="font-size:12px;padding:3px 10px;border-radius:20px;background:#f6f8fa;color:#57606a;border:1px solid #e1e4e8;">Run linters</span>
        <span style="font-size:12px;padding:3px 10px;border-radius:20px;background:#f6f8fa;color:#57606a;border:1px solid #e1e4e8;">Run tests</span>
        <span style="font-size:12px;padding:3px 10px;border-radius:20px;background:#f6f8fa;color:#57606a;border:1px solid #e1e4e8;">Check trailing whitespace</span>
        <span style="font-size:12px;padding:3px 10px;border-radius:20px;background:#f6f8fa;color:#57606a;border:1px solid #e1e4e8;">Inspect staged snapshot</span>
      </div>
    </div>
  </div>

  <div style="padding-top:1rem;border-top:1px solid #e1e4e8;display:flex;gap:1.5rem;">
    <a href="https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks" style="font-size:13px;color:#0969da;">Git hooks guide &rarr;</a>
    <a href="https://git-scm.com/docs/githooks" style="font-size:13px;color:#0969da;">githooks reference &rarr;</a>
  </div>

</div>