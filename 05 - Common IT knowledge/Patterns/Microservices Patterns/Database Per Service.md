Each service owns its persistent data and controls access to it.

```
Order Service   → Order database
Payment Service → Payment database
```

Other services use APIs or events—not direct table access.

### Does this require separate database servers?

No. Services can share a database cluster while using separate databases or schemas and credentials.

The important distinction is **exclusive ownership**, although shared infrastructure still creates resource contention and a shared failure domain.

### Why use it?

- Independent schema changes and deployments.
- Clear responsibility for business invariants.
- Freedom to choose suitable storage.
- Prevention of hidden coupling through SQL.

### Practical limitations

You lose convenient cross-service:

- Joins.
- Foreign keys.
- Local ACID transactions.

Replace them selectively with:

- **API aggregation** for composed reads.
- **CQRS projections** for query-heavy views.
- **Sagas** for multi-service workflows.
- **Outbox events** for reliable propagation.