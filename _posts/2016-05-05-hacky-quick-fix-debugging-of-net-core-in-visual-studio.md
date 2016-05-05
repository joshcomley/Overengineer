---
layout: blog
category: blog
splash: ""
tags: ""
published: true
title: "Hacky, quick-fix debugging of .NET Core in Visual Studio"
---
In the latest .NET Core there is currently no compatible Visual Studio tooling for debugging from within VS.

This is due to a shift from DNX to CLI, and on-going work.

Anyway, that's dull, let's cut to the chase. Want some primitive debugging support to your .NET Core stuff?

## Step 1 - Install the latest CLI

<https://dotnetcli.blob.core.windows.net/dotnet/beta/Installers/Latest/dotnet-dev-win-x64.latest.exe>

## Step 2 - Install Visual Commander VS extension

Install:
<https://visualstudiogallery.msdn.microsoft.com/deda8ac1-75e6-4068-89ab-b607cee38f2d>

Info:
<https://vlasovstudio.com/visual-commander/>

## Step 3 - Import my debug and run commands

<https://www.dropbox.com/s/tq8bqp00pvja3w6/netcoredebugging.vcmd?dl=0>

The (current at the time of writing) code for the commands is below.

The code for both are identical, with the exception that the following line is commented out in the "Run" version:

