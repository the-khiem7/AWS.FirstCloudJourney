---
title: "Mountpoint for Amazon S3 CSI driver v2: Accelerated performance and improved resource usage for Kubernetes workloads"
date: 2025-08-06
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# **Mountpoint for Amazon S3 CSI driver v2: Accelerated performance and improved resource usage for Kubernetes workloads**

by Ran Pergamin, Ankit Kalyani, Burak Varli, and Dmitry Nutels on 06 AUG 2025 in [Advanced (300)](https://aws.amazon.com/blogs/storage/category/advanced-300/), [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/), [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/), [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/), [Open Source](https://aws.amazon.com/opensource/), [Storage](https://aws.amazon.com/storage/), [Technical How-to](https://aws.amazon.com/blogs/storage/category/how-to/) [Permalink](https://aws.amazon.com/blogs/storage/mountpoint-for-amazon-s3-csi-driver-v2-accelerated-performance-and-improved-resource-usage-for-kubernetes-workloads/) 

[Amazon S3](https://aws.amazon.com/s3/) là nơi tốt nhất để xây dựng data lake nhờ độ bền, khả dụng, khả năng mở rộng và bảo mật. Năm 2023, chúng tôi giới thiệu [Mountpoint for Amazon S3](https://aws.amazon.com/s3/mountpoint/), một file client mã nguồn mở cho phép các ứng dụng trên Linux truy cập S3 objects thông qua file API. Ngay sau đó, chúng tôi tiến thêm một bước với [Mountpoint for Amazon S3 Container Storage Interface (CSI) driver](https://github.com/awslabs/aws-mountpoint-csi-driver) cho các ứng dụng containerized.

Cách tiếp cận này giúp bạn truy cập S3 objects từ ứng dụng Kubernetes thông qua giao diện file system. Người dùng đánh giá cao **aggregate throughput** cao mà CSI driver này mang lại. Tuy nhiên, bạn đã yêu cầu một cách chạy Mountpoint process **bên trong containers** (không phải trên host) và **không cần quyền root nâng cao**. Thiết kế lại này có ba lợi ích chính: (1) giải phóng tài nguyên systemd của host cho các nhu cầu vận hành khác; (2) cho phép các pod trên cùng một node chia sẻ Mountpoint pod và **cache** của nó; (3) tương thích với các môi trường Kubernetes bật **SELinux** như Red Hat OpenShift. Vì vậy, chúng tôi đã phát hành bản nâng cấp lớn đầu tiên: **Mountpoint for Amazon S3 CSI driver v2**.

Trong bài viết này, chúng tôi đi sâu vào các thành phần cốt lõi của driver, các lợi ích chính khi dùng nó để truy cập S3 từ ứng dụng Kubernetes, và một minh họa thực tế về cách sử dụng.

## **Cách chúng tôi xây dựng Mountpoint for Amazon S3 CSI driver v2**

Chúng tôi xây dựng CSI driver này với ba thành phần chính sau (cũng được minh họa trong hình bên dưới):

1. **CSI driver node**: Triển khai dạng DaemonSet trên mỗi node. Thành phần này triển khai CSI Node Service RPC và xử lý các thao tác mount; giao tiếp với kubelet và quản lý vòng đời các Mountpoint instance.

2. **CSI driver controller**: Lập lịch một Mountpoint pod trên node phù hợp cho mỗi workload pod sử dụng S3 volumes. Thành phần này dùng một custom resource tên là MountpointS3PodAttachment để theo dõi workload nào được gán cho Mountpoint pod nào.

3. **CSI driver Mounter/Mountpoint pod**: Các pod này thực thi các Mountpoint instance; nhận mount options từ CSI Driver Node và khởi chạy Mountpoint processes bên trong container của chúng.

![Architecture diagram showing 3 main components of the CSI driver v2](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/Architecture.png)

[Hình: Architecture diagram hiển thị 3 thành phần chính của CSI driver v2](https://aws.amazon.com/blogs/storage/mountpoint-for-amazon-s3-csi-driver-v2-accelerated-performance-and-improved-resource-usage-for-kubernetes-workloads/)

## **Tổng quan về các lợi ích chính**

Mountpoint for Amazon S3 CSI driver v2 mang lại các lợi ích nổi bật sau:

* **Efficient resource usage** cho các node trong Kubernetes cluster khi các pod trên cùng một [Amazon EC2](https://aws.amazon.com/ec2/) instance (tức một node) có thể chia sẻ chung một [Mountpoint pod](https://github.com/awslabs/aws-mountpoint-csi-driver).

* **Optimized performance và giảm chi phí** cho dữ liệu được truy cập lặp lại khi các pod chia sẻ cache cục bộ.

* **Streamlined credential management** với EKS Pod Identity bởi [Amazon Elastic Kubernetes Service (Amazon EKS)](https://aws.amazon.com/eks/).

* **Tương thích với SELinux**, hỗ trợ tuân thủ quy định và các môi trường Red Hat OpenShift.

* **Cải thiện quan sát (observability)** với log của Mountpoint pod có sẵn qua công cụ dòng lệnh Kubernetes tiêu chuẩn như kubectl logs. Bạn cũng có thể bật tích hợp [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) để tập trung logging với [CloudWatch Observability EKS add-on](https://docs.aws.amazon.com/eks/latest/userguide/cloudwatch.html).

## **Tối ưu sử dụng tài nguyên cho các node trong Kubernetes cluster của bạn**

Mountpoint for Amazon S3 CSI driver v2 cho phép bạn chạy **một** [Mountpoint instance](https://github.com/awslabs/aws-mountpoint-csi-driver) duy nhất và chia sẻ nó giữa nhiều worker pods (như hình minh họa) trên cùng một node khi các pod đó có [các tham số cấu hình giống hệt](https://github.com/awslabs/aws-mountpoint-csi-driver#pod-sharing) như namespace, service account name, hoặc nguồn xác thực. Cách này cải thiện sử dụng tài nguyên trong Kubernetes cluster để bạn có thể chạy nhiều pod hơn trên mỗi node.

![Hình: Mountpoint for Amazon S3 pod instance sharing](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/PodSharing.png)

[Hình: Mountpoint for Amazon S3 pod instance sharing](https://aws.amazon.com/blogs/storage/mountpoint-for-amazon-s3-csi-driver-v2-accelerated-performance-and-improved-resource-usage-for-kubernetes-workloads/)

## **Tối ưu hiệu năng và giảm chi phí cho dữ liệu được truy cập lặp lại**

Driver cho phép **cache cục bộ** dữ liệu được truy cập lặp lại và **chia sẻ cache** giữa các pod trên cùng node (như hình ở phần trước). Khi bạn [cấu hình caching](https://github.com/awslabs/aws-mountpoint-csi-driver#caching), driver tự động tạo thư mục cache cho bạn; cho phép điều khiển kích thước cache và đặc tính lưu trữ; bạn có thể chọn cache trên [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) (dùng storage mặc định của node) hoặc [generic ephemeral volume](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#generic-ephemeral-volumes).

Nhờ khả năng caching mới trong Mountpoint for Amazon S3 CSI driver v2, bạn có thể hoàn thành các job mô phỏng tài chính quy mô lớn nhanh hơn đến **2x** bằng cách loại bỏ overhead của việc mỗi pod tự cache cùng một dữ liệu.

## **Đơn giản hóa quản lý credential với hỗ trợ EKS Pod Identity**

Trong bản nâng cấp lớn này, chúng tôi bổ sung hỗ trợ [EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html), một lựa chọn **đơn giản và dễ mở rộng** so với [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) để quản lý access policies trên các EKS cluster, bao gồm cả **cross-account access**. EKS Pod Identity có thể được cài đặt như một [Amazon EKS add-on](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html#podi-install).

Khi dùng EKS Pod Identity, bạn **không cần** tạo [OpenID Connector (OIDC)](https://openid.net/connect/) riêng cho mỗi cluster; cũng **không cần** namespace và service account tồn tại sẵn, từ đó đơn giản hóa thiết lập (ví dụ: tenant onboarding). Ngoài ra, EKS Pod Identity hỗ trợ đính [session tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_session-tags.html) vào temporary credentials gắn với service account, qua đó cho phép [Attribute-based access control (ABAC)](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html). Pod Identity cung cấp các tag như namespace session, kubernetes-service-account hoặc eks-cluster-name để bạn kiểm soát truy cập ở mức độ chi tiết mong muốn.

Driver có thể cấu hình để dùng hai kiểu credential:

* **Driver-level credentials**: một bộ permission gắn với service account của driver được dùng chung cho các pod trong cluster.

* **Pod-level credentials**: credential gắn với service account của từng pod được dùng để cấu hình quyền truy cập cho pod đó. Tùy chọn thứ hai, kết hợp với EKS Pod Identity và tính năng session tags, cho phép bạn xây dựng các giải pháp **multi-tenant** có khả năng mở rộng.

Ví dụ, với EKS Pod Identity, bạn có thể định nghĩa các điều khoản trong policy [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) gắn với IAM role của một service account, sử dụng các session tag như sau:

{  
  "Version": "2012-10-17",  
  "Statement": \[  
    {  
      "Effect": "Allow",  
      "Action": \["s3:GetObject"\],  
      "Resource": \[  
        "arn:aws:s3:::some-bucket/s3mp-apps-data/shared/\*",  
        "arn:aws:s3:::some-bucket/s3mp-apps-data/${aws:PrincipalTag/kubernetes-namespace}/\*"  
      \]  
    }  
  \]  
}

## **Các mẫu sử dụng Mountpoint for Amazon S3 CSI driver v2**

Trong phần này, chúng tôi trình bày một ví dụ minh họa driver hoạt động. Ví dụ giả định bạn đã cài đặt driver và cấu hình quyền truy cập tới S3 bucket và các prefix tương ứng thông qua [EKS Pod Identity](https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html) hoặc [IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). 

### **Tạo PersistentVolume**

Để ứng dụng có thể xử lý các file trong S3 bucket như thể chúng nằm trong thư mục cục bộ, chúng ta cần tạo một PersistentVolume (vì tại thời điểm viết bài, driver chỉ hỗ trợ Kubernetes [static provisioning](https://github.com/awslabs/aws-mountpoint-csi-driver)). 

apiVersion: v1  
kind: PersistentVolume  
metadata:  
  name: s3mp-data-pv  
spec:  
  storageClassName: '' \# Required for static provisioning  
  capacity:  
    storage: 1Gi \# Ignored, required  
  accessModes:  
    \- ReadWriteMany  
  claimRef: \# To ensure no other PVCs can claim this PV  
    namespace: apps  
    name: s3mp-data-pvc  
  mountOptions:  
    \- region: eu-central-1  
    \- prefix data/  
  csi:  
    driver: s3.csi.aws.com \# Required  
    volumeHandle: my-s3-data-volume \# Must be unique  
    volumeAttributes:  
      bucketName: mp-s3-bucket-\<some unique suffix\>  
      cache: emptyDir  
      cacheEmptyDirSizeLimit: 1Gi  
      cacheEmptyDirMedium: Memory

Persistent Volume định nghĩa:

* Dung lượng lưu trữ của nó (yêu cầu bởi đặc tả CSI, nhưng không được sử dụng thực tế)  
* Các chế độ truy cập (access modes)  
* Tên và vùng (region) của bucket được sử dụng  
* Kích thước và vị trí của bộ nhớ đệm (cache) mà các pod của driver Mountpoint sẽ sử dụng

### **Sử dụng PersistentVolume**

Bây giờ chúng ta tạo ứng dụng dùng PersistentVolume ở trên. 

apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  name: s3mp-data-pvc  
  namespace: apps  
spec:  
  storageClassName: ''  
  accessModes:  
    \- ReadWriteMany  
  resources:  
    requests:  
      storage: 1Gi  
  volumeName: s3mp-data-pv  
\---  
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
        \- name: app  
          image: busybox  
          command: \['sh', '-c', 'trap : TERM INT; sleep infinity & wait'\]  
          resources:  
            ...  
          volumeMounts:  
            \- name: data  
              mountPath: /data  
      volumes:  
        \- name: data  
          persistentVolumeClaim:  
            claimName: s3mp-data-pvc

Đoạn mã trên định nghĩa:

* Một PersistentVolumeClaim sử dụng PersistentVolume  
* Dung lượng lưu trữ gắn kết (theo yêu cầu của đặc tả CSI, nhưng không được sử dụng thực tế)  
* Thuộc tính storageClassName phải trùng khớp với giá trị được định nghĩa trong PersistentVolume  
* Một ứng dụng gắn PersistentVolumeClaim vào đường dẫn /data

Chúng tôi ép cả hai bản sao (replica) của ứng dụng chạy trên cùng một node worker để minh họa khả năng cache.

### **Xác thực triển khai**

Các PV/PVC và ứng dụng được triển khai sẽ trông tương tự như sau (rút gọn một số dữ liệu cho dễ đọc): 

$ kubectl get pods \-n apps \-o wide

NAME                        READY   STATUS    AGE     IP            NODE  
some-app-556c9447fd-sjf4k  1/1     Running   3m14s   10.0.94.129   ip-10-0-85-106.eu-central-1.compute.internal  
some-app-556c9447fd-wkpbv  1/1     Running   3m14s   10.0.89.177   ip-10-0-85-106.eu-central-1.compute.internal

$ kubectl get pv,pvc \-n apps

NAME                            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE  
persistentvolume/s3mp-data-pv   1Gi        RWX            Retain           Bound    apps/s3mp-data-pvc                   \<unset\>                          4m  
NAME                                   STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE  
persistentvolumeclaim/s3mp-data-pvc    Bound    s3mp-data-pv    1Gi        RWX

Khi ứng dụng được triển khai, driver tạo một Mountpoint pod trên mỗi node nơi các pod đang chạy (trong ví dụ này là một node) để thiết lập chia sẻ pod cho phép các pod khác trên node truy cập cache cục bộ. 

$ kubectl get pods \-n mount-s3 \-o wide  
NAME     READY   STATUS    AGE     IP            NODE  
mp-xgscz 1/1     Running   3m48s   10.0.94.38    ip-10-0-85-106.eu-central-1.compute.internal

### **Sử dụng cache**

Truy cập một file trong bucket đã mount (prefix `data/`) để xem cache hoạt động, ví dụ tải xuống và đếm số từ trong một file 2.5MB: 

$ POD\_ID=$(kubectl get pod \-n apps \-l name=some-app \-o jsonpath='{.items\[0\].metadata.name}')  
$ echo ${POD\_ID}  
some-app-556c9447fd-sjf4k

$ kubectl exec \-it \-n apps ${POD\_ID} \-- time wc /data/some-file  
  129419    130195   2424064 /data/some-file  
 real    0m 0.22s  
 user    0m 0.01s  
 sys     0m 0.00s

$ POD\_ID=$(kubectl get pod \-n apps \-l name=some-app \-o jsonpath='{.items\[1\].metadata.name}')  
$ echo ${POD\_ID}  
some-app-556c9447fd-wkpbv

$ kubectl exec \-it \-n apps ${POD\_ID} \-- time wc /data/some-file  
  129419    130195   2424064 /data/some-file  
 real    0m 0.09s  
 user    0m 0.01s  
 sys     0m 0.00s

Vì việc đếm từ trên cùng một instance có thời gian gần như nhau, bạn có thể thấy khác biệt do cache tạo ra qua thời gian `real` trong hai lệnh trên. 

## **Nâng cấp từ Mountpoint for Amazon S3 CSI driver v1 lên v2**

Để cài đặt driver, hãy làm theo [installation guide](https://github.com/awslabs/aws-mountpoint-csi-driver). Nếu bạn nâng cấp từ v1, lưu ý bản phát hành mới có thay đổi về cách cấu hình volume. Khuyến nghị xem qua [list of configuration changes](https://github.com/awslabs/aws-mountpoint-csi-driver) trước khi nâng cấp vì bạn sẽ cần điều chỉnh tương ứng. Tại thời điểm viết bài, v2 có một số ràng buộc khi làm việc với node autoscaler như [Karpenter](https://karpenter.sh) và [Cluster Autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html). Xem chi tiết tại [GitHub issue](https://github.com/awslabs/aws-mountpoint-csi-driver). 

## **Kết luận**

Mountpoint for Amazon S3 CSI driver v2 cải thiện cách ứng dụng Kubernetes tương tác với dữ liệu S3: chia sẻ pod để tăng hiệu quả tài nguyên và tốc độ, hỗ trợ SELinux, logging qua `kubectl`, và quản lý quyền truy cập đơn giản hơn với Amazon EKS Pod Identity. Driver sẵn sàng trên [GitHub](https://github.com/awslabs/aws-mountpoint-csi-driver); chúng tôi hoan nghênh phản hồi/đóng góp của bạn. 

Tags: [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/), [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/), [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/), [Open Source](https://aws.amazon.com/opensource/)

**![Ran Pergamin](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2023/06/15/ran-pergamin.jpg)**

**Ran Pergamin**  
Ran Pergamin là Software Engineer tại AWS, hiện đang làm việc trong nhóm Amazon S3, tập trung phát triển các giải pháp lưu trữ và công cụ truy cập dữ liệu quy mô lớn cho khách hàng doanh nghiệp.

**![Ankit Kalyani](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/Ankit-Kalyani.png)**

**Ankit Kalyani**  
Ankit Kalyani là Senior Software Engineer tại AWS, chuyên về hệ thống phân tán và nền tảng lưu trữ, với trọng tâm là hiệu năng và khả năng mở rộng của Amazon S3.

**![Burak Varli](https://d2908q01vomqb2.cloudfront.net/e1822db470e60d090affd0956d743cb0e7cdf113/2025/08/06/Burak-Varli.png)**

**Burak Varli**  
Burak Varli là Senior Solutions Architect tại AWS, hỗ trợ khách hàng trong việc xây dựng và tối ưu hóa khối lượng công việc lưu trữ, đặc biệt trong các môi trường Kubernetes và containerized workloads.

**![Dmitry Nutels](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2022/11/18/passport.png)**

**Dmitry Nutels**  
Dmitry Nutels là Principal Software Engineer tại AWS, dẫn dắt phát triển các công cụ lưu trữ và truy cập dữ liệu trên quy mô lớn, bao gồm Mountpoint for Amazon S3 và các công nghệ CSI driver liên quan.