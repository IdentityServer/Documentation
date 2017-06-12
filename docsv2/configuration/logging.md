---
layout: docs-default
---

# Logging

IdentityServer has two logging related features. Development-time logging and production-time events ([see here](events.html)).

Development-time logging produces a quite extensive output and is mostly useful for developers that customize IdentityServer.
Logging might store sensitive data like passwords and thus is typically not suitable for production use.

IdentityServer uses [LibLog](https://github.com/damianh/LibLog) for logging. 
Liblog picks up any of the following logging libraries automatically:

* NLog
* Enterprise Library
* SeriLog
* Log4Net
* Loupe

There is no IdentityServer3 specific configuration required - you just need to configure one of the above logging frameworks in your host.

*Warning: LibLog will pick the first library in the above order and will discard the others. So if you have a reference to SeriLog for example and you're trying to configure Log4net it will **not** work.*

## Configuring Diagnostics
The `LoggingOptions` class has the following settings:

* `EnableWebApiDiagnostics`
   * If enabled, Web API internal diagnostic logging will be forwarded to the log provider
* `WebApiDiagnosticsIsVerbose`
   * If enabled, the Web API diagnostics logging will be set to verbose
* `EnableHttpLogging`
   * If enabled, HTTP requests and responses will be logged
* `EnableKatanaLogging`
   * If enabled, the Katana log output will be logged (this is often useful to troubleshoot problems with external identity providers)

Set the `LoggingOptions` property of your `IdentityServerOptions` to a new instance of the above class with the desired options. For example:

```csharp
var options = new IdentityServerOptions
{
	// other options here...

	LoggingOptions = new LoggingOptions
	{
		EnableKatanaLogging = true
	}
};

coreApp.UseIdentityServer( options );
```

## Example: Using Serilog to log to System.Diagnostics tracing
The following example wires up [Serilog](http://serilog.net/) to log to the diagnostics trace (put that e.g. in Startup or in your hosting code). 
**Note:** Serilog provides various logging sinks as separate packages, so you may need to install the Serilog.Sinks.Trace package to get `WriteTo.Trace()` to work as expected. 

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Trace()
    .CreateLogger();
```

Add the following snippet to your configuration file to funnel all logging messages to a simple text file.
We use [Baretail](https://www.baremetalsoft.com/baretail/) for viewing the log files.

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

**Note:** If you use this method you need to ensure that the account running the application pool has write access 
to the directory containing the log file. 
If you don't specify a path, this will be the application directory, which is not recommended for production scenarios. 
For production log to a file **outside** the application directory.

## Example: Log to the console
Logging to the console gives you a friction free and immediate insight into the internals of IdentityServer. Serilog has a nice colored
console logging sink called `Serilog.Sinks.Literate`. Wire it up like this:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.LiterateConsole()
    .CreateLogger();
```

## Instrumenting your own code
You can also use the logging system in your own extensibility code.

Add a static `ILog` instance to your class

```csharp
private readonly static ILog Logger = LogProvider.For<MyClass>();
```
Log your messages using the logger

```csharp
Logger.Debug("Getting claims for identity token");
```

## Using your own Logging Infrastructure
You may have an existing logging infrastructure in place and want IdentityServer logging to use that.
The recommended approach for this is to write a custom sink using one of the supported logging frameworks (our favourite is Serilog).
You can find a sample [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/dev/source/Logging).

See [here](http://leastprivilege.com/2015/10/22/identityserver3-logging-monitoring-using-serilog-and-seq/) for a post about logging and eventing.

## Suppressing all logging output
**(added in v2.5)**

For certain scenarios (e.g. production) you want to make sure that no logging output is produced. For this you can configure a no-op logger (typically done in `Startup` or in the hosting code):

```csharp
LogProvider.SetCurrentLogProvider(new NoopLogProvider());
```
