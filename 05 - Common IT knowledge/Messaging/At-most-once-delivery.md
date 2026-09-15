Commit the offset **before** processing the message.

- If the consumer crashes mid-processing, the offset is already committed → on restart, Kafka thinks that message is done → it's **never redelivered** → the work is simply lost.
- Rarely intentional. You'd only choose this if losing occasional messages is truly acceptable and reprocessing/duplicates would be worse (rare in practice — maybe non-critical metrics/logging).