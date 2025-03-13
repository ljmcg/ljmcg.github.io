---
title: "New Challenge"
date: 2025-03-12 21:30:00 -5:00
---

### Why?

Since I spent decades of my life working in a specific industry, I want to branch out and explore some other areas -- ones where I honestly don't know the requirements so well, where I don't know the market and expectations so well.

I want to stretch a bit and push myself into challenges that will encourage and reward developing skills that I haven't had the chance to develop before. I *enjoy* creating software, and now is a time when I can direct myself a little and try anything that catches my fancy.

### Criteria

I want a project that has a publicly available specification, that has unique technical challenges, and where meeting those technical challenges will be difficult and also recognized.

I have enjoyed my share of informal and low-stakes coding competitions in the past. But they tend to be contrived and disconnected from application.

Likewise, I have enjoyed delving into new or different programming languages for the joy of it. But very often, that's a lateral move. Sometimes a language does really change the way you think, but I'm not chasing language trends right now either (I'll pick up proficiency in what I need when I need it).

Nor do I want to build libraries for the sake of building libraries. I'm sure I'll create some along the way, maybe they'll even prove useful to others, but they aren't an end in themselves.

### My Project

When I looked around at high-throughput server software that has a publicly-available specification and is of ongoing commercial interest, I found something fascinating and a little intimidating. It's outside my experience, but it does share some features with things I've done before.

I've decided to scope, design, and prototype a sort of an online advertising DSP bidder ([Some background here](https://clearcode.cc/blog/building-rtb-bidder/)).

This won't become something full-featured, nor will I build out the remainder of the infrastructure needed to make real time bidding function, but the constraints are a fascinating challenge.

### The Spec

Thankfully, the [Interactive Advertising Bureau (IAB)](https://www.iab.com/) publishes their specification for integrating bidders with ad exchanges, and it seems to be a well supported standard (though not everyone supports the latest versions and there are features left to extensions in practice). The specification for [OpenRTB v2.6](https://iabtechlab.com/wp-content/uploads/2022/04/OpenRTB-2-6_FINAL.pdf) is available and licensed (as Creative Commons Attribution 3.0) for anyone to read and implement.

Page 4 of the document shows the whole set of message flow. I'm _only_ considering the item labeled as ``1. Bid Request/Bid Response``

My requirement in this regard is beautifully simple: Implement a REST endpoint that can accept a POST containing JSON text encoding the spec's item ``3.2.1 Object BidRequest`` (and child objects). It should then respond with a JSON-formatted ``4.3.1 Object: BidResponse`` (or a HTTP 204 (No Content)).

## Challenges

Access to the spec makes this project possible, but that doesn't make it interesting or challenging. The performance requirements of bidders and latency demands are what make this interesting. All resources I've read make it _crystal clear_ that lower latency is better and that there are hard limits on response latency. I don't know which figures/requirements to rely on, but a target of 100ms or less seems advisable.

Additionally, volume can be very high. Again, I can't find concrete figures, but I've seen vendors brag about processing 600,000 BidRequests per second. I imagine that for bigger operators, the volume passes 1 million per second. That creates some interesting design constraints.

In 2025, bidders are also liable to be cloud-native systems. This condition is a double-edged sword. On the one hand, the technical challenges I mention are feasible to solve with the application of money.

Microsoft's Azure CosmosDB will sell you up to 1,000,000 RU/s, which should easily support a million bids per second (with ordinary caching and a huge number of front-end servers) -- but it won't be cheap. [Looking at Microsoft's pricing for Cosmos DB standard provisioned throughput], they price RU/s at $5.84/month for 100 RU/s. To buy a million RU/s, you'd have a CosmosDB bill starting at $58,400/month (for one write region). That sort of spending isn't within my budget -- tackling this successfully will involve judicious use of resources and thoughtful design.

The operations per second challenge could also be solved without much trouble by deploying an army of virtual machines. Surely 1000 VMs could scale suitably, even with naive code and no optimization.

## Opportunities

Despite those challenges, this is clearly something that does exist and succeeds already. It's also something where the open spec and public product documentation for existing vendors allows some reasonable insight into what features exist currently and what options are realistic to expect and implement. Since I'm doing this for my own edification and to develop and demonstrated my own skills, I know for a fact that I'm not creating a world-beating product.

## Scope

To direct and focus myself, I'll keep the scope relatively narrow.

* Implement an OpenRTB 2.6 Bidder
    * Design a plausible, but minimal set of features that cover important use cases.
    * Design and implement some of the supporting software in the critical path of the BidRequest/BidResponse flow.
    * Configure the Bidder with config files and/or HTTP REST configuration endpoints. This is fine for a research project and demo.
    * Prove feasibility for areas first.
    * Continue making progress and building features out.
    * Test throughout.
    * Benchmark throughout, at the micro level, at the level of infrastructure, and in integration.
* Do not implement file delivery, a configuration UI, or accounting/monetary reconciliation systems.
    * These are critical for a production system, but too wide a scope for a one-person research project.

ljmcg