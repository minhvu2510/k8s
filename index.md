## Nội dung
1. Cài đặt k8s
2. K9s
3. Pod, Node, Kubectl
4. ReplicaSet, HPA
5. Deployment
6. Service, Secret
7. Service headlees
8. DaemonSet Job và CronJob trong Kubernetes
9. Pv, pvc
10. PersistentVolume NFS trên Kubernetes
11. Sử dụng Ingress trong Kubernetes

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

Tạo Pod từ file cấu hình .yaml

##### 3.Node trong Kubernetes

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/minhvu2510/k8s/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
