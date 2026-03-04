# Kafka Consumer Lag Scaler Runbook

This runbook provides guidance for diagnosing and resolving Kafka consumer lag issues.

## Symptoms

- Pipeline throttling due to lag
- Consumer group showing high lag metrics
- Enrichment service performance degradation

## Diagnosis Steps

### 1. Check Consumer Group Status

```bash
kafka-consumer-groups --bootstrap-server <BOOTSTRAP_SERVER> --describe --group <CONSUMER_GROUP>
```

### 2. Check Time-Based Lag in Dashboard

**Important:** If you don't see time-based lag in the Kafka dashboard, this is a sign that a rebalance is occurring. During a rebalance:
- Consumers stop fetching messages temporarily
- Lag metrics may not be reported correctly
- The consumer group is in a transitional state

### 3. Check Partition Assignments

Verify that partitions are properly assigned to consumers:

```bash
kafka-consumer-groups --bootstrap-server <BOOTSTRAP_SERVER> --describe --group <CONSUMER_GROUP> --members
```

## Resolution Steps

### 4. Standard Rebalancing

If lag is caused by uneven partition distribution, trigger a rebalance by scaling the consumer group.

### 5. Force Clean Rebalance

If the consumer group appears stuck or partitions are not being assigned properly, force a clean rebalance:

```bash
# Scale down the consumer deployment temporarily, then scale back up
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=0
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=<DESIRED_REPLICAS>
```

### 6. No Assignment Visible in kafka-consumer-groups

If `kafka-consumer-groups --describe` shows **no assignment**, the consumer group may be stuck in a rebalancing state.

In this case, use the revocation script to force a clean rebalance:

```bash
./revocation.sh <CONSUMER_GROUP>
```

This has been observed during incidents where partitions become stuck and cannot be reassigned through normal rebalancing.

## Key Indicators of Rebalance Issues

| Symptom | Likely Cause |
|---------|--------------|
| No time-based lag visible in dashboard | Rebalance in progress |
| No partition assignments in consumer-groups output | Stuck rebalance state |
| Intermittent lag spikes | Frequent rebalancing |
| All consumers showing as "Unknown" state | Consumer group coordination failure |

## Related Incidents

- Pipeline throttling incidents may be resolved by rebalancing lagging partitions
- If standard troubleshooting doesn't work, escalate to the team owning the consumer application
