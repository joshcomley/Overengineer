---
layout: blog
category: blog
published: false
title: Get list of installed programs in Windows 10
---
Having found the `WMIC` method does not produce a complete list (for example, I couldn't see `VLC` with the `WMIC` output), this `PowerShell` script seems to do the job:

    Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table –AutoSize >installed.txt