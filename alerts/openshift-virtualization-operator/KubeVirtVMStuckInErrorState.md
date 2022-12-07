# KubeVirtVMStuckInErrorState
<!-- Edited by apinnick, Nov 2022 -->

## Meaning

This alert fires when a virtual machine (VM) is in an error state for more than
5 minutes.

Error states:

- CrashLoopBackOff
- Unknown
- Unschedulable
- ErrImagePull
- ImagePullBackOff
- PvcNotFound
- DataVolumeError

This alert might indicate an issue with the VM configuration, such as a missing
persistent volume claim, or a problem in the cluster infrastructure, such as
network disruptions or insufficient node resources.

## Impact

There is no immediate impact. However, if this alert persists, you should try
to resolve the issue.

## Diagnosis

1. Check the virtual machine instance (VMI) details:

   ```bash
   $ oc describe vmi <vmi> -n <namespace>
   ```

   Example output:

   ```yaml
   Name:          testvmi-hxghp
   Namespace:     kubevirt-test-default1
   Labels:        name=testvmi-hxghp
   Annotations:   kubevirt.io/latest-observed-api-version: v1
                  kubevirt.io/storage-observed-api-version: v1alpha3
   API Version:   kubevirt.io/v1
   Kind:          VirtualMachineInstance
   ...
   Spec:
     Domain:
   ...
       Resources:
         Requests:
           Cpu:     5000000Gi
           Memory:  5130000240Mi
   ...
   Status:
   ...
     Conditions:
       Last Probe Time:       2022-10-03T11:11:07Z
       Last Transition Time:  2022-10-03T11:11:07Z
       Message:               Guest VM is not reported as running
       Reason:                GuestNotRunning
       Status:                False
       Type:                  Ready
       Last Probe Time:       <nil>
       Last Transition Time:  2022-10-03T11:11:07Z
       Message:               0/2 nodes are available: 2 Insufficient cpu, 2
         Insufficient memory.
       Reason:                Unschedulable
       Status:                False
       Type:                  PodScheduled
     Guest OS Info:
     Phase:  Scheduling
     Phase Transition Timestamps:
       Phase:                        Pending
       Phase Transition Timestamp:   2022-10-03T11:11:07Z
       Phase:                        Scheduling
       Phase Transition Timestamp:   2022-10-03T11:11:07Z
     Qos Class:                       Burstable
     Runtime User:                    0
     Virtual Machine Revision Name:   revision-start-vm-3503e2dc-27c0-46ef-9167-7ae2e7d93e6e-1
   Events:
     Type    Reason            Age   From                       Message
     ----    ------            ----  ----                       -------
     Normal  SuccessfulCreate  27s   virtualmachine-controller  Created virtual
       machine pod virt-launcher-testvmi-hxghp-xh9qn
   ```

2. Check the node resources:

   ```bash
   $ oc get nodes -l node-role.kubernetes.io/worker= -o json | jq '.items | \
     .[].status.allocatable'
   ```

   Example output:

   ```json
   {
     "cpu": "5",
     "devices.kubevirt.io/kvm": "1k",
     "devices.kubevirt.io/sev": "0",
     "devices.kubevirt.io/tun": "1k",
     "devices.kubevirt.io/vhost-net": "1k",
     "ephemeral-storage": "33812468066",
     "hugepages-1Gi": "0",
     "hugepages-2Mi": "128Mi",
     "memory": "3783496Ki",
     "pods": "110"
   }
   ```

3. Check the node for error conditions:

   ```bash
   $ oc get nodes -l node-role.kubernetes.io/worker= -o json | jq '.items | \
     .[].status.conditions'
   ```

   Example output:

   ```json
   [
     {
       "lastHeartbeatTime": "2022-10-03T11:13:34Z",
       "lastTransitionTime": "2022-10-03T10:14:20Z",
       "message": "kubelet has sufficient memory available",
       "reason": "KubeletHasSufficientMemory",
       "status": "False",
       "type": "MemoryPressure"
     },
     {
       "lastHeartbeatTime": "2022-10-03T11:13:34Z",
       "lastTransitionTime": "2022-10-03T10:14:20Z",
       "message": "kubelet has no disk pressure",
       "reason": "KubeletHasNoDiskPressure",
       "status": "False",
       "type": "DiskPressure"
     },
     {
       "lastHeartbeatTime": "2022-10-03T11:13:34Z",
       "lastTransitionTime": "2022-10-03T10:14:20Z",
       "message": "kubelet has sufficient PID available",
       "reason": "KubeletHasSufficientPID",
       "status": "False",
       "type": "PIDPressure"
     },
     {
       "lastHeartbeatTime": "2022-10-03T11:13:34Z",
       "lastTransitionTime": "2022-10-03T10:14:30Z",
       "message": "kubelet is posting ready status",
       "reason": "KubeletReady",
       "status": "True",
       "type": "Ready"
     }
   ]
   ```

## Mitigation

Try to identify and resolve the issue.

If you cannot resolve the issue, log in to the
[Customer Portal](https://access.redhat.com) and open a support case,
attaching the artifacts gathered during the Diagnosis procedure.