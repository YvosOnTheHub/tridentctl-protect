# tridentctl-protect
Trident Protect is a solution developed by NetApp to protect stateful applications and Kubernetes based Virtual Images.  
It can perform application consistent snapshots, backups, as well as disaster recovery.  

A S3 compatible bucket is used to host the app metadata, as well as the data in the context of backups.  

Tridentctl-protect is the product cli that can:  
- interact with the Trident Protect controller. 
- list the content of a S3 bucket

The latter may be challenging as not all Kubernetes admins or application owners have access to the bucket... 

This repository contains the necesary files to create container images with the tridentctl-protect (v25.10) included:  
- [an interactive image](./Container-interactive/)
- [a scratch image](./Container-scratch/), with args passed in the YAML manifest

Images were tested with Kubernetes 1.29 as well as OpenShift 4.18.  
They need to be created in the _trident-protect_ namespace, in order to use the same _service account_.  
If you were planning on creating the pods in a different namespace, you would need to create the corresponding RBAC to get access to the AppVault.  

## A. Interactive image

You can find the YAML manifest in the Container-interactive folder:  
```bash
$ kubectl create -f Container-interactive/tridentctl-protect.yaml
pod/tridentctl-protect created

$ kubectl get -n trident-protect po tridentctl-protect
NAME                 READY   STATUS    RESTARTS   AGE
tridentctl-protect   1/1     Running   0          72s
```

You can now run your commands directly from the container:  
```bash
$ kubectl exec -n trident-protect tridentctl-protect -- /tridentctl-protect get appvaultcontent ontap-vault --show-resources all -n trident-protect
---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
| CLUSTER | APP  |   TYPE   |   NAME    | NAMESPACE |   STATE   | ERROR |          CREATED          |         COMPLETED         |
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
|         | bbox | snapshot | bboxsnap1 | bbox      | Completed |       | 2026-03-03 14:42:11 (UTC) | 2026-03-03 14:42:24 (UTC) |
|         | bbox | backup   | bboxbkp1  | bbox      | Completed |       | 2026-03-03 14:42:27 (UTC) | 2026-03-03 14:43:48 (UTC) |
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
```
or enter the pod in an interactive mode:  
```bash
$ kubectl exec -n trident-protect tridentctl-protect -it -- /bin/sh
/ # /tridentctl-protect get appvaultcontent ontap-vault --show-resources all -n trident-protect
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
| CLUSTER | APP  |   TYPE   |   NAME    | NAMESPACE |   STATE   | ERROR |          CREATED          |         COMPLETED         |
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
|         | bbox | snapshot | bboxsnap1 | bbox      | Completed |       | 2026-03-03 14:42:11 (UTC) | 2026-03-03 14:42:24 (UTC) |
|         | bbox | backup   | bboxbkp1  | bbox      | Completed |       | 2026-03-03 14:42:27 (UTC) | 2026-03-03 14:43:48 (UTC) |
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
```

## B. Scratch image

The goal of this image is to log the result of your command directly in the pod.  
You can find the YAML manifest in the Container-scratch folder.
You can notice that the parameters of the tridentctl-protect binary are all passed as _args_:  
```yaml
    command: ["/tridentctl-protect"]
    args: ["get","appvaultcontent","ontap-vault","--show-resources","all","-n","trident-protect"]
```
Let's create an instance of that pod and check the result:  
```bash
$ kubectl create -f Container-scratch/tridentctl-protect.yaml
pod/tridentctl-protect-scratch created

$ kubectl get -n trident-protect po tridentctl-protect-scratch
NAME                         READY   STATUS      RESTARTS   AGE
tridentctl-protect-scratch   0/1     Completed   0          14s
```
The pod is _completed_, you can get the results from its logs:  
```bash
$ kubectl logs -n trident-protect po/tridentctl-protect-scratch
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
| CLUSTER | APP  |   TYPE   |   NAME    | NAMESPACE |   STATE   | ERROR |          CREATED          |         COMPLETED         |
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
|         | bbox | snapshot | bboxsnap1 | bbox      | Completed |       | 2026-03-03 14:42:11 (UTC) | 2026-03-03 14:42:24 (UTC) |
|         | bbox | backup   | bboxbkp1  | bbox      | Completed |       | 2026-03-03 14:42:27 (UTC) | 2026-03-03 14:43:48 (UTC) |
+---------+------+----------+-----------+-----------+-----------+-------+---------------------------+---------------------------+
```