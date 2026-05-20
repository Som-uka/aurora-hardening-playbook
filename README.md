# Aurora Hardening Playbook

A structured approach to hardening Amazon Aurora/RDS clusters — covering security group containment, public accessibility remediation, and access control tightening, applied pilot-on-dev-first before production.

---

## Overview

This playbook documents the process of identifying and remediating over-exposed Aurora/RDS database clusters in a live AWS environment. The highest-risk finding was a production Aurora cluster exposed to the public internet via permissive security group rules — including open ingress on database and cache ports from `0.0.0.0/0`.

The remediation follows a **pilot-on-dev-first** approach: validate all steps on the development cluster, confirm no disruption, then apply to production with full rollback procedures in place.

---

## Risk Profile

| Risk | Detail |
|---|---|
| **Public Aurora access** | RDS/Aurora instances with `Publicly Accessible: Yes` |
| **Open security group** | Ingress on database/cache ports open to `0.0.0.0/0` |
| **Scope** | Production cluster (multi-AZ, writer + reader instances) |
| **Business impact** | Direct database exposure to the internet: highest severity finding |

---

## Remediation Phases

### Phase 1: Dev Cluster Pilot
- Identify all application layers connecting to the dev cluster
- Remove `0.0.0.0/0` ingress rules from attached security groups
- Replace with specific CIDR ranges or security group references
- Validate all application connections still function

### Phase 2: Staging Validation
- Repeat Phase 1 on staging cluster
- Load test and validate under simulated traffic

### Phase 3: Production Hardening
- Apply validated security group changes to production cluster
- Monitor CloudWatch for connection errors post-change
- Keep rollback commands ready for immediate execution

### Phase 4: Private Subnet Migration (Future)
- Move Aurora clusters to private subnets with no public route
- Access via Client VPN or bastion host only

---

## Security Group Hardening Template

```bash
# Document current ingress rules (pre-change state)
aws ec2 describe-security-groups \
  --group-ids <sg-id> \
  --query 'SecurityGroups[*].IpPermissions'

# Remove open ingress on database port
aws ec2 revoke-security-group-ingress \
  --group-id <sg-id> --protocol tcp \
  --port 3306 --cidr 0.0.0.0/0

# Replace with application-layer security group reference
aws ec2 authorize-security-group-ingress \
  --group-id <sg-id> --protocol tcp \
  --port 3306 --source-group <app-sg-id>

# Validate public accessibility
aws rds describe-db-instances \
  --query 'DBInstances[*].{ID:DBInstanceIdentifier,Public:PubliclyAccessible}'
```

---

## Repository Structure

```
aurora-hardening-playbook/
├── README.md
├── phases/
│   ├── phase-1-dev-pilot.md
│   ├── phase-2-staging.md
│   └── phase-3-production.md
├── change-records/
│   ├── CR-dev-sg-containment.md
│   └── CR-prod-sg-containment.md
└── scripts/
    ├── audit-rds-public-access.sh
    └── harden-security-group.sh
```

---

## Tech Stack

- AWS RDS, Aurora (MySQL-compatible), VPC, Security Groups, CloudWatch
- AWS CLI, Bash / PowerShell

> All resource identifiers, security group IDs, ARNs, and internal naming have been sanitized.
