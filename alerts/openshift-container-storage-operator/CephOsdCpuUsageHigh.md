# OSDCpuUsageHigh

## Meaning

Ceph OSD CPU usage for the OSD daemon has exceeded the threshold for adequate
performance.

## Impact

Ceph OSD is a disk io daemon. It is required for every ceph io operation.
This alert means the default CPU allocation is unable to cope with load and we need to
increase the CPU allocation.

## Diagnosis

To diagnose the alert, click on the workloads->pods and select the
corresponding OSD pod and click on the metrics tab.
You should be able to see the allocated and used CPU. By default,
the alert is fired if the used CPU is 80% of allocated CPU for 30 mins
If this is the case take the steps mentioned in mitigation.

## Mitigation

We need to increase the allocated CPU. The below command describes
how to set the number of allocated CPU for OSD server.

```bash
oc patch -n openshift-storage storagecluster ocs-storagecluster \
    --type merge \
    --patch '{"spec": {"resources": {"osd": {"limits": {"cpu": "8"},
    "requests": {"cpu": "8"}}}}}'
```
