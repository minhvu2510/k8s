## Nội dung
1. Cài đặt k8s
2. K9s
3. Pod, Node, Kubectl
4. ReplicaSet, HPA
5. Deployment
6. Service, Secret
7. DaemonSet Job và CronJob trong Kubernetes
8. Pv, pvc
9. PersistentVolume NFS trên Kubernetes
10. Sử dụng Ingress trong Kubernetes

Tham khảo các file config tại source [link](https://github.com/minhvu2510/k8s)


#### 1. Cài đặt k8s
Kubernetes (còn gọi là k8s) là một hệ thống để chạy, quản lý, điều phối các ứng dụng được container hóa trên một cụm máy (1 hay nhiều) gọi là cluster.
Bạn có thể điều chỉnh tăng giảm tài nguyên, bản chạy phục vụ cho dịch vụ (scale), bạn có thể cập nhật (update), thu hồi update khi có vấn đề ... Kubernetes là một công cụ mạnh mẽ, mềm dẻo, dễ mở rộng khi so sánh nó với công cụ tương tự là Docker Swarm!

##### Kiến trúc
![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115216153-6df0d780-a12e-11eb-8931-0a23fc305b2b.png)
##### Tạo Cluster Kubernetes
Cụm k8s gồm 1 master(10.5.22.109) và 2 node

1.Tạo master

> kubeadm init --apiserver-advertise-address=10.5.22.109 --pod-network-cidr=192.168.0.0/16

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115218013-4e5aae80-a130-11eb-9cdc-9fc29fd59cc3.png)

Sau khi lệnh chạy xong, chạy tiếp cụm lệnh nó yêu cầu chạy sau khi khởi tạo- để chép file cấu hình đảm bảo trình kubectl trên máy này kết nối Cluster

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115219396-b231a700-a131-11eb-9c8a-6c9abd55a7cf.png)

Tiếp đó, nó yêu cầu cài đặt một Plugin mạng trong các Plugin tại addon, ở đây đã chọn calico, nên chạy lệnh sau để cài nó
> kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml

2. Tạo cụm node

ssh vào máy master chạy lệnh
> kubeadm token create --print-join-command

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115220384-abeffa80-a132-11eb-8b47-5011ddc95411.png)

Chạy lệnh được sinh ra trên các cụm worker.

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115220638-fc675800-a132-11eb-950a-19ce87edd8e3.png)

Kết quả thu được

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115220808-32a4d780-a133-11eb-8652-0d06b6148742.png)

Hoàn thành cài đặt cụm k8s với 1 master, 2 node

#### 2. Cài đặt k9s

K9S là công cụ CLI để quản lý, tương tác với K8S (Kubernetes) với giao diện dòng lệnh trên Terminal

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115221791-3ab14700-a134-11eb-83b2-58fc9ee363c1.png)

Coppy nội dung file admin.conf trong thư  mục /etc/kubernetes/ trên master về máy local (vd conf)
Tải source k9s tại trang chủ chạy lênh sau để khởi chạy

>./k9s --kubeconfig conf

Gõ help để xem các phím tắt thao tác với k9s

#### 3. Pod, Node, Kubectl

##### 1.Node trong Kubernetes
Trong Kubernetes Node là đơn vị nhỏ nhất xét về phần cứng. Nó là một máy vật lý hay máy ảo (VPS) trong cụm máy (cluster). Xem các nút (node) trong cụm (cluster) chạy lệnh:
Mở k9s gõ:
> Shift :

Sau đó gõ tài nguyên node
> node

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115326455-7fca8d00-a1b7-11eb-9946-e7f95f40d45f.png)
##### 2.Pods trong Kubernetes

Kubernetes không chạy các container một cách trực tiếp, thay vào đó nó bọc một hoặc vài container vào với nhau trong một cấu trúc gọi là POD. Các container cùng một pod thì chia sẻ với nhau tài nguyên và mạng cục bộ của pod.