{% highlight c# %}
   	// Comment out this line if you want just "Run" functionality
	proc.Attach();
{% endhighlight %}

This is the line that attaches VS debugging to the process, so without it it's just `Ctrl+F5` type "Run".

The commands expect to find `Newtonsoft.Json.dll` somewhere, notably here:
    
    C:\Program Files (x86)\Common Files\Microsoft Shared\VsHub\1.0.0.0\lib\Newtonsoft.Json.dll

You can edit the commands yourself if you want to change the location of that DLL (or anything else).

Finally, if you want to have debugging right from your entry point (e.g. `Program.cs`) you need to add this:

	public static void Main(string[] args)
	{
        args = DotNetRunner.AppStart(args);
        ...
    }

Where the `DotNetRunner.cs` looks like this:

{% highlight c# %}
public static class DotNetRunner
{
	public static string[] AppStart(params string[] args)
	{
		var newArgs = new List<string>();
		if (args.Length > 0)
		{
			var prefix = "wait:";
			foreach (var arg in args)
			{
				if (arg.StartsWith(prefix))
				{
					var wait = Int32.Parse(arg.Substring(prefix.Length));
					Thread.Sleep(wait);
				}
				else
				{
					newArgs.Add(arg);
				}
			}
		}
		return newArgs.ToArray();
	}
}
{% endhighlight %}

This is a very, VERY hacky hack hack that simply tells the app to sleep for 5 seconds, otherwise we don't attach debugging in time to jump into the entry point's code. You can adjust the 5 seconds to someting else as suited by changing the "wait:5000" in the command code to something else if you need. Yes, it's horrible, truly horrible, but until the Visual Studio CLI tooling is here, it's fine.

So, overall this is VERY primitive, but here's what it does:

- It saves all documents open in VS first, just like a normal build
- It finds the current startup project
- It tries to find the `launchSettings.json` for the current startup project and looks for a "launchUrl" specifically at the following JSON path: `\profiles\IIS Express\launchUrl`
- It looks for the exe to run as `"bin\Debug\netstandardapp1.5\win81-x64\" + projectName + ".exe"`, where project name is without any extensions, just the folder name of the project. You can modify this behaviour as needed in `GetProjectExe(..)`
- It also tries to call an `onbeforerun.bat` and an `onrun.bat` so you can inject some stuff if you want
- It calls `dotnet restore` and then `dotnet build` *every* time

## Code for the Visual Commander command(s):

{% highlight c# %}
using EnvDTE;
using EnvDTE80;
using System;
using System.IO;
using System.Windows.Forms;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public class C : VisualCommanderExt.ICommand
{
	public string ProjectPath { get; set; }

	public void Run(EnvDTE80.DTE2 DTE, Microsoft.VisualStudio.Shell.Package package)
	{
		DTE.Documents.SaveAll();
		var startupProjectFile = GetStartupProject(DTE);
		var exe = GetProjectExe(DTE, startupProjectFile);
		ProjectPath = Path.GetDirectoryName(startupProjectFile);

		TryRun("onbeforerun.bat", true);

		var projectName = Path.GetFileName(ProjectPath);
		var existingProcesses = System.Diagnostics.Process.GetProcessesByName(projectName);
		foreach (var existingProcess in existingProcesses)
		{
			existingProcess.Kill();
		}

		// Call "dotnet restore"
		DotNet("restore");

		// Call "dotnet build"
		DotNet("build");

		var process = new System.Diagnostics.Process();
		process.OutputDataReceived += (sender, args) =>
		{
			var line = args.Data;
			if (line == null)
			{
				return;
			}
			var prefix = "Now listening on: ";
			if (line.Trim().StartsWith(prefix))
			{
				var url = line.Substring(prefix.Length);
				var launchSettingsPath = Path.Combine(ProjectPath, @"properties\launchSettings.json");
				if (File.Exists(launchSettingsPath))
				{
					JObject obj = JObject.Parse(File.ReadAllText(launchSettingsPath));
					var settingsUrl = "";
					try
					{
						JToken token = obj["profiles"]["IIS Express"]["launchUrl"];
						settingsUrl = token.Value<string>();
					}
					catch (NullReferenceException e)
					{
					}
					if (!string.IsNullOrWhiteSpace(settingsUrl))
					{
						if (IsAbsoluteUrl(settingsUrl))
						{
							url = settingsUrl;
						}
						else
						{
							url = url.TrimEnd('/') + "/" + settingsUrl.TrimStart('/');
						}
					}

				}
				if (!string.IsNullOrWhiteSpace(url))
				{
					System.Diagnostics.Process.Start("explorer", url);
				}
			}
		};
		var ps = new System.Diagnostics.ProcessStartInfo(exe, "wait:5000");
		ps.RedirectStandardOutput = true;
		ps.UseShellExecute = false;
		process.EnableRaisingEvents = true;
		process.StartInfo = ps;
		process.Start();
		process.BeginOutputReadLine();

		TryRun("onrun.bat");

		foreach (Process proc in DTE.Debugger.LocalProcesses)
		{
			if (proc.ProcessID == process.Id)
			{
            	// Comment out this line if you want just "Run" functionality
				proc.Attach();
				return;
			}
		}
	}

	bool IsAbsoluteUrl(string url)
	{
		Uri result;
		return Uri.TryCreate(url, UriKind.Absolute, out result);
	}

	public void TryRun(string batch, bool waitForExit = false)
	{
		var onrun = Path.Combine(ProjectPath, batch);
		if (File.Exists(onrun))
		{
			var process = new System.Diagnostics.Process();
			var psi = new System.Diagnostics.ProcessStartInfo(onrun);
			psi.WorkingDirectory = ProjectPath;
			process.StartInfo = psi;
			process.Start();
			if (waitForExit)
			{
				process.WaitForExit();
			}
		}
	}

	public void DotNet(string command)
	{
		var process = new System.Diagnostics.Process();
		var ps = new System.Diagnostics.ProcessStartInfo("cmd", "/k call dotnet " + command + " & exit");
		ps.WorkingDirectory = ProjectPath;
		process.StartInfo = ps;
		process.Start();
		process.WaitForExit();
	}

	public string GetStartupProject(EnvDTE80.DTE2 DTE)
	{
		var solutionPath = Path.GetDirectoryName(DTE.Solution.FullName);
		var projectFile = "";
		foreach (String s in (Array)DTE.Solution.SolutionBuild.StartupProjects)
		{
			projectFile = Path.Combine(solutionPath, s);
			break;
		}
		return projectFile;
	}

	public string GetProjectExe(EnvDTE80.DTE2 DTE, string projectFile)
	{
		var projectPath = Path.GetDirectoryName(projectFile);
		var projectName = Path.GetFileName(projectPath);
		var guess = @"bin\Debug\netstandardapp1.5\win81-x64\" + projectName + ".exe";
		var finalExePath = Path.Combine(projectPath, guess);
		return finalExePath;
	}
}
{% endhighlight %}
