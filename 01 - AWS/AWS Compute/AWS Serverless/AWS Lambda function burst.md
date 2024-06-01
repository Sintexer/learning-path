Burst behavior refers to how Lambda handles sudden increases in request volume.

**Immediate concurrency increase**: When a burst happens, lambda scales up immediately to a predefined level of concurrency. Concurrency is the number of invocations your function runs at any given moment. 

**Incremental scaling**: AWS Lambda adds 500 more invocations per minute until the burst is handled or limits are reached.

**Throttling**: not all account concurrency is available immediately, leading to potential throttling during bursts.