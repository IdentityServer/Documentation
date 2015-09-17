---
layout: docs-default
---

The validation middleware uses the standard Katana [logging](https://katanaproject.codeplex.com/wikipage?title=Debugging&referringTitle=Documentation) facilities.

By default Katana uses the `TraceSource` mechanism in .NET for logging. 
Add the following snippet to your config file to enable logging to a file:

```xml
<system.diagnostics>
  <trace autoflush="true" />

  <sources>
    <source name="Microsoft.Owin">
      <listeners>
        <add name="KatanaListener" />
      </listeners>
    </source>
  </sources>

  <sharedListeners>
    <add name="KatanaListener"
          type="System.Diagnostics.TextWriterTraceListener"
          initializeData="katana.trace.log"
          traceOutputOptions="ProcessId, DateTime" />
  </sharedListeners>

  <switches>
    <add name="Microsoft.Owin"
          value="Verbose" />
  </switches>
</system.diagnostics>
```