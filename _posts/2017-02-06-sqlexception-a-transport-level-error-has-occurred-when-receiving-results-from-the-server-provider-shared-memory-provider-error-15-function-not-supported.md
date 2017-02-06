---
layout: blog
category: blog
published: false
title: >-
  SqlException: A transport-level error has occurred when receiving results from
  the server. (provider: Shared Memory Provider, error: 15 - Function not
  supported)
splash: ''
tags: ''
---
This error might come after a particular SQL Server Windows Update (KB3197875 and KB3196684, listed as KB3195387 in installed updates).

A common suggestion is to install .NET 4.6.2, but for me this didn't work as the installer simply told me that this version of the framework was alrady installed.

The solution for me was to open *SQL Server Configuration Manager*:

![2017-02-06 02_31_32-Cortana2.png]({{site.baseurl}}/media/2017-02-06 02_31_32-Cortana2.png =0.5x)

Then navigate to the *SQL Server Network Configuration* and disable "Shared Memory", but make sure you leave one of the others on so SQL Server can communicate **somehow**:

![2017-02-06 03_04_27-Manage your account - Hazception.png]({{site.baseurl}}/media/2017-02-06 03_04_27-Manage your account - Hazception.png)

Then just head over to your SQL Server instance and restart it:

![2017-02-06 02_57_50-Editing _posts_2017-02-06-sqlexception-a-transport-level-error-has-occurred-when.png]({{site.baseurl}}/media/2017-02-06 02_57_50-Editing _posts_2017-02-06-sqlexception-a-transport-level-error-has-occurred-when.png)
