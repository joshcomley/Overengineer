---
layout: blog
category: blog
published: false
title: Fix "style/TextAppearance.Compat.Notification.Info" missing
---
I found suddenly I was unable to compile Android (using NativeScript), and was receiving a number of errors similar to this:

    "style/TextAppearance.Compat.Notification.Info" missing
    
#Solution

Delete `C:\Users\[username]\.gradle\caches`. Rebuild. Happy days (at least for me).