# Warden Server Watcher

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Server.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Watchers.Server)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Server.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Watchers.Server/branches)

**ServerWatcher** can be used for monitoring server availability including opened ports (if specified).

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Watchers.Server)**. 
```
dotnet add package Warden.Watchers.Server
```

### Configuration:

- **WithTimeout()** - timeout after which the invalid result will be returned.
- **EnsureThat()** - predicate containing a received *IHttpResponse* that has to be met in order to return a valid result.
- **EnsureThatAsync()** - async predicate containing a received *IHttpResponse* that has to be met in order to return a valid result.
- **WithDnsResolverProvider()** - provide a  custom *IDnsResolver* which is responsible for resolving the DNS. 
- **WithPingerProvider()** - provide a  custom *IDnsResolver* which is responsible for pinging the specified hostname.
- **WithTcpClientProvider()** - provide a  custom *ITcpClient* which is responsible for handling the TCP connection.
 

**ServerWatcher** can be configured by using the **ServerWatcherConfiguration** class or via the lambda expression passed to a specialized constructor.

Example of configuring the watcher via provided configuration class:
```csharp
var configuration = ServerWatcherConfiguration
    .Create("www.google.com", 80)
    .Build();
var serverWatcher = ServerWatcher.Create("Port watcher", configuration);

var wardenConfiguration = WardenConfiguration
    .Create()
    .AddWatcher(serverWatcher)
    //Configure other watchers, hooks etc.
```

Example of adding the watcher directly to the **Warden** via one of the extension methods:
```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .AddServerWatcher("www.google.com", 80)
    //Configure other watchers, hooks etc.
```

Please note that you may either use the lambda expression for configuring the watcher or pass the configuration instance directly. You may also configure the **hooks** by using another lambda expression available in the extension methods.

### Check result type:
**ServerWatcher** provides a custom **ServerWatcherCheckResult** type which contains additional values.

```csharp
public class ServerWatcherCheckResult: WatcherCheckResult
{
    public string Hostname { get; }
    public int Port { get; }
}
```
### Custom interfaces:
```csharp
public interface IDnsResolver
{
    IPAddress GetIpAddress(string hostnameOrIp);
}

public interface IPinger
{
    Task<IPStatus> PingAsync(IPAddress addressIp, TimeSpan? timeout = null);
}

public interface ITcpClient
{
    bool IsConnected { get; }
    Task ConnectAsync(IPAddress ipAddress, int port, TimeSpan? timeout = null);
}

public class ConnectionInfo
{
    public string Hostname { get; }
    public IPAddress IpAddress { get; }
    public int Port { get; }
    public bool PortOpened { get; }
    public IPStatus PingStatus { get; }
    public string PingStatusMessage { get; }
}
```

**IDnsResolver** is responsible for resolving the DNS. It can be configured via the *WithDnsResolverProvider()* method. 
**IPinger** is responsible for pinging the specified hostname. It can be configured via the *WithPingerProvided()* method. 
**ITcpClient** is responsible for handling the TCP connection for a given port number. It can be configured via the *WithTcpClientProvider()* method. By default these services are based on the **[System.Net.Sockets](https://msdn.microsoft.com/en-us/library/system.net.sockets)**.