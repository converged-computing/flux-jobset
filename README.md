# Flux JobSet

This is a small experiment to package HPC applications in JobSet. Since we don't want to require exposing ssh from the containers, we are going to use flux. This requires a little extra prep, but it's fairly easy and worth it! For any of the applications in this repository, here are the setup and usage instructions.


## Usage

### 1. Create a Curve Certificate

You'll need to create a curve certificate. While we could reuse the same one, this is better to re-generate. The [Flux Operator](https://github.com/flux-framework/flux-operator) does this for you, and here we will use a container that uses flux.

```bash
# Generate a curve.cert in the present working directory
docker run -it --user root -v $PWD/:/home/fluxuser fluxrm/flux-sched:jammy flux keygen /home/fluxuser/curve.cert

# Create a curve.cert config map
kubectl create configmap curve-cert --from-file=curve-cert=curve.cert
```

### Create a workload

Each workload works by way of installing and configuring flux, and then running your application using it. The alternative is to use MPI, and (personally speaking) I find flux much better to orchestrate, etc. We first need to install JobSet:

```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/jobset/releases/download/v0.7.0/manifests.yaml
```

Try creating an interactive flux cluster:

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

That is done with just a Python base container! More HPC apps coming soon!

## License

HPCIC DevTools is distributed under the terms of the MIT license.
All new contributions must be made under this license.

See [LICENSE](https://github.com/converged-computing/cloud-select/blob/main/LICENSE),
[COPYRIGHT](https://github.com/converged-computing/cloud-select/blob/main/COPYRIGHT), and
[NOTICE](https://github.com/converged-computing/cloud-select/blob/main/NOTICE) for details.

SPDX-License-Identifier: (MIT)

LLNL-CODE- 842614

