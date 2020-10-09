# DnsClient.NET

## Purpose of D2L Fork

DnsClient found on github is owned by one individual, and uses an older version of System.Buffers.

We needed to update that dependency, but got no reply on our PR. We had several options to rectify the dependency mismatch:

1. Use Binding Redirects in all the entry point app/web.config files. We want to avoid this in the LMS because of the massive amount of places we'd need to update.
1. Put in a PR on the open source version of DnsClient. We tried this, and got no reply. Only one maintainer.
1. `AutoGenerateBindingRedirects` won't work with how we generate our web.config.
1. Fork the repo and update the dependency. What we did for now.
1. Future plan - move this logic to a lambda function and use Node.js's native MX/SPF/TXT record checking and get rid of this dependency.

[![Build Status](https://dev.azure.com/michaco/DnsClient/_apis/build/status/MichaCo.DnsClient.NET?branchName=dev&label=Build)](https://dev.azure.com/michaco/DnsClient/_build/latest?definitionId=1&branchName=dev)
[![Code Coverage](https://img.shields.io/azure-devops/coverage/michaco/DnsClient/1?label=Coverage&style=flat&color=informational)](https://dev.azure.com/michaco/DnsClient/_build/latest?definitionId=1&branchName=dev)
[![NuGet](https://img.shields.io/nuget/v/DnsClient?color=brightgreen&label=NuGet%20Stable)](https://www.nuget.org/packages/DnsClient)
[![NuGet](https://img.shields.io/nuget/vpre/DnsClient?color=yellow&label=NuGet%20Latest)](https://www.nuget.org/packages/DnsClient) 

DnsClient.NET is a simple yet very powerful and high performance open source library for the .NET Framework to do DNS lookups.

## Usage

See http://dnsclient.michaco.net for more details and documentation.

The following example instantiates a new `LookupClient` to query some IP address.

``` csharp

var lookup = new LookupClient();
var result = await lookup.QueryAsync("google.com", QueryType.A);

var record = result.Answers.ARecords().FirstOrDefault();
var ip = record?.Address;
``` 

## Features

### General

* Sync & Async API
* UDP and TCP lookup, configurable if TCP should be used as fallback in case the UDP result is truncated (default=true).
* Configurable EDNS support to change the default UDP buffer size and request security relevant records
* Caching
  * Query result cache based on provided TTL 
  * Minimum TTL setting to overrule the result's TTL and always cache the responses for at least that time. (Even very low value, like a few milliseconds, do make a huge difference if used in high traffic low latency scenarios)
  * Maximum TTL to limit cache duration
  * Cache can be disabled
* Multiple DNS endpoints can be configured. DnsClient will use them in random or sequential order (configurable), with re-tries.
* Configurable retry of queries
* Optional audit trail of each response and exception
* Configurable error handling. Throwing DNS errors, like `NotExistentDomain` is turned off by default
* Optional Trace/Logging

### Supported resource records

* A, AAAA, NS, CNAME, SOA, MB, MG, MR, WKS, HINFO, MINFO, MX, RP, TXT, AFSDB, URI, CAA, NULL, SSHFP, TLSA, RRSIG, NSEC, DNSKEY, DS
* PTR for reverse lookups
* SRV for service discovery. `LookupClient` has some extensions to help with that.
* AXFR zone transfer (as per spec, LookupClient has to be set to TCP mode only for this type. Also, the result depends on if the DNS server trusts your current connection)

## Build from Source

The solution requires a .NET Core 3.x SDK and the [.NET 4.7.1 Dev Pack](https://www.microsoft.com/net/download/dotnet-framework/net471) being installed.

Just clone the repository and open the solution in Visual Studio 2017/2019.

The unit tests don't require any additional setup right now.

If you want to test the different record types, there are config files for Bind under tools. 
Just [download Bind](https://www.isc.org/downloads/) for Windows and copy the binaries to tools/BIND, then run bind.cmd.
If you are running this on Linux, you can use my config files and replace the default ones if you want.

Now, you can use **samples/MiniDig** to query the local DNS server. 
The following should return many different resource records:

``` cmd
dotnet run -s localhost mcnet.com any
```

## Examples

* More documentation and a simple query window on http://dnsclient.michaco.net
* The [Samples](https://github.com/MichaCo/DnsClient.NET.Samples) repository (there might be more in the future).
* [MiniDig](https://github.com/MichaCo/DnsClient.NET/tree/dev/samples/MiniDig)