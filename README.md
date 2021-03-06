# LogPack

Welcome to the LogPack Beta. This repo is used for documentation purposes. Issues are used to track features and bugs until LogPack goes is published as a separate product.

![LogPack Explorer](/LogPack%20Explorer.png?raw=true)

# What and Why?

Services that provide logging as a service collect all logs and allow you to search and filter using keywords or specific log files. In production however, we would like to collect as less logs as possible to limit the price for the services. Furthermore, we ran into issues where environment variables were not correctly set in the production or staging environment and had issues with different versions of NuGet packages. Analyzing such issues took us a lot of time which is why we came up with the idea of LogPack. Whenever an unexpected issue happens, collect everything related to this issue and store it packaged as a zip file on an FTP server. So no more logs for API calls with return code 200. But literally everything necessary to analyze failed API calls with return code 500.

> Special note: LogPack is a product created while creating FeatureNinjas. We use it to analyze our server issues. If you want to have more infos on FeatureNinjas, a feature flagging service for developers, then check out https://featureninjas.com.

# Open Source

We're planning on open sourcing the core product so that you can push logs to any storage that you want. Currently it is closed source until the core reaches a certain level of maturity. Watch this repo to get notified when we release the code.

# Prerequisites

- LogPack currently only runs in ASP.net core 3.2+. We're looking forward to implement this also for NodeJS and others. 
- You will need an FTP server to store the log packs. In future, more storage types will be support. Tell us what you need.

# Installation

- Download the NuGet package and VS code extension from the Releases in this repository (we will upload those to nuget.org and the VS Code marketplace later) (Subscribe to get updates on new releases)
- Install the NuGet package in your asp.net core project and add the required configuration into the `Configure()` methods of `Startup.cs`

``` cs
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseLogPack(new LogPackOptions()
    {
        Sinks = new LogPackSink[]
        {
            new FtpSink(
                [ftp-server-url], 
                [ftp-server-port], 
                [ftp-server-username], 
                [ftp-server-password], 
        },
        ProgramType = typeof(Program)
    });
    // ...
    app.UseRouting();
}
```

- Install the VS Code extension (install the .vsix file that you downloaded) and configure the same FTP server connection in the LogPack configuration section

# Additional Features

By default, a log pack is created and uploaded whenever the request responds with a 5xx return code. You can use include filters (even create your own) to change this. For example, to create a log pack for all return code 3xx, 4xx and 5xx, add the following lines to the `LogPackOptions` object (see above)

``` cs
Include = new IIncludeFilter[]
{
  new StatusIncludeFilter(0, 1000),
},
```
    
In order to create a custom include filter, implement the `IIncludeFilter` interface. Example:

``` cs
public class LogPackAccountIdIncludeFilter : IIncludeFilter
{
  public bool Include(HttpContext context)
  {
      if (context.Items.ContainsKey("accountId")
          && (context.Items["accountId"].ToString() == "1234"
          || context.Items["accountId"].ToString() == "5678"))
      {
          return true;
      }

      return false;
  }
}
```
    
If this include filter is added to the log pack options, then for all requests for the user with the account ID is 1234 or 5678, a log pack is created.

You could even base include filters on feature flags, e.g. by using FeatureNinjas ;).

# Roadmap

The following list contains features that we're planning to implement in the next updates. Help us prioritize by giving us feedback

- Support for NodeJS
- Add more upload sinks (e.g. OneDrive, AWS, ...)
- Offer online storage so you don't have to care about FTP or any other service 
- ...

Missing something? Create an issue or contact us directly.
