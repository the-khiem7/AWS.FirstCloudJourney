---
title: "Mountpoint for Amazon S3 CSI driver v2: Accelerated performance and improved resource usage for Kubernetes workloads"
date: 2025-08-06
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# **Mountpoint for Amazon S3 CSI driver v2: Accelerated performance and improved resource usage for Kubernetes workloads**

by Ran Pergamin, Ankit Kalyani, Burak Varli, and Dmitry Nutels on 06 AUG 2025 in [Advanced (300)](https://aws.amazon.com/blogs/storage/category/advanced-300/), [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/), [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/), [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/), [Open Source](https://aws.amazon.com/opensource/), [Storage](https://aws.amazon.com/storage/), [Technical How-to](https://aws.amazon.com/blogs/storage/category/how-to/) [Permalink](https://aws.amazon.com/blogs/storage/mountpoint-for-amazon-s3-csi-driver-v2-accelerated-performance-and-improved-resource-usage-for-kubernetes-workloads/)

[Amazon S3](https://aws.amazon.com/s3/) is the best place to build data lakes because of its durability, availability, scalability, and security. In 2023, we introduced [Mountpoint for Amazon S3](https://aws.amazon.com/s3/features/mountpoint/), an open source file client that allows Linux-based applications to access S3 objects through a file API. Shortly after, we took this one step further with the [Mountpoint for Amazon S3 Container Storage Interface (CSI) driver](https://github.com/awslabs/mountpoint-s3-csi-driver) for containerized applications. This enabled you to access S3 objects from your Kubernetes applications through a file system interface.

Users appreciated the high aggregate throughput that this CSI driver offered. However, you asked for a way to run Mountpoint processes inside containers (not on the host) and without elevated root permissions. This redesign provides three key benefits: (1) free up `systemd` resources on the host for other operational needs; (2) let pods on the same node share the Mountpoint pod and its cache. By using the new caching capabilities in Mountpoint for Amazon S3 CSI driver v2, you can finish large-scale financial simulation jobs up to 2x faster by eliminating the overhead of multiple pods individually caching the same data; and (3) compatibility with Security-Enhanced Linux (SELinux) enabled Kubernetes environments such as Red Hat OpenShift. That is why we released the first major upgrade: Mountpoint for Amazon S3 CSI driver v2.

In this post, we dive into the core components of this driver, the key benefits of using it to access S3 from your Kubernetes applications, and a real-world illustration of how to use it.

## **How we built the Mountpoint for Amazon S3 CSI driver v2**

We built this CSI driver with the following three main components (shown in the diagram below):

1. **CSI driver node:** Deployed as a DaemonSet on each node. This component implements the CSI Node Service RPC and handles mount operations. It communicates with the kubelet and manages the lifecycle of Mountpoint instances.
2. **CSI driver controller:** Schedules a Mountpoint pod on the appropriate node for each workload pod that uses S3 volumes. It uses a custom resource called `MountpointS3PodAttachment` to track which workloads are assigned to which Mountpoint pods.
3. **CSI driver Mounter/Mountpoint pod:** These pods run the Mountpoint instances. They receive mount options from the CSI Driver Node and spawn Mountpoint processes within their containers.

![Architecture diagram showing 3 main components of the CSI driver v2](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/Architecture.png)

## **Overview of the key benefits**

Mountpoint for Amazon S3 CSI driver v2 offers the following key benefits:

- **Efficient resource usage for nodes** in your Kubernetes cluster when pods on the same [Amazon EC2](https://aws.amazon.com/ec2/) instance share a common [Mountpoint pod](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/MOUNTPOINT_POD_SHARING.md).
- **Optimized performance and reduced cost** for repeatedly accessed data when pods share the local cache.
- **Streamlined credential management with EKS Pod Identity** provided by [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/).
- **Compatibility with SELinux requirements**, enabling regulatory compliance and supporting Red Hat OpenShift environments.
- **Improved observability** with Mountpoint pod logs available via `kubectl logs`, plus optional [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) integration through the [CloudWatch Observability EKS add-on](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-EKS-addon.html).

In the following section, we explore how this driver improves resource usage, accelerates performance, and streamlines credential management.

## **Efficient resource usage for Kubernetes nodes**

Mountpoint for Amazon S3 CSI driver v2 allows you to run a single [Mountpoint instance](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/MOUNTPOINT_POD_SHARING.md) and share it among multiple worker pods (as shown in the figure) on the same node when those pods have [identical configuration parameters](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/MOUNTPOINT_POD_SHARING.md#how-mountpoint-pod-sharing-works) such as namespace, service account name, or source of authentication. This improves resource usage in your Kubernetes clusters so you can launch more pods per node.

![Mountpoint for Amazon S3 pod instance sharing](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/PodSharing.png)

## **Optimized performance and reduced cost for repeatedly accessed data**

The driver lets you locally cache frequently accessed data and share that cache across pods on the same node. When you [configure caching](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/CACHING.md), it automatically creates the cache folder for you. You can control cache size and storage characteristics, choosing to cache on either an [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) volume (using the node’s default storage medium) or a [generic ephemeral volume](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/). By using these new caching capabilities, you can complete large-scale financial simulation jobs up to 2x faster by eliminating the overhead of multiple pods individually caching the same data.

## **Streamlined credential management with EKS Pod Identity**

This major upgrade adds support for [EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html), a simpler and more scalable alternative to [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) to manage access policies across EKS clusters, including cross-account access. EKS Pod Identity can be installed as an [Amazon EKS add-on](https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html).

When you use EKS Pod Identity, you do not need to create a separate [OpenID Connector (OIDC)](https://openid.net/developers/how-connect-works/) provider per cluster. The namespace and the service account do not have to exist ahead of time, which simplifies onboarding. EKS Pod Identity also supports attaching [session tags](https://docs.aws.amazon.com/eks/latest/userguide/pod-id-abac.html) to the temporary credentials associated with a service account, enabling [Attribute-based access control (ABAC)](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html). Pod Identity provides tags for namespace, `kubernetes-service-account`, and `eks-cluster-name` so you can control access at the desired levels of granularity.

The driver can be configured with two types of credentials:

- **Driver-level credentials**: a single set of permissions associated with the service account of the driver, used for pods in the cluster.
- **Pod-level credentials**: credentials associated with each pod’s service account, which configure access permissions for that pod.

Using pod-level credentials with EKS Pod Identity and session tags enables scalable, multi-tenant solutions. For example, with EKS Pod Identity, you can define clauses in [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) policies attached to the IAM role associated with a service account using session tags:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": [
        "arn:aws:s3:::some-bucket/s3mp-apps-data/shared/*",
        "arn:aws:s3:::some-bucket/s3mp-apps-data/${aws:PrincipalTag/kubernetes-namespace}/*"
      ]
    }
  ]
}
```

## **Example usage patterns for Mountpoint for Amazon S3 CSI driver v2**

The following example demonstrates the driver in action. It assumes the driver is installed and configured to access the corresponding S3 bucket and prefixes using EKS Pod Identity or IRSA.

### **Creating a PersistentVolume**

To let applications handle the files in the S3 bucket as if they were in a local directory, create a PersistentVolume (the driver currently supports Kubernetes [static provisioning](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/CONFIGURATION.md#static-provisioning)):

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: s3mp-data-pv
spec:
  storageClassName: '' # Required for static provisioning
  capacity:
    storage: 1Gi # Ignored, required
  accessModes:
    - ReadWriteMany
  claimRef: # To ensure no other PVCs can claim this PV
    namespace: apps
    name: s3mp-data-pvc
  mountOptions:
    - region: eu-central-1
    - prefix data/
  csi:
    driver: s3.csi.aws.com # Required
    volumeHandle: my-s3-data-volume # Must be unique
    volumeAttributes:
      bucketName: mp-s3-bucket-<some unique suffix>
      cache: emptyDir
      cacheEmptyDirSizeLimit: 1Gi
      cacheEmptyDirMedium: Memory
```

The PersistentVolume defines:

- Storage capacity (mandated by the CSI spec but unused)
- Access modes
- The bucket (name and region) to use
- The size and location of the cache used by the driver’s Mountpoint pods

### **Using the PersistentVolume**

Now create an application that uses the preceding PersistentVolume:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: s3mp-data-pvc
  namespace: apps
spec:
  storageClassName: ''
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: s3mp-data-pv
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: some-app
  namespace: apps
spec:
  replicas: 2
  selector:
    matchLabels:
      name: some-app
  template:
    metadata:
      labels:
        name: some-app
    spec:
      nodeName: ip-10-0-82-168.eu-central-1.compute.internal
      containers:
        - name: app
          image: busybox
          command: ['sh', '-c', 'trap : TERM INT; sleep infinity & wait']
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: s3mp-data-pvc
```

This defines:

- A PersistentVolumeClaim that uses the PersistentVolume
- A requested storage capacity (required by the CSI spec but unused)
- A `storageClassName` that must match the PersistentVolume
- An application that mounts the PersistentVolumeClaim under `/data`

We forced both application replicas onto the same worker node to demonstrate the caching capability.

### **Verifying the deployment**

The created PV and PVC and the deployed application look similar to the following (with some data abridged for clarity):

```
$ kubectl get pods -n apps -o wide

NAME                         READY   STATUS    AGE     IP            NODE                                      
some-app-556c9447fd-sjf4k   1/1     Running   3m14s   10.0.94.129   ip-10-0-85-106.eu-central-1.compute.internal
some-app-556c9447fd-wkpbv   1/1     Running   3m14s   10.0.89.177   ip-10-0-85-106.eu-central-1.compute.internal

$ kubectl get pv,pvc -n apps

NAME                         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/s3mp-data-pv   1Gi     RWX            Retain           Bound    apps/s3mp-data-pvc                                      <unset>           4m

NAME                             STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/s3mp-data-pvc   Bound   s3mp-data-pv   1Gi        RWX
```

When the application is deployed, the driver creates a Mountpoint pod on each node where the pods are running (a single one in this case) to set up pod sharing and allow other pods on the node to access the local cache.

```
$ kubectl get pods -n mount-s3 -o wide

NAME       READY   STATUS    AGE     IP            NODE                                      
mp-xgscz   1/1     Running   3m48s   10.0.94.38    ip-10-0-85-106.eu-central-1.compute.internal
```

### **Using the cache**

Access one of the files in the mounted bucket’s `data/` prefix to see caching in action by downloading and counting words in a 2.5 MB file:

```
$ POD_ID=$(kubectl get pod -n apps -l name=some-app -o jsonpath='{.items[0].metadata.name}')
$ echo ${POD_ID}

some-app-556c9447fd-sjf4k

$ kubectl exec -it -n apps ${POD_ID} -- time wc /data/some-file

  129419    130195   2424064 /data/some-file
 real    0m 0.22s
 user    0m 0.01s
 sys     0m 0.00s

$ POD_ID=$(kubectl get pod -n apps -l name=some-app -o jsonpath='{.items[1].metadata.name}')
$ echo ${POD_ID}

some-app-556c9447fd-wkpbv

$ kubectl exec -it -n apps ${POD_ID} -- time wc /data/some-file

  129419    130195   2424064 /data/some-file
 real    0m 0.09s
 user    0m 0.01s
 sys     0m 0.00s
```

Because the word counting on the same instance takes roughly the same time, you can see the difference the cache makes in the `real` times above.

## **Upgrading from Mountpoint for Amazon S3 CSI driver v1 to v2**

To install the driver, follow the [installation guide](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/INSTALL.md). If you are upgrading from v1, note that the new release changes how volumes are configured. Review the [list of configuration changes](https://github.com/awslabs/mountpoint-s3-csi-driver/blob/main/docs/UPGRADING_TO_V2.md) before upgrading so you can adjust accordingly.

At the time of writing, v2 has some constraints when working with node autoscalers such as [Karpenter](https://karpenter.sh) and [Cluster Autoscaler](https://docs.aws.amazon.com/eks/latest/best-practices/cas.html). See the related [GitHub issue](https://github.com/awslabs/mountpoint-s3-csi-driver/issues/543) for details.

## **Conclusion**

Mountpoint for Amazon S3 CSI driver v2 improves how Kubernetes applications interact with S3 data: pod sharing for better resource usage and speed, SELinux support, logging via `kubectl`, and simpler access management with Amazon EKS Pod Identity. The driver is available on [GitHub](https://github.com/awslabs/mountpoint-s3-csi-driver); we welcome your feedback and contributions.

Tags: [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/), [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/), [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/), [Open Source](https://aws.amazon.com/opensource/)

**![Ran Pergamin](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2023/06/15/ran-pergamin.jpg)**

**Ran Pergamin**  
Ran Pergamin is a Software Engineer at AWS working on Amazon S3, focused on building storage solutions and large-scale data access tools.

**![Ankit Kalyani](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/Ankit-Kalyani.png)**

**Ankit Kalyani**  
Ankit Kalyani is a Senior Software Engineer at AWS specializing in distributed systems and storage, with a focus on performance and scalability for Amazon S3.

**![Burak Varli](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/Burak-Varli.png)**

**Burak Varli**  
Burak Varli is a Senior Solutions Architect at AWS who helps customers build and optimize storage workloads, especially in Kubernetes and containerized environments.

**![Dmitry Nutels](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2022/11/18/passport.png)**

**Dmitry Nutels**  
Dmitry Nutels is a Principal Software Engineer at AWS leading development of storage and large-scale data access tools, including Mountpoint for Amazon S3 and related CSI technologies.
