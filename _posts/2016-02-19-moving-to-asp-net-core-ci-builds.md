---
layout: blog
category: blog
splash: ""
tags: null
published: false
title: Moving to ASP.NET Core CI builds
---

# Environment variable:
DNX_UNSTABLE_FEED
https://www.myget.org/F/aspnetcidev/api/v2

<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="AspNetCI" value="https://www.myget.org/F/aspnetcidev/api/v3/index.json" />
    <add key="NuGet.org" value="https://api.nuget.org/v3/index.json" />
    <add key="dotnet-core" value="https://www.myget.org/F/dotnet-core/api/v3/index.json" />
    <add key="dotnet-cli" value="https://www.myget.org/F/dotnet-cli/api/v3/index.json" />
    <add key="AspNetvNext" value="https://www.myget.org/F/aspnetvnext/api/v3/index.json" />
  </packageSources>
</configuration>