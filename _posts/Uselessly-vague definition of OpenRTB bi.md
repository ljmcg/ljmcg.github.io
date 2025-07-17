Uselessly-vague definition of OpenRTB bidder:

At an extremely vague level, a bidder evaluates a function, F, of ( BidRequest, BidderHistory, AdConfiguration ), returning a pair, { BidResponse, BidderHistory* }

Based only on the bidrequest, configured advertisements, and bidder history, it will determine which (if any) bid response to generate and will update the bidderhistory, as necessary.

In any short timespan, AdConfiguration is a constant (ad campaigns and creatives change slowly, over the course of days/weeks) and BidderHistory abstracts all data that changes over a short term (seconds/minutes/hours).

Examples of AdConfiguration include:
* Creative assets
* PMP deals
* Forbidden web domains or apps
* Geotargeting
* Frequency Cap Settings

Examples of BidderHistory include:
* Frequency Cap History/Data
* Campaign budget remaining
* Adaptive pacing settings

This brings up important facts:

1. AdConfiguration data can be pre-processed with pre-computed lookup tables, indexes, etc... to optimize processing of this part of the function. As a constant value, it can be propagated to each bidder independently and the bidder does not need to store it in a non-volatile fashion.
    * Pre-processing this constant data will require development of both the pre-processor and a modular and flexible bidder to work with pre-processed data.
        * These will need to be paired together, of course.
    * The flexibility of this arrangement will permit apples-to-apples comparisons.
    * The flexibility of this arrangement will permit extensive randomized testing to verify consistent behavior.
    * The pre-computed data should be small enough to fit in RAM, if possible.
        * The number of seats, campaigns, and frequency caps should be essentially negligible.
        * Audience member lists may be voluminous
            * It's plausible that these could contain tens of millions or a hundred million elements, at dozens of bytes apiece.
        * Forbidlist and allowlist data for sites and apps may be voluminous.
            * This may be as many as a million values, at 50-100 bytes each.
        * Ad-Markup may be voluminous (thousands of bytes per creative)
            * I assume (without data) that burl and nurl data have stricter size limits.
            * Using these methods would also defer work until the auction win.
        * The same applies for VAST markup.
    * If we assume ~1kb of data per creative, then a million creatives would fit into a gigabyte of RAM.
        * Even if this estimate is low by a factor of 2 or 4 (on the average) then it would still be feasible to store this in RAM.
2. BidderHistory must be -shared- across all bidders (or all bidders serving one region), and must be available despite bidders being created and destroyed.
    * As a practical matter, this means that it must be stored in a resource external to any one bidder, e.g. Redis or memcached instances.
    * BidderHistory can also be partitioned, to divide the workload among multiple instances. For frequency capping, the user/device id is a natural partition key.
    * These out-of-process accesses will use other virtual or physical compute resources and such operations can introduce latency and variance in processing time.
        * Maintain persistent connections from each bidder to each BidderHistory instance (avoid TCP handshake overhead)
        * Perform work in the fewest number of calls possible (e.g. check frequency cap state for many creatives in a single call)
          Consider using modules to extend the data engine in order to minimize round-trips.
        * Make these calls in parallel (if a process requires a frequency cap query and a remaining-budget query that go to two different History shards, run them simultaneously)
    * The BidderHistory may also be updated by processes other than Bidders, e.g. the win-notice server or billing-notice server.