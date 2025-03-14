---
title: "New Project & First Numbers"
date: 2025-03-13 18:58:00 -5:00
---
With an OpenRTB 2.6 bidder as the project, the obvious first step is to represent the json data elements. I searched github for relevant projects and only found one C# project for OpenRTB integration. It's untouched for about 5 years and doesn't look like it would save me a lot of effort.

As this is my own project and my recent and relevant experience is in C# Asp.Net Core webapis, I'm going to continue using C#.

My initial milestone will be to stand up a C# Asp.Net Core webapi with an endpoint to accept BidRequest json and to return a No-Content (204) response in all cases. This will permit some very early-stage benchmarking hint towards a top-end throughput, and give an idea of what to expect.

Accepting properly-formed BidRequests and creating valid BidResponses will require implementing about 33 types defined in the OpenRTB spec.

_hours of furious typing later_

Now that I have my own hand-keyed in datatypes (using C# record constructor syntax to keep things brief), I can set up some unit testing to check things for reasonableness.

The spec has [5 examples defined](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/main/2.6.md#62---bid-requests-) and it's easy to extract those to use for unit tests.

The first round of testing is simple, excerpting it here:

```C#
        public static readonly JsonSerializerOptions jsonSerializerOptions = new JsonSerializerOptions() { UnmappedMemberHandling = JsonUnmappedMemberHandling.Disallow };
        [Theory]
        [MemberData(nameof(exampleBidRequestAdaptToMemberData))]
        public void CanDeserializeBid(string testData)
        {
            //These options dictate that deserialization will fail when it encounters an undeclared attribute.
            var BidReq = JsonSerializer.Deserialize<BidRequest>(testData, jsonSerializerOptions);
            Assert.NotNull(BidReq);
        }
```

The ``JsonUnmappedMemberHandling.Disallow`` option will cause my tests to break if they encounter any undocumented elements. This simple test revealed a few of my own typos and suggested some errors in the published examples. I opened an issue in the source repo to ask about the discrepancies.

With those first typos fixed (I'm sure I'll find more later, but a manual review would be a poor use of time -- I'll find more example data), I turned to some rough performance numbers.

BenchmarkDotNet was very easy to set up and got me running right away. Since a Bidder will have to decode the Json BidRequest for everything it does, benchmarking Json deserialization using the System.Text.Json deserializer is sensible to benchmark.

```
// * Summary *

BenchmarkDotNet v0.14.0, Windows 11 (10.0.26100.3476)
AMD Ryzen 5 5500, 1 CPU, 12 logical and 6 physical cores
.NET SDK 9.0.201
  [Host]     : .NET 9.0.3 (9.0.325.11113), X64 RyuJIT AVX2
  DefaultJob : .NET 9.0.3 (9.0.325.11113), X64 RyuJIT AVX2


| Method            | fileName             | Mean     | Error     | StdDev    | Gen0   | Allocated |
|------------------ |--------------------- |---------:|----------:|----------:|-------:|----------:|
| DeserializeString | Bench(...).json [30] | 4.003 us | 0.0607 us | 0.0568 us | 0.3357 |   2.79 KB |
| DeserializeUTF8   | Bench(...).json [30] | 3.766 us | 0.0538 us | 0.0503 us | 0.3395 |   2.79 KB |
```

Mean times for the deserialize alone are about 4 microseconds. That's compatible with overall high throughput, but less time would be more better.

BenchmarkDotNet was easy to set up and gave me some quick data, but it's still a microbenchmark, and doesn't exercise a complete system.

To do _that_, I'll head straight towards the simplest possible webapi.

And then exercise it with Microsoft's tool [``crank``](https://github.com/dotnet/crank), which will benchmark the webapi.

First, I got a baseline, using the [sample "hello" benchmark](https://github.com/dotnet/crank/blob/main/samples/hello/hello.benchmarks.yml).

```
crank --config hello.benchmarks.yml --scenario hello --profile local

<snip>

| application               |                     |
| ------------------------- | ------------------- |
| Max Process CPU Usage (%) | 58                  |
| Max Cores usage (%)       | 695                 |
| Max Working Set (MB)      | 209                 |
| Max Private Memory (MB)   | 193                 |
| Build Time (ms)           | 4,363               |
| Start Time (ms)           | 488                 |
| Published Size (KB)       | 100,002             |
| Symbols Size (KB)         | 0                   |
| .NET Core SDK Version     | 8.0.407             |
| ASP.NET Core Version      | 8.0.14+25ef4aa38b77 |
| .NET Runtime Version      | 8.0.14+1584e493603c |
| Max Global CPU Usage (%)  | 100                 |


| load                      |                     |
| ------------------------- | ------------------- |
| Max Process CPU Usage (%) | 46                  |
| Max Cores usage (%)       | 558                 |
| Max Working Set (MB)      | 39                  |
| Max Private Memory (MB)   | 36                  |
| Build Time (ms)           | 4,131               |
| Start Time (ms)           | 267                 |
| Published Size (KB)       | 72,252              |
| Symbols Size (KB)         | 0                   |
| .NET Core SDK Version     | 8.0.407             |
| ASP.NET Core Version      | 8.0.14+25ef4aa38b77 |
| .NET Runtime Version      | 8.0.14+1584e493603c |
| Max Global CPU Usage (%)  | 100                 |
| First Request (ms)        | 153                 |
| Requests                  | 3,388,893           |
| Bad responses             | 0                   |
| Latency 50th (ms)         | 1.12                |
| Latency 75th (ms)         | 2.01                |
| Latency 90th (ms)         | 2.08                |
| Latency 95th (ms)         | 2.55                |
| Latency 99th (ms)         | 3.27                |
| Mean latency (ms)         | 1.13                |
| Max latency (ms)          | 43.97               |
| Requests/sec              | 225,479             |
| Requests/sec (max)        | 288,296             |
| Read throughput (MB/s)    | 26.50               |

```

This will be a point of comparison, as the benchmark and app here do _practically nothing_. It's nice to see that the 99th percentile latency for doing nothing is at 3.27ms, and the Asp.Net Core stack can deliver 225,479 sustained ``Requests/sec``.

Repeating this with my simplest webapi gives:

```
| application               |                                 |
| ------------------------- | ------------------------------- |
| Max Process CPU Usage (%) | 88                              |
| Max Cores usage (%)       | 1,058                           |
| Max Working Set (MB)      | 141                             |
| Max Private Memory (MB)   | 102                             |
| Build Time (ms)           | 7,481                           |
| Start Time (ms)           | 914                             |
| Published Size (KB)       | 104,691                         |
| Symbols Size (KB)         | 52                              |
| .NET Core SDK Version     | 9.0.100-rtm.24513.10            |
| ASP.NET Core Version      | 9.0.0-rtm.24515.11+00ac5bf2367a |
| .NET Runtime Version      | 9.0.0-rtm.24516.5+9305d7f71d73  |
| Max Global CPU Usage (%)  | 100                             |


| load                      |                     |
| ------------------------- | ------------------- |
| Max Process CPU Usage (%) | 27                  |
| Max Cores usage (%)       | 323                 |
| Max Working Set (MB)      | 40                  |
| Max Private Memory (MB)   | 36                  |
| Build Time (ms)           | 4,164               |
| Start Time (ms)           | 270                 |
| Published Size (KB)       | 72,252              |
| Symbols Size (KB)         | 0                   |
| .NET Core SDK Version     | 8.0.407             |
| ASP.NET Core Version      | 8.0.14+25ef4aa38b77 |
| .NET Runtime Version      | 8.0.14+1584e493603c |
| Max Global CPU Usage (%)  | 100                 |
| First Request (ms)        | 126                 |
| Requests                  | 1,440,834           |
| Bad responses             | 0                   |
| Latency 50th (ms)         | 2.51                |
| Latency 75th (ms)         | 3.30                |
| Latency 90th (ms)         | 4.02                |
| Latency 95th (ms)         | 4.46                |
| Latency 99th (ms)         | 6.64                |
| Mean latency (ms)         | 2.66                |
| Max latency (ms)          | 313.38              |
| Requests/sec              | 95,859              |
| Requests/sec (max)        | 127,288             |
| Read throughput (MB/s)    | 7.42                |
```

This is good, though I don't love the Max latency number, and I'd be happier to see it improve.

ljmcg