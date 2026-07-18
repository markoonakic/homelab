# Internal Longhorn backup target

MinIO is pinned to `talos-dy9-gja` and stores data directly in
`/var/lib/backup-target` on that node. This is deliberately outside
Longhorn. The ClusterIP service has no ingress route, and the one-shot Job
idempotently creates only `longhorn-backups`.

The 10 GiB `ephemeral-storage` reservation and probes require node free space
to exceed 10 GiB plus 25% of `/var`, and require the data directory to remain
under 10 GiB. Kubernetes does not account hostPath bytes against an ephemeral
storage limit, so the reservation and limit are advisory; the liveness probe
provides the native stop policy. A breach emits an `Unhealthy` event, restarts
MinIO, and the startup check keeps it stopped until capacity is recovered.

Recovery is node-local: retain `/var/lib/backup-target`, restore or
rejoin `talos-dy9-gja` with that persistent `/var`, then reconcile this path.
If the directory is intentionally moved to another node, update `nodeName` and
the hostPath together only after copying and validating its contents.

Rollback (not executed):

```sh
flux suspend kustomization infrastructure -n flux-system
kubectl delete namespace backup-target
flux resume kustomization infrastructure -n flux-system
```

Namespace deletion removes Kubernetes objects but not the host data directory.
Delete `/var/lib/backup-target` separately only after its data is no
longer required.
