---
layout: docs-default
---

#Logging

IdentityServer has two logging related features. Development-time logging and production-time events ([see here](events.html)).

For development-time logging produces quite extensive output and is mostly useful for developers that customize IdentityServer.
Logging might store sensitive data like passwords and thus is typically not suitable for production use.

IdentityServer uses [LibLog](https://github.com/damianh/LibLog) for logging. 
Liblog picks up any of the following logging libraries automatically:

* NLog (`NLogLogProvider`)
* Enterprise Library (`EntLibLogProvider`)
* SeriLog (`SerilogLogProvider`)
* Log4Net (`Log4NetLogProvider`)
* Loupe (`LoupeLogProvider`)

##Configuring Diagnostics
The `LoggingOptions` class has the following settings:

* `EnableWebApiDiagnostics`
   * If enabled, Web API internal diagnostic logging will be forwarded to the log provider
* `WebApiDiagnosticsIsVerbose`
   * If enabled, the Web API diagnostics logging will be set to verbose
* `EnableHttpLogging`
   * If enabled, HTTP requests and response will be logged
* `EnableKatanaLogging`
   * If enabled, the Katana log output will be captured


## Using System.Diagnostics tracing
The following example wires up [Serilog] to log to the diagnostics trace:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Trace()
    .CreateLogger();
```

Add the following snippet to your configuration file to funnel all logging messages to a simple text file.
We use [Baretail](https://www.baremetalsoft.com/baretail/) for viewing the log files.

**Note:** If you use this method you need to ensure that the account running the application pool has write access 
to the directory containing the log file. 
If you don't specify a path, this will be the application directory, which is not recommended for production scenarios. 
For production log to a file **outside** the application directory.

```xml
<system.diagnostics>
  <trace autoflush="true"
         indentsize="4">
    <listeners>
      <add name="myListener"
           type="System.Diagnostics.TextWriterTraceListener"
           initializeData="Trace.log" />
      <remove name="Default" />
    </listeners>
  </trace>
</system.diagnostics>
```

## Instrumenting your own code
You can also use the logging system in your own extensibility code.

Add a static `ILog` instance to your class

```csharp
private readonly static ILog Logger = LogProvider.GetCurrentClassLogger();
```
Log your messages using the logger

```csharp
Logger.Debug("Getting claims for identity token");
```