Xem các pod bằng lệnh pod trong k9s
![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115327071-8574a280-a1b8-11eb-834c-013b8d5f6f73.png)

Tạo Pod từ file cấu hình .yaml. vd

1-swarmtest-node.yaml
```markdown
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: app1
    ungdung: ungdung1
  name: ungdungnode
spec:
  containers:
  - name: c1
    image: ichte/swarmtest:node
    resources:
      limits:
        memory: "150M"
        cpu: "100m"
    ports:
      - containerPort: 8085
      # - containerPort: 8086
```

File trên khai báo một Pod, đặt tên là ungdungnode, gán nhãn app: app1, ungdung: ungdung1 trong Pod chạy một Container từ image ichte/swarmtest:php, cổng của Container 8085

Triển khai tạo Pod từ file này, thực hiện lệnh sau
> kubectl apply -f 1-swarmtest-node.yaml

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115334500-b3141880-a1c5-11eb-8919-3847fb30a782.png)

###### Xem thông tin chi tiết của Pod
1.xem thông tin chi tiết của Pod
>kubectl describe pod/namepod

2 .Tra cứu log của Pod
>kubectl logs pod/podname

3 . Chạy tiến trình trong Pod và gắn vào terminal
>kubectl exec -it mypod bash

Chú ý, nếu pod có nhiều container bên trong, thì cần chỉ rõ thi hành container nào bên trong nó bằng tham số -c containername

4 .xóa Pod 
>kubectl delete -f firstpod.yaml

#### 4. ReplicaSet, HPA
ReplicaSet là một điều khiển Controller - nó đảm bảo ổn định các nhân bản (số lượng và tình trạng của POD, replica) khi đang chạy.

Ví dụ: Cấu hình sau định nghĩa một ReplicaSet đặt tên là rsapp, nó quản lý nhân bản 3 POD có nhãn app=rsapp, POD có một container từ image ichte/swarmtest:node

2.rs.yaml
```markdown
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rsapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rsapp
  template:
    metadata:
      name: rsapp
      labels:
        app: rsapp
    spec:
      containers:
      - name: app
        image: ichte/swarmtest:node
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        ports:
          - containerPort: 8085
```

Thực hiện lệnh để triển khai/cập nhật

>kubectl apply -f 2.rs.yaml

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115337150-ab0aa780-a1ca-11eb-8300-ecec8c0d4906.png)

Để lấy các ReplicaSet thực hiện lệnh
>kubectl get rs

Thông tin về ReplicaSet có tên rsapp
>kubectl describe rs/rsapp

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115337341-ffae2280-a1ca-11eb-9a76-1b62b091c9a4.png)

Liệt kê các POD có nhãn "app=rsapp"
>kubectl get po -l "app=rsapp"

###### Horizontal Pod Autoscaler với ReplicaSet
Horizontal Pod Autoscaler là chế độ tự động scale (nhân bản POD) dựa vào mức độ hoạt động của CPU đối với POD, nếu một POD quá tải - nó có thể nhân bản thêm POD khác và ngược lại - số nhân bản dao động trong khoảng min, max cấu hình

Ví dụ, với ReplicaSet rsapp trên đang thực hiện nhân bản có định 3 POD (replicas), nếu muốn có thể tạo ra một HPA để tự động scale (tăng giảm POD) theo mức độ đang làm việc CPU, có thể dùng lệnh sau:
> kubectl autoscale rs rsapp --max=2 --min=1

Để linh loạt và quy chuẩn, nên tạo ra HPA (HorizontalPodAutoscaler) từ cấu hình file yaml (Tham khảo HPA API ) , ví dụ:
```markdown
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: rsapp-scaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: ReplicaSet
    name: rsapp
  minReplicas: 5
  maxReplicas: 10
  # Thực hiện scale CPU hoạt động ở 50% so với CPU mà POD yêu cầu
  targetCPUUtilizationPercentage: 50
```

#### 5. Deployment
Deployment quản lý một nhóm các Pod - các Pod được nhân bản, nó tự động thay thế các Pod bị lỗi, không phản hồi bằng pod mới nó tạo ra. Như vậy, deployment đảm bảo ứng dụng của bạn có một (hay nhiều) Pod để phục vụ các yêu cầu.

Deployment sử dụng mẫu Pod (Pod template - chứa định nghĩa / thiết lập về Pod) để tạo các Pod (các nhân bản replica), khi template này thay đổi, các Pod mới sẽ được tạo để thay thế Pod cũ ngay lập tức.

Tạo file cấu hình Deployment (yaml) tham khảo API - Deployment API

Ví dụ khai báo file Deployment sau

1.myapp-deploy.yaml
```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  # tên của deployment
  name: deployapp
spec:
  # số POD tạo ra
  replicas: 3

  # thiết lập các POD do deploy quản lý, là POD có nhãn  "app=deployapp"
  selector:
    matchLabels:
      app: deployapp

  # Định nghĩa mẫu POD, khi cần Deploy sử dụng mẫu này để tạo Pod
  template:
    metadata:
      name: podapp
      labels:
        app: deployapp
    spec:
      containers:
      - name: node
        image: ichte/swarmtest:node
        resources:
          limits:
            memory: "128Mi"
            cpu: "100m"
        ports:
          - containerPort: 8085
```

Thực hiện lệnh sau để triển khai
> kubectl apply -f 1.myapp-deploy.yaml

Khi Deployment tạo ra, tên của nó là deployapp, có thể kiểm tra với lệnh:
>kubectl get deploy -o wide

Deploy này quản sinh ra một ReplicasSet và quản lý nó, gõ lệnh sau để hiện thị các ReplicaSet
>kubectl get rs -o wide

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115338387-ed34e880-a1cc-11eb-939c-605efffd20ce.png)

##### Scale Deployment
Scale thay đổi chỉ số replica (số lượng POD) của Deployment, ý nghĩa tương tự như scale đối với ReplicaSet trong phần trước. Ví dụ để scale với 10 POD thực hiện lệnh:
>kubectl scale deploy/deployapp --replicas=10

Muốn thiết lập scale tự động với số lượng POD trong khoảng min, max và thực hiện scale khi CPU của POD hoạt động ở mức 50% thì thực hiện
>kubectl autoscale deploy/deployapp --min=2 --max=5 --cpu-percent=50

#### 6. Service, Secret

Mặc dù mỗi POD khi tạo ra nó có một IP để liên lạc, tuy nhiên vấn đề là mỗi khi POD thay thế thì là một IP khác, nên các dịch vụ truy cập không biết IP mới nếu ta cấu hình nó truy cập đến POD nào đó cố định. Để giải quết vấn đề này sẽ cần đến Service.

Service (micro-service) là một đối tượng trừu tượng nó xác định ra một nhóm các POD và chính sách để truy cập đến POD đó. Nhóm cá POD mà Service xác định thường dùng kỹ thuật Selector (chọn các POD thuộc về Service theo label của POD).

Cũng có thể hiểu Service là một dịch vụ mạng, tạo cơ chế cân bằng tải (load balancing) truy cập đến các điểm cuối (thường là các Pod) mà Service đó phục vụ.

###### Tạo Service có Selector, chọn các Pod là Endpoint của Service
pods.yaml
```markdown
apiVersion: v1
kind: Pod
metadata:
  name: myapp1
  labels:
    app: app1
spec:
  containers:
  - name: n1
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "100m"
    ports:
      - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: myapp2
  labels:
    app: app1
spec:
  containers:
  - name: n1
    image: httpd
    resources:
      limits:
        memory: "128Mi"
        cpu: "100m"
    ports:
      - containerPort: 80
```

Triển khai file trên
>kubectl apply -f 3.pods.yaml

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115355999-6a6c5780-a1e5-11eb-8dff-76fd3928e964.png)

Tiếp tục tạo ra service có tên svc2 có thêm thiết lập selector chọn nhãn app=app1

svc2.yaml
```markdown
apiVersion: v1
kind: Service
metadata:
  name: svc2
spec:
  selector:
     app: app1
  type: ClusterIP
  ports:
    - name: port1
      port: 80
      targetPort: 80
```

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115359594-06e42900-a1e9-11eb-9d34-6436c831b908.png)
Thông tin trên ta có, endpoint của svc2 là 192.168.201.4:80,192.168.74.197:80, hai IP này tương ứng là của 2 POD trên. Khi truy cập địa chỉ svc2:80 hoặc 10.101.4.19:80 thì căn bằng tải hoạt động sẽ là truy cập đến 192.168.201.4:80 hoặc 192.168.74.197:80

###### Tạo Service kiểu NodePort

Kiểu NodePort này tạo ra có thể truy cập từ ngoài internet bằng IP của các Node, ví dụ sửa dịch vụ svc2 trên thành dịch vụ svc3 kiểu NodePort
svc3.yaml
```markdown
apiVersion: v1
kind: Service
metadata:
  name: svc3
spec:
  selector:
     app: app1
  type: NodePort
  ports:
    - name: port1
      port: 80
      targetPort: 80
      nodePort: 31080
```
Triển khai file trên
>kubectl appy -f 5.svc3.yaml

Sau khi triển khai có thể truy cập với IP là địa chỉ IP của các Node và cổng là 31080
![image](https://user-images.githubusercontent.com/36092539/115361055-6858c780-a1ea-11eb-9307-54babb33ccea.png)

![Tux, the Linux mascot](https://user-images.githubusercontent.com/36092539/115360603-f97b6e80-a1e9-11eb-8941-6087655a1293.png)

#### 7. PersistentVolume NFS trên Kubernetes
DaemonSet (ds) đảm bảo chạy trên mỗi NODE một bản copy của POD. Triển khai DaemonSet khi cần ở mỗi máy (Node) một POD, thường dùng cho các ứng dụng như thu thập log, tạo ổ đĩa trên mỗi Node

Job (jobs) có chức năng tạo các POD đảm bảo nó chạy và kết thúc thành công. Khi các POD do Job tạo ra chạy và kết thúc thành công thì Job đó hoàn thành. Khi bạn xóa Job thì các Pod nó tạo cũng xóa theo. Một Job có thể tạo các Pod chạy tuần tự hoặc song song. Sử dụng Job khi muốn thi hành một vài chức năng hoàn thành xong thì dừng lại (ví dụ backup, kiểm tra ...)

CronJob (cj) - chạy các Job theo một lịch định sẵn. Việc lên lịch cho CronJob khai báo giống Cron của Linux.

#### 8. Pv, pvc
Tạo ổ đĩa lưu dữ liệu lâu dài PV và yêu cầu truy cập đến PV bằng PVC, cách mount PVC vào POD

PersistentVolume (pv) là một phần không gian lưu trữ dữ liệu tronnng cluster, các PersistentVolume giống với Volume bình thường tuy nhiên nó tồn tại độc lập với POD (pod bị xóa PV vẫn tồn tại), có nhiều loại PersistentVolume có thể triển khai như NFS, Clusterfs ...

PersistentVolumeClaim (pvc) là yêu cầu sử dụng không gian lưu trữ (sử dụng PV). Hình dung PV giống như Node, PVC giống như POD. POD chạy nó sử dụng các tài nguyên của NODE, PVC hoạt động nó sử dụng tài nguyên của PV

Tạo 1 pv
```markdown
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
  labels:
    name: pv1
spec:
  storageClassName: mystorageclass
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/v1"
```

Tạo 1 pvc
```markdown
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  labels:
    name: pvc1
spec:
  storageClassName: mystorageclass
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 150Mi
```

Sử dụng PVC với Pod

```markdown
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      name: myapp
  template:
    metadata:
      name: myapp
      labels:
        name: myapp
    spec:
      volumes:
      # Khai báo VL sử dụng PVC
      - name: myvolume
        persistentVolumeClaim:
          claimName: pvc1
      containers:
      - name: myapp
        image: busybox
        resources:
          limits:
            memory: "50Mi"
            cpu: "500m"
        command:
          - sleep
          - "600"
        volumeMounts:
        - mountPath: "/data"
          name: myvolume
```

#### 9. PersistentVolume NFS trên Kubernetes
Cài đặt NFS làm Server chia sẻ file (Kubernetes)

Tạo PersistentVolume NFS
pv-nfs.yaml
```markdown
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  storageClassName: mystorageclass
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: "/data/mydata/"
    server: "10.5.22.109"
```

Triển khai và kiểm tra
>kubectl apply -f pv-nfs.yaml

>kubectl get pv -o wide

>kubectl describe pv/pv1

![image](https://user-images.githubusercontent.com/36092539/115368907-c0df9300-a1f1-11eb-96cc-d37c25bfdbe6.png)

Tạo PersistentVolumeClaim NFS
pvc-nfs.yaml
```markdown
apiVersion: v1
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: mystorageclass
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```
Triển khai và kiểm tra
> kubectl apply -f pvc-nfs.yaml

> kubectl get pvc,pv -o wide

![image](https://user-images.githubusercontent.com/36092539/115369222-187dfe80-a1f2-11eb-8462-b90b488d47d9.png)

SSH vào máy master, vào thư mục chia sẻ /data/mydata tạo một file index.html với nội dung đơn giản, ví dụ:
> <p>Mount PersistentVolumeClaim NFS vào Container ...</p>
Tạo file triển khai, gồm có POD chạy http và dịch vụ kiểu NodePort, ánh xạ cổng host 31080 vào cổng 80 của POD

httpd.yaml
```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      volumes:
        - name: htdocs
          persistentVolumeClaim:
            claimName: pvc1
      containers:
      - name: app
        image: httpd
        resources:
          limits:
            memory: "100M"
            cpu: "100m"
        ports:
          - containerPort: 80
        volumeMounts:
          - mountPath: /usr/local/apache2/htdocs/
            name: htdocs
---
apiVersion: v1
kind: Service
metadata:
  name: httpd
  labels:
    run: httpd
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
    nodePort: 31080
  selector:
    app: httpd
```

Sau khi triển khai, truy cập từ một IP của các node và cổng 31080 thu được nội dung file index.html vừa tạo
#### Sử dụng Ingress trong Kubernetes
Triển khai và sử dụng NGINX Ingress Controller trong Kubernetes, ví dụ tạo Ingress chuyển hướng traffic http, https vào một dịch vụ trong Kubernetes

###### Cài đặt NGINX Ingress Controller
Các menifest (yaml) cần triển khai ở trong thư mục k8s/exams/ingress_deployments, hãy vào thư mục này.

```markdown
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f daemon-set/nginx-ingress.yaml
```

Kiểm tra daemonset và các pod của Nginx Ingress Controller
```markdown
kubectl get ds -n nginx-ingress
kubectl get po -n nginx-ingress
```

tạo Ingress
app-test.yaml
```markdown
apiVersion: v1
kind: Service
metadata:
  name: http-test-svc
  namespace: nginx-ingress
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: http-test-app
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: http-test-svc
  name: http-test-svc
  namespace: nginx-ingress
spec:
  replicas: 2
  selector:
    matchLabels:
      run: http-test-app
  template:
    metadata:
      labels:
        run: http-test-app
    spec:
      containers:
      - image: httpd
        imagePullPolicy: IfNotPresent
        name: http
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
```

app-test-ingress.yaml
```markdown
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: app
  namespace: nginx-ingress
spec:
  rules:
    # Tên miền truy cập
  - host: testk8s.test
    http:
      paths:
      - path: /
        backend:
          # dịch vụ phục vụ tương ứng với tên miền và path
          serviceName: http-test-svc
          servicePort: 80
```


