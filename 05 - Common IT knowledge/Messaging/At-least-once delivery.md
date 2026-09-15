A message might be delivered and processed more than once in case of retries.

Commit the offset **after** successful processing.

- If the consumer crashes _after_ finishing the work but _before_ the commit lands, on restart it re-reads from the last committed offset — meaning it **reprocesses** that message again.
- Consequence: **duplicates are possible**, but **nothing is ever silently lost**.
- This is the **default, practical choice** in almost all real systems — because losing data is almost always worse than occasionally doing something twice, _provided_ you've made the processing safe to repeat.