# Flux

This example just creates a JobSet with an interactive Flux cluster. You can use this as a bare bones example for your own application. Note that the base container can be changed - the only requirement to install Flux is to be able to install Python. See the [README](../README.md) to install JobSet, and then try creating an interactive flux cluster:

```bash
kubectl apply -f flux/job.yaml
```
You'll have an interactive flux cluster!

```bash
kubectl logs flux-worker-0-0-xxx
```
```console
...
/opt/conda/bin/flux proxy local:///var/run/flux/local bash
🌀 flux broker --config-path /opt/conda/etc/flux/system/conf.d/broker.toml -Scron.directory=/opt/conda/etc/flux/system/cron.d -Stbon.fanout=256 -Srundir=/var/run/flux -Sbroker.rc2_none -Sstatedir=/opt/conda/etc/flux/system -Slocal-uri=local:///var/run/flux/local -Slog-stderr-level=6 -Slog-stderr-mode=local
This is flux-worker-0-0
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.244.3.16	flux-worker-0-0.m.default.svc.cluster.local	flux-worker-0-0
broker.info[0]: start: none->join 0.310499ms
broker.info[0]: parent-none: join->init 0.008727ms
cron.info[0]: synchronizing cron tasks to event heartbeat.pulse
job-manager.info[0]: restart: 0 jobs
job-manager.info[0]: restart: 0 running jobs
job-manager.info[0]: restart: checkpoint.job-manager not found
broker.info[0]: rc1.0: running /opt/conda/etc/flux/rc1.d/01-sched-fluxion
sched-fluxion-resource.info[0]: version 0.38.0
sched-fluxion-resource.warning[0]: create_reader: allowlist unsupported
sched-fluxion-resource.info[0]: populate_resource_db: loaded resources from core's resource.acquire
sched-fluxion-qmanager.info[0]: version 0.38.0
broker.info[0]: rc1.0: running /opt/conda/etc/flux/rc1.d/02-cron
broker.info[0]: rc1.0: /opt/conda/etc/flux/rc1 Exited (rc=0) 0.4s
broker.info[0]: rc1-success: init->quorum 0.432494s
broker.info[0]: online: flux-worker-0-0 (ranks 0)
broker.info[0]: online: flux-worker-0-[0-3] (ranks 0-3)
broker.info[0]: quorum-full: quorum->run 0.591215s
```

That's it! Shell in and play around. Likely I'll add a script to make it easy to connect to the lead broker, but I haven't yet.
