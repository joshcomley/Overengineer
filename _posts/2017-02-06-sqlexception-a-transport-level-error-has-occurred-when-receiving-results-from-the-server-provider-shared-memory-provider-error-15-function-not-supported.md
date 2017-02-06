---
layout: blog
category: blog
published: false
title: >-
  SqlException: A transport-level error has occurred when receiving results from
  the server. (provider: Shared Memory Provider, error: 15 - Function not
  supported)
---
This error might come after a particular SQL Server Windows Update (KB3197875 and KB3196684, listed as KB3195387 in installed updates).

A common suggestion is to install .NET 4.6.2, but for me this didn't work as the installer simply told me that this version of the framework was alrady installed.

The solution for me was to open *SQL Server Configuration Manager*:


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
