Model of mouse/elephant problem:

1. Needs to use a huge data stream.
2. Needs to maintain relatively small amount of state (and update a relatively small amount per operation)
3. Needs to see thw whole data stream.
4. Needs to discard almost-all low-frequency items.
5. Probably needs multiple thresholds (?) to be sure to arrest surges quickly. Determine the time-scales they occur in, based on empirical data.
 a. Multiple entry points into a higher-level filter may be appropriate, catching a few frequencies/rates.
  i. e.g. Use a Bloom Filter on requestorigin that resets every 100ms. Anything that hits that with the BF already set gets escalated.
  ii. Use a second bloom filter that splits each second into 10 and triggers if a requestorigin appears 3+ times in that.
 b. After any of the first-level filters occurs, add that requestorigin to a filter that accumulates suspected elephants. Reprocess this list periodically, to maintain it.
 c. But I expect that the proportion of messages caught this way will be extremely small. From ~1M bidrequests/sec to ~100 over-rate.
 d. Make the thresholds and times configurable as well, so it can be altered via a config file policy.