---
layout: docs-default
---

# Logging

IdentityServer produces extensive logging output.
The logging mechanism and sink is determined by the hosting application via setting a `LogProvider` (e.g. in your `startup` class):

```csharp
LogProvider.SetCurrentLogProvider(new DiagnosticsTraceLogProvider());
```

The following providers are supported out of the box:

* System.Diagnostics.Trace (`DiagnosticsTraceLogProvider`)
* TraceSource (`TraceSourceLogProvider`)
* NLog (`NLogLogProvider`)
* Enterprise Library (`EntLibLogProvider`)
* SeriLog (`SerilogLogProvider`)
* Log4Net (`Log4NetLogProvider`)
* Loupe (`LoupeLogProvider`)

You can create additional providers by deriving from `LogProvider`.

## Configuring Diagnostics
The `LoggingOptions` class has the following settings:

* `EnableWebApiDiagnostics`
   * If enabled, Web API internal diagnostic logging will be forwarded to the log provider
* `WebApiDiagnosticsIsVerbose`
   * If enabled, the Web API diagnostics logging will be set to verbose
* `EnableHttpLogging`
   * If enabled, HTTP requests and response will be logged
* `IncludeSensitiveDataInLogs`
   * If enabled, the standard logging might include sensitive data like PII data

**Warning** `EnableHttpLogging` might conflict with other frameworks loaded into the same web application (it does for sure with MVC). You can't use HTTP logging in that situation.

## Configuring the System.Diagnostics Provider
Add the following snippet to your configuration file to funnel all logging messages to a simple text file.
We use [Baretail](https://www.baremetalsoft.com/baretail/) for viewing the log files.

**Note:** If you use this method you need to ensure that the account running the application pool has write access to the directory containing the log file. If you don't specify a path, this will be the application directory, which is not recommended for production scenarios. For production log to a file **outside** the application directory.

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

## Configuring the TraceSource Provider
Add the following snippet to your config file. You can use the service trace viewer tool from the .NET SDK to inspect the trace files.

```xml
<sources>
  <source name="Thinktecture.IdentityServer"
          switchValue="Information, ActivityTracing">
    <listeners>
      <add name="xml"
            type="System.Diagnostics.XmlWriterTraceListener"
            initializeData= "trace.svclog" />
    </listeners>
  </source>
</sources>
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
