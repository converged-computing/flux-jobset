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

That is done with just a Python base container! Note that if you want to interactively connect to a broker, a script
is provided:

```bash
/flux-connect.sh
# try "flux resource list"
```

You can also run an app, which will complete:

```bash
kubectl apply -f lammps/job.yaml
```
```console
Setting up Verlet run ...
  Unit style    : real
  Current step  : 0
  Time step     : 0.1
Per MPI rank memory allocation (min/avg/max) = 215.0 | 215.0 | 215.0 Mbytes
Step Temp PotEng Press E_vdwl E_coul Volume 
       0          300   -113.27833    437.52122   -111.57687   -1.7014647    27418.867 
      10    299.38517   -113.27631    1439.2857   -111.57492   -1.7013813    27418.867 
      20    300.27107   -113.27884    3764.3739   -111.57762   -1.7012246    27418.867 
      30    302.21063   -113.28428    7007.6914   -111.58335   -1.7009363    27418.867 
      40    303.52265   -113.28799      9844.84   -111.58747   -1.7005186    27418.867 
      50    301.87059   -113.28324    9663.0443   -111.58318   -1.7000524    27418.867 
      60    296.67807   -113.26777    7273.7928   -111.56815   -1.6996137    27418.867 
      70    292.19999   -113.25435    5533.6428   -111.55514   -1.6992157    27418.867 
      80    293.58677   -113.25831    5993.4151   -111.55946   -1.6988533    27418.867 
      90    300.62636   -113.27925    7202.8651   -111.58069   -1.6985591    27418.867 
     100    305.38276   -113.29357    10085.748   -111.59518   -1.6983875    27418.867 
Loop time of 17.2465 on 1 procs for 100 steps with 2432 atoms

Performance: 0.050 ns/day, 479.070 hours/ns, 5.798 timesteps/s
99.9% CPU use with 1 MPI tasks x 1 OpenMP threads

MPI task timing breakdown:
Section |  min time  |  avg time  |  max time  |%varavg| %total
---------------------------------------------------------------
Pair    | 12.723     | 12.723     | 12.723     |   0.0 | 73.77
Neigh   | 0.21998    | 0.21998    | 0.21998    |   0.0 |  1.28
Comm    | 0.0074962  | 0.0074962  | 0.0074962  |   0.0 |  0.04
Output  | 0.00055908 | 0.00055908 | 0.00055908 |   0.0 |  0.00
Modify  | 4.2943     | 4.2943     | 4.2943     |   0.0 | 24.90
Other   |            | 0.0008124  |            |       |  0.00

Nlocal:        2432.00 ave        2432 max        2432 min
Histogram: 1 0 0 0 0 0 0 0 0 0
Nghost:        10685.0 ave       10685 max       10685 min
Histogram: 1 0 0 0 0 0 0 0 0 0
Neighs:        823958.0 ave      823958 max      823958 min
Histogram: 1 0 0 0 0 0 0 0 0 0

Total # of neighbors = 823958
Ave neighs/atom = 338.79852
Neighbor list builds = 5
Dangerous builds not checked
Total wall time: 0:00:17
broker.info[0]: rc2.0: /opt/conda/bin/flux submit --quiet --watch lmp -v x 2 -v y 2 -v z 2 -in in.reaxc.hns -nocite Exited (rc=0) 18.3s
broker.info[0]: rc2-success: run->cleanup 18.2654s
broker.info[0]: cleanup.0: flux queue stop --quiet --all --nocheckpoint Exited (rc=0) 0.1s
broker.info[0]: cleanup.1: flux resource acquire-mute Exited (rc=0) 0.1s
broker.info[0]: cleanup.2: flux cancel --user=all --quiet --states RUN Exited (rc=0) 0.1s
broker.info[0]: cleanup.3: flux queue idle --quiet Exited (rc=0) 0.1s
broker.info[0]: cleanup-success: cleanup->shutdown 0.322195s
broker.info[0]: children-complete: shutdown->finalize 82.5735ms
broker.info[0]: rc3.0: running /opt/conda/etc/flux/rc3.d/01-sched-fluxion
broker.info[0]: rc3.0: /opt/conda/etc/flux/rc3 Exited (rc=0) 0.2s
broker.info[0]: rc3-success: finalize->goodbye 0.194591s
broker.info[0]: goodbye: goodbye->exit 0.030218ms
Return value for follower worker is 0
🤓 Success! Cleaning up
```
```console
$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
lammps-worker-0-0-xgl82   0/1     Completed   0          102s
lammps-worker-0-1-5hjmf   0/1     Completed   0          102s
lammps-worker-0-2-6smhp   0/1     Completed   0          102s
lammps-worker-0-3-c2bft   0/1     Completed   0          102s
```

More HPC apps coming soon!

## License

HPCIC DevTools is distributed under the terms of the MIT license.
All new contributions must be made under this license.

See [LICENSE](https://github.com/converged-computing/cloud-select/blob/main/LICENSE),
[COPYRIGHT](https://github.com/converged-computing/cloud-select/blob/main/COPYRIGHT), and
[NOTICE](https://github.com/converged-computing/cloud-select/blob/main/NOTICE) for details.

SPDX-License-Identifier: (MIT)

LLNL-CODE- 842614

