<center>
## rktlet: rkt as a Kubernetes CRI implementation
<img src="talk-rktlet/rkt-1.svg" width="100" />
</center>
​

<small>
Iago López Galeiras, Kinvolk
</small>



<center>
### About me
</center>
* Living in Berlin
* Kinvolk co-founder
* Worked a lot on rkt and the rktlet
* Currently a maintainer of rkt and the rktlet

```
* github: https://github.com/iaguis
* twitter: @iaguis
```
Note:
Kinvolk: for-hire Linux engineering team working on core cloud infrastructure components



<center>
### Plan
</center>
* What is rkt?
* How does rkt work? (rkt internals)
* What is the rktlet?
* How does the rktlet work?
* What's new?
* What's missing?



<center>
<img src="talk-rktlet/rkt-1.svg" width="100" />
</center>
<center>
a modern, secure, composable container runtime
</center>
Note:
* modern: try to take advantage of all of the latest technologies wherever possible
* thinking about the Linux kernel, we support user namespaces and the overlayfs filesystem for providing union filesystems
* secure: rkt always tries to take an approach of "secure by default", so things like image signing are enabled by default and need to be disabled rather than the other way around



<center>
<img src="talk-rktlet/rkt-1.svg" width="100" />
</center>
<center>
simple CLI tool<br />
golang + Linux<br />
self-contained<br />
init system/distro agnostic
</center>
Note:
* although we're init system agnostic we do unashamedly optimize for systemd



<center>
### simple CLI tool
</center>
```
bash/systemd/runit
 └── rkt (stage0)
      └── pod (stage1)
           ├── app1 (stage2)
           └── app2 (stage2)

```
<center>
<small>
* stage0
 * rkt binary (CLI)
 * discover, fetch, manage application images
 * set up pod filesystems
* stage1
 * exec environment for pods
 * process lifecycle management
 * resource constraints
* stage2
 * actual application

</small>

</center>
Note:
* Three different stages
* stage0: your typical CLI tool
* stage1: the actual container. Swappable.
* stage2: the app itself



<center>
### stage1 (swappable)
</center>
<small>
* default implementation
 * Linux namespaces + cgroups for isolation
 * based on `systemd-nspawn` + `systemd`
* kvm implementation
 * hardware virtualization for isolation
 * based on `qemu` + `systemd`
* fly
 * no isolation
 * just a `chroot`
* others

</small>



<center>
### kubernetes
</center>
<section data-background-image="talk-rktlet/k8s.svg" data-background-size="contain">
</section>
Note:
* Kubernetes is basically an API to schedule containers
* You don't care about where your app runs
* Scales horizontally
* etcd distributed key-value store
 * single source of truth for the cluster




<center>
### kubelet + CRI
</center>
<small>
* CRI defines a `Runtime` interface
 * `RunPodSandbox()`
 * `PodSandboxStatus()`
 * `CreateContainer()`
 * `StartContainer()`
 * `StopPodSandbox()`
 * `...`
* Kubelet calls those methods via gRPC
* Shims implement it
 * cri-containerd
 * cri-o
 * **rktlet**

</small>
Note:
* gRPC: remote procedure call by google
 * kubelet will call method
 * rktlet will be on the other side to do the work



<center>
### rktlet
</center>
The rkt implementation of a Kubernetes Container Runtime
</center>
<section data-background-image="talk-rktlet/rktlet-cri.svg" data-background-size="contain">
</section>
Note:



<center>
### rkt: app experiment
</center>
<small>
* The CRI operates in sandbox and container terms
* rkt had to be redesigned with that in mind
* New rkt subcommand: app
 * rkt app sandbox
 * rkt app add
 * rkt app start
 * ...
* For now under `RKT_EXPERIMENT_APP`

</small>
Note:
* These map pretty cleanly to systemd
 * add: create a new unit file and load it
 * start: start the unit file
 * etc.



<center>
### rkt: CRI logs
</center>
<small>
* rkt logging was journald
* CRI wants logs in custom text format
* Initial solution
 * journal2cri sidecar
  * Unreliable
  * Resource-consuming
* Proper solution
 * `RKT_EXPERIMENT_ATTACH`
 * iottymux
 * write directly into CRI format

</small>



<center>
### iottymux
</center>
</center>
<section data-background-image="talk-rktlet/iottymux.svg" data-background-size="contain">
</section>



<center>
### rkt: recent developments to accomodate rktlet
</center>
<small>
* Bugfixes, especially within the rkt experiments
* Integration between `RKT_EXPERIMENT_ATTACH` and k8s

</small>



<center>
### rkt: what comes next?
</center>
<small>
* Update CNI version to 0.6.0
* Stabilize rkt experiments and remove `RKT_EXPERIMENT_*` env variables
* Experiment with using `runc` to set up stage2 runtime
* Experiment with casync
* General bugfixing :)

</small>



<center>
### First casync tests
</center>
<section data-background-image="talk-rktlet/casync.svg" data-background-size="contain">
</section>
<center>
<small>
https://github.com/kinvolk/casync-measurements
</small>
</center>
Note:
* casync
 * combination of the rsync algorithm and content-addressable storage
 * efficient way to deliver and update container images over the Internet
 * Lennart's talk tomorrow



<center>
### rktlet: what comes next?
</center>
<small>
* Implement `kubectl attach`
* Implement CRI container stats
* Improve performance
* Support kubernetes v1.8.x
* Improve error propagation
* Pass more k8s e2e conformance tests
  * Current: 130 Passed | 15 Failed | 145 Total

</small>



<center>
# Demo
</center>
<center>
#### rktlet k8s cluster on kube-spawn
</center>
Note:
* dongsu's talk at ASG!



<center>
# Thank You!
</center>
<center>
## Questions?
</center>
Note:
* The end!
