# Troubleshooting with Kubernetes

## Deployment

### Stuck Deployment Pipeline (Helm)
**Error:** UPGRADE FAILED: another operation (install/upgrade/rollback) is in progress

**Cause:** A parallel or aborted pipeline disturbs the deployment 

```shell
# login into kubernetes env
# ...

# list resources in namespace
helm ls -a -n my-namespace

# show deployment history of my-service
helm history my-service -n my-namespace

# rollback stuck deployment
helm rollback my-service 123 -n my-namespace

```