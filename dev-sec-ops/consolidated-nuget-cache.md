# Save drive space by using a single global shared nuget cache

> **TLDR;** set the environment variables mentioned in **section 2.2**, and optionally set the package source to only come from Azure Artifacts mentioned in **section 3.2**.

## 1. Migrate projects to Package References

In the Solution Explorer you can switch to the folder view (button top left of the panel) and search for *packages* to finde any still in use **packages.config**.



packages.config example :

```xml
<packages>
  <package id="Antlr" version="3.5.0.2" targetFramework="net461" />
  ...
  <package id="Microsoft.AspNet.Mvc" version="5.2.3" targetFramework="net461" />
  ...
</packages>
```

PackageReference example inside the project file :

```xml
<Project>
  ...
  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="2.2.6" />
    <PackageReference Include="Neo4j.Driver" Version="5.13.0" />
  </ItemGroup>
  ...
</Project>
```

## 2. Move the NuGet **global-packages** and **cache** to a shared location, on the D: drive

### What NuGet settings are you using?
```cli
# Display locals for all folders: global-packages, http cache, temp and plugins cache
dotnet nuget locals all -l
```

### 2.1. Clear the local, **per user**, global packages and cache

Recommend if Visual Studio is running, that a project or solution is not open.

Take action one of two ways:

1. Either through the IDE (Visual Studio)

> Starting in Visual Studio 2017, use the **Tools > NuGet Package Manager > Package Manager Settings** menu command, then select **Clear All NuGet Cache(s)**
>
> Any packages used by projects that are currently open in Visual Studio are not cleared from the global-packages folder.

2. Or through the CLI

```cli
# Clear the 3.x+ cache (use either command)
dotnet nuget locals http-cache --clear
nuget locals http-cache -clear

# Clear the 2.x cache (NuGet CLI 3.5 and earlier only)
nuget locals packages-cache -clear

# Clear the global packages folder (use either command)
dotnet nuget locals global-packages --clear
nuget locals global-packages -clear

# Clear the temporary cache (use either command)
dotnet nuget locals temp --clear
nuget locals temp -clear

# Clear the plugins cache (use either command)
dotnet nuget locals plugins-cache --clear
nuget locals plugins-cache -clear

# Clear all caches (use either command)
dotnet nuget locals all --clear
nuget locals all -clear
```

**Important :** It's not explicitly stated, but assumed these commands are per user, one user running it does not clear every users folder. That may be easier to do via a powershell script.

### 2.2 Use environment variables to override per user settings

| setting | user location | environment variable | proposed value | comments |
|---|---|---|---|---|
| **global-packages** | `%userprofile%\.nuget\packages` | NUGET_PACKAGES | `D:\_DevEnv\.nuget\packages` | |
| **http-cache** (Nuget 3.5.1+) | `%localappdata%\NuGet\v3-cache` | NUGET_HTTP_CACHE_PATH | `D:\_DevEnv\Nuget\v3-cache` | Expires after 30 minutes |
| **packages-cache** (Nuget prior to 3.5.1) | `%localappdata%\NuGet\Cache` | None, upgrade **Nuget.exe** | None, upgrade `Nuget.exe` | Easiest way to handle is to upgrade |
| **temp** | `%temp%\NuGetScratch` | NUGET_SCRATCH | `D:\_DevEnv\Nuget\scratch` | maybe don't move this since nothing persists |

NOTE: We can skip talking about **machine level nuget.config** settings, because environment variables override everything. Or that's the way it's worded in the docs (will update if incorrect).

## 3. Update the nuget.config file(s)

The heirarchy of nuget.config files is complex, with potentially multiple files coming into play depending on where the nuget command is run from and which solution or project is used.

TIP: avoid solution level nuget.config if you ever intend to do nuget commands against one of the projects. nuget restore against a project will ignore the solution level settings.

> NOTE: The primary concern for the nuget.config files is where the global package(s) are stored. If we use the ENVIRONMENT VARIABLE, it'll be overridden and won't matter if it appears or not.

### 3.1 **NuGet.Config** location and heirarchy
| Level | Location | Comment |
|---|---|---|
| machine | `%ProgramFiles(x86)%\NuGet\Config\` | defines the VS Offline source `%ProgramFiles(x86)%\Microsoft SDKs\NugetPackages` same as the user config. |
| user | `%appdata%\Roaming\NuGet\NuGet.Config` | defines the sources `api.nuget.org` and `%ProgramFiles(x86)%\Microsoft SDKs\NugetPackages` |
| folder | Any folder above the solution or project on the same drive. | Ex:  `D:\Projects\nuget.config` will influence every project in a subfolder of `D:\Projects\` |

### 3.2 Integration with Azure Pipelines for private package repos

To connect to Azure Pipelines, the config `packageSources` needs to be cleared and the new source added. Clearing is necessary in cases where you want nuget.org only to be reached through your private package repo.

```xml
<configuration>
  <packageSources>
    <!-- the clear removes the defaults -->
    <clear />
    <!-- cleared: key="nuget.org" -->
    <!-- cleared: key="Microsoft Visual Studio Offline Packages" -->
    <add key="osmis-nuget-feed" value="https://azure-devops-server/collection/project/_packaging/feed-name/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

### 3.3 Update the Global Packages Folder
> Yes, we did say you can ignore setting this value. It's here in case you're a curious cat and just want to know; OR are personally offended by Environment Variables.

```xml
<configuration>
  <config>
    <clear />
    <add key="globalPackagesFolder" value="D:\_DevEnv\nuget\packages\" />
  </config>
</configuration>
```
