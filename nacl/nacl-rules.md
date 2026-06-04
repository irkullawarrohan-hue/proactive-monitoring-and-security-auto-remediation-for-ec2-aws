# VPC NACL Rules — CloudGuard Production Protection

## NACL Name
CloudGuard-Prod-Protection-NACL

## Inbound Rules

| Rule # | Protocol | Port Range | Source    | Action |
|--------|----------|------------|-----------|--------|
| 100    | TCP      | 1 - 1000   | Dev-IP/32 | DENY   |
| 200    | All      | All        | 0.0.0.0/0 | ALLOW  |

## Outbound Rules

| Rule # | Protocol | Port Range | Destination | Action |
|--------|----------|------------|-------------|--------|
| 100    | All      | All        | 0.0.0.0/0   | ALLOW  |

## Association
Associated with the Production subnet.

## Validation
Re-ran nmap port scan from Dev instance after NACL deployment — scan blocked.
