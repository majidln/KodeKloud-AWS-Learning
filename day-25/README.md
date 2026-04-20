# Day 25: Setting Up an EC2 Instance and CloudWatch Alarm

Set up a **CloudWatch metric alarm** on **CPU utilization** for an EC2 instance and send notifications to an **Amazon SNS** topic when average CPU is at or above a threshold (for example **90%**).

**EC2:** This lab uses a **new EC2 instance** for monitoring. Steps to create or launch an instance are **not** included here—see [Day 06](../day-06/README.md), [Day 21](../day-21/README.md), or other compute-focused days for that.

**SNS:** You need an **SNS topic ARN** for `--alarm-actions` (create the topic in the console or with `aws sns create-topic` if your task requires it).

---

## Create the metric alarm

Required flags that are easy to miss:

- **`--namespace AWS/EC2`** — without it: *Namespace must not be blank*.
- **`--evaluation-periods`** — without it: *valid EvaluationPeriods parameter*.

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name nautilus-alarm \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=<instance-id> \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 90 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --unit Percent \
  --alarm-actions <sns-topic-arn> \
  --region <region>
```

- **`--period 300`**: 5-minute periods.
- **`--evaluation-periods`**: how many periods are evaluated (raise it if the lab wants sustained high CPU).
- **`--alarm-actions`**: SNS topic ARN when the alarm goes to **ALARM**.

---
