---
layout: blog
category: blog
published: true
title: >-
  VS Code: How to set a keyboard shortcut to focus the active file in file
  explorer
---
Simple.

Add this to your `keybindings.json`:

    { "key": "shift+alt+l", "command": "workbench.files.action.focusFilesExplorer" }
    
This works just like ReSharper's `Shift+Alt+L` binding in Visual Studio.
