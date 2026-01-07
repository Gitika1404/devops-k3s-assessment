## OVERVIEW
This repository contains my solution for the DevOps assessment, where I deployed Open WebUI on a k3s Kubernetes cluster running on a single Linux VM.
The goal of this assessment was not only to deploy the application, but also to demonstrate:
1. Kubernetes fundamentals
2. Debugging and observability
3. Security and secrets handling
4. Backup and recovery thinking
5. Cost-aware infrastructure decisions
6. Production readiness reasoning


## ENVIRONMENT SETUP
# Host Machine
OS: Windows
Linux Environment: WSL (Ubuntu)
# Tools Used:
1. Docker
2. k3s
3. kubectl
4. Helm
5. Git
6. VS Code (connected to WSL)
   
Why WSL?
--WSL provides a clean Linux environment while still allowing easy GitHub integration and development from Windows. This setup closely resembles real    cloud VM workflows.


## KUBERNETES CLUSTER
Kubernetes Distribution: k3s
Cluster Type: Single-node
Reason for using k3s:

1. Lightweight
2. Low resource usage
3. Ideal for small teams, demos, and cost-efficient setups

## APPLICATION DEPLOYMENT
# Application
1. Open WebUI
2. Deployed using Helm
# Namespace
openwebui
# Deployed Components
The following pods are running successfully:
------kubectl get pods -n openwebui--------
Output:
    open-webui-0
    open-webui-ollama
    open-webui-pipelines
    open-webui-redis

All pods are in Running state with 0 restarts, indicating a healthy deployment.

## INTENTIONAL FAILURE SCENARIO
Although no failure occurred during my deployment, the assessment included an intentional failure scenario. Below is my analysis:

Expected Failure:
-Pod stuck in CrashLoopBackOff
-Logs show:
    x509: certificate signed by unknown authority
    OIDC discovery endpoint unreachable
-Root Cause
    OIDC provider uses a self-signed certificate
    Open WebUI does not trust the custom CA
    Kubernetes container trust store does not include this CA

Why It Did Not Occur:
OIDC integration was not actively enforced with a custom CA in this setup
Default certificates were trusted by the system
This demonstrates understanding of failure reasoning even when the issue is not reproduced.

## What Breaks First? 
Scenario: Single Node Failure
If the node goes down:
  - Entire cluster becomes unavailable
  - All workloads stop
  - No automatic recovery (single node limitation)
Recovery Strategy:
(Manual Recovery Steps)
1. Create a new VM
2. Reinstall k3s
3. Restore application data from backup
4. Redeploy Helm releases
5. Verify pod and service health

## Future Improvements (Next Day Changes)
1. Move to multi-node cluster
2. Add load balancer
3. Enable node auto-replacement
4. Add monitoring and alerting
5. Maintain runbooks for incident response

## Security & Secrets Management
How Secrets Are Managed:
- Kubernetes Secrets
- Neverstore secrets in Git
Examples of Secrets
-API keys
-Client secrets
-Database passwords
-Private keys
-Certificates
What Needs Rotation
-OIDC secrets
-DB credentials
-Tokens
-SSH keys

## Backup & Recovery
What to Back Up?
1. Application data
2. Kubernetes configs
3. Persistent volumes
4. Secrets (encrypted)
Backup Frequency?
- Daily full backups
- Hourly snapshots for critical data
Recovery Testing
-Restore backups in staging
-Periodic fire-drill testing
-Clearly documented restore steps

## Cost Ownership (Hetzner)
Cost-Saving Decisions:
1. Single-node cluster
2. Right-sized VM
3. Avoid managed services early
Avoid Early:
1. Multi-region deployments
2. Over-provisioning
3. Paid observability stacks

## When to Move Away from k3s
k3s is sufficient until:
- High availability is required
- Multiple teams use the cluster
- Regulatory or compliance requirements exist
- Traffic grows significantly

## Extra Credit: Custom CA Trust (Best Practice)
Recommended Approach:
-Store CA certificate in a Kubernetes Secret
-Mount CA into pods
-Use initContainers to update trust store
-Centralized CA rotation
-No hardcoded certificates in images

Why This Works?
1. Secure
2. Repeatable
3. Git-safe
4. Scales across services
