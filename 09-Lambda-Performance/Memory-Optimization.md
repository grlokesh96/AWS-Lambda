# AWS Lambda Memory Optimization

Lambda memory controls the memory available to the function and also influences the CPU capacity allocated to it. Increasing memory can reduce execution time for CPU-bound workloads, but it can also increase cost per unit of execution time.

## Why memory matters

A function with too little memory may run slowly or fail due to memory exhaustion. A higher memory setting can provide more CPU and improve performance, depending on the workload.

## Configure memory

Set memory to 1024 MB:

```bash
aws lambda update-function-configuration \
  --function-name demo-lambda \
  --memory-size 1024
```

The allowed memory range and increments are defined by current Lambda quotas; check AWS documentation before selecting a value.

## Benchmark different settings

Test several memory values with the same input and load. Record:

- Average duration
- p95 and p99 duration
- Maximum memory used from the `REPORT` log line
- Error rate
- Cost per request
- Downstream latency

Example experiment table:

| Memory | Duration | Max memory used | Cost / request |
|---:|---:|---:|---:|
| 256 MB | Measure | Measure | Calculate |
| 512 MB | Measure | Measure | Calculate |
| 1024 MB | Measure | Measure | Calculate |
| 2048 MB | Measure | Measure | Calculate |

Do not treat these as expected results; populate the table from your own tests.

## Power Tuning

AWS Lambda Power Tuning is an open-source tool that can run a function at different memory configurations and help compare speed and cost. Review its permissions and deployment model before using it in an AWS account.

## Example optimization process

1. Capture a baseline under representative load.
2. Identify whether the function is CPU-bound, memory-bound, or waiting on I/O.
3. Test multiple memory sizes.
4. Compare latency and cost.
5. Choose the best trade-off for the workload.
6. Monitor after deployment.

## Best practices

- Do not automatically choose the smallest memory size.
- Do not assume more memory always means lower total cost.
- Use realistic test payloads and concurrency.
- Optimize code and downstream calls as well as memory.
- Watch maximum memory used to avoid out-of-memory failures.
