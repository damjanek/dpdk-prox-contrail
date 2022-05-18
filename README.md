**Runbook DPDK Rapid test framework**

Rapid framework uses 3 VMs:
*  jump-vm - controls test execution. *No high speed VM required*, can be setup on kernel or DPDK Hypervisor
*  gen-vm - traffic generator VM responsible for traffic generation (requires large number of vCPUs)
*  swap-vm - loops traffic coming from gen-vm. This is the node which actually gets tested (requires large number of vCPUs, but less than gen-vm)

1. Upload Rapid image into glance

Get rapid VM image, then:

```
openstack image create --disk-format qcow2 --container-format bare --public --property hw_vif_multiqueue_enabled="true" --file rapidVM.qcow2 rapidVM-1908

```

Or if you're using ceph backend:

```
qemu-img convert rapidVM-1908.qcow2 rapidVM-1908.raw

openstack image create --disk-format raw --container-format bare --public --property hw_vif_multiqueue_enabled="true" --file rapidVM.raw rapidVM-1908

```

2. Copy example file to example-env.yaml, adjust and then create a stack:


```
cp example-env.yaml environment.yaml
vim environment.yaml
openstack stack create -t build-rapid.yml -e environment.yaml perf-test1
```

You can obtain IP address of jump VM with following:

```
openstack stack output show perf-test1-with-floating jump_ip -c output_value -f value
```

You can get configuration of deployed stack with:

```
openstack stack output show perf-test1 desc
```

Login credentials:
User: **root**
Pass: **c0ntrail123**

3. Execute tests

On Jump VM:

```
cd /root/prox/helper-scripts/rapid/
./runrapid.py --runtime <time> # replace <time> with time per one execution in seconds
```
You can edit scenarios by editing file "basicrapid.test"
where in section [test2] you can change/add number of flow and packet size
```
[test2]
test=flowsizetest
packetsizes=[64,128]
# the number of flows in the list need to be powers of 2, max 2^20
# Select from following numbers: 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768, 65535, 131072, 262144, 524280, 1048576
flows=[512,1]
```
