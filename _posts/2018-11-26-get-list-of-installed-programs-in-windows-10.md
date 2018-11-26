---
layout: blog
category: blog
published: true
title: Get list of installed programs in Windows 10
---
I have found to get a *complete* list of installed programs, I need to combine the `WMIC` method and the `PowerShell` method:

From `PowerShell`:

    Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table –AutoSize > C:\installed-ps.txt

From `cmd` with admin priveleges:

    wmic product get name,version > C:\installed-wmic.txt
    
You can then manually combine those results to give you a complete (I hope) list.