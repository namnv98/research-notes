Dưới đây là phiên bản tài liệu **liền mạch và liên tục** về các thành phần quan trọng trong Kubernetes, bao gồm Node,
Pod, Volume, ReplicaSet, Deployment, ConfigMap cũng như các đối tượng nâng cao như StatefulSet và DaemonSet. Bạn có thể
sử dụng tài liệu này để nắm bắt toàn bộ khái niệm và cách quản lý các tài nguyên trong Kubernetes.

---

# Tổng Quan về Kubernetes

**Kubernetes** (viết tắt là **k8s**) là một hệ thống mã nguồn mở dùng để **triển khai, quản lý và điều phối các ứng dụng
container hóa** trên một cụm máy chủ (cluster). Với Kubernetes, bạn có thể tự động điều chỉnh tài nguyên, scale số lượng
container phục vụ cho một service, cập nhật ứng dụng một cách mềm mại và thu hồi phiên bản cập nhật khi cần thiết.
Kubernetes giúp tối ưu hóa quản lý ứng dụng trên quy mô lớn với tính linh hoạt, tự động hóa và khả năng phục hồi cao.

---

# 1. Node trong Kubernetes

**Node** là máy vật lý hoặc máy ảo chạy các thành phần của Kubernetes và cung cấp tài nguyên tính toán cho các Pod. Mỗi
cluster Kubernetes có thể có một hoặc nhiều Node, và chúng được chia thành:

- **Master Node:**  
  Chịu trách nhiệm quản lý, điều phối tài nguyên, giám sát và điều khiển toàn bộ cluster. Các thành phần chủ chốt trên
  Master Node bao gồm:
  - **kube-apiserver:** Cung cấp API RESTful cho các công cụ như `kubectl`.
  - **etcd:** Cơ sở dữ liệu phân tán lưu trữ trạng thái và cấu hình toàn bộ cluster.
  - **kube-scheduler:** Phân phối các Pod mới đến các Node dựa trên tài nguyên và hạn chế.
  - **kube-controller-manager:** Chạy các controller để đảm bảo trạng thái mong muốn của cluster.
  - *(Tùy chọn)* **cloud-controller-manager:** Hỗ trợ tích hợp các dịch vụ từ nhà cung cấp cloud.

- **Worker Node:**  
  Nơi chạy các Pod thực tế. Mỗi Worker Node chủ yếu bao gồm:
  - **kubelet:** Agent đảm bảo rằng container trong Pod hoạt động đúng theo cấu hình.
  - **kube-proxy:** Quản lý các quy tắc mạng, định tuyến lưu lượng đến Pod.
  - **Container Runtime:** Môi trường chạy container như Docker, containerd hay CRI-O.

📝 **Một số lệnh quan trọng:**

```bash
kubectl get nodes                 # Liệt kê các Node trong cluster
kubectl describe node <node-name>  # Xem chi tiết một Node cụ thể
```

---

# 2. Pod – Đơn Vị Triển Khai Cơ Bản

**Pod** là đơn vị triển khai nhỏ nhất trong Kubernetes và có thể chứa một hoặc nhiều container. Các container trong cùng
một Pod chia sẻ tài nguyên như IP, tên máy chủ, và các volume, giúp chúng giao tiếp với nhau một cách hiệu quả.

Ví dụ file YAML định nghĩa Pod:

```yaml
apiVersion: v1         # Phiên bản API được sử dụng
kind: Pod              # Loại tài nguyên: Pod
metadata:
  name: example-pod    # Tên của Pod
spec:
  containers:
    - name: nginx-container    # Tên của container
      image: nginx:1.27.2      # Image và tag của container
      ports:
        - containerPort: 80    # Cổng mà container expose ra
      resources:
        limits: # Giới hạn tài nguyên sử dụng
          memory: "128Mi"
          cpu: "500m"
```

*Lưu ý:* Pod không tự động tái khởi động khi gặp lỗi, do đó chúng thường được quản lý bởi các đối tượng cấp cao hơn (như
ReplicaSet, Deployment) để đảm bảo tính khả dụng của ứng dụng.

---

# 3. Volume – Lưu Trữ Dữ Liệu Bền Vững

**Volume** trong Kubernetes giúp lưu trữ dữ liệu cho các container một cách bền vững. Khi container xoá hoặc khởi động
lại, dữ liệu lưu trữ tạm thời bên trong container sẽ mất đi, nhưng dữ liệu trên Volume có thể được duy trì tùy theo loại
Volume sử dụng.

Ví dụ file YAML cho Pod có Volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-volume
spec:
  containers:
    - name: busybox
      image: busybox:latest
      command: [ "sleep", "3600" ]
      volumeMounts:
        - name: my-volume
          mountPath: /data    # Đường dẫn mount trong container
  volumes:
    - name: my-volume
      emptyDir: { }           # Volume kiểu tạm thời, dữ liệu mất khi Pod dừng
```

Các loại Volume khác như **PersistentVolume (PV)** và **PersistentVolumeClaim (PVC)** cung cấp lưu trữ lâu dài cho các
ứng dụng cần dữ liệu bền vững.

---

# 4. ReplicaSet – Duy Trì Số Lượng Pod

**ReplicaSet** đảm bảo rằng số lượng Pod chạy theo mẫu định sẵn luôn đúng theo yêu cầu của người dùng. Nếu một Pod gặp
lỗi hoặc bị xoá, ReplicaSet sẽ tự động tạo Pod mới để duy trì số lượng replica.

Ví dụ file YAML định nghĩa ReplicaSet:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: example-replicaset
spec:
  replicas: 3                 # Số lượng Pod mong muốn
  selector:
    matchLabels:
      app: nginx            # Chọn các Pod có label app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.2
          ports:
            - containerPort: 80
```

📝 **Các lệnh để quản lý ReplicaSet:**

```bash
kubectl apply -f replicaset.yaml               # Tạo hoặc cập nhật ReplicaSet
kubectl get replicasets                        # Liệt kê các ReplicaSet hiện có
kubectl describe replicaset <replicaset-name>  # Xem chi tiết ReplicaSet
kubectl delete replicaset <replicaset-name>    # Xóa ReplicaSet
```

---

# 5. Deployment – Quản Lý Triển Khai & Cập Nhật Ứng Dụng

**Deployment** là đối tượng cấp cao dùng để quản lý việc triển khai, cập nhật (rolling update), rollback và scale các
ứng dụng. Nó quản lý các ReplicaSet bên dưới để đảm bảo rằng ứng dụng luôn chạy theo trạng thái mong muốn.

Ví dụ file YAML định nghĩa Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3                     # Số lượng Pod mong muốn
  selector:
    matchLabels:
      app: nginx                # Chỉ định các Pod thuộc Deployment
  template:
    metadata:
      labels:
        app: nginx              # Gán nhãn cho Pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.2
          ports:
            - containerPort: 80
```

📝 **Các lệnh quản lý Deployment:**

```bash
kubectl apply -f deployment.yaml                    # Tạo hoặc cập nhật Deployment
kubectl get deployments                             # Liệt kê các Deployment
kubectl describe deployment <deployment-name>       # Xem chi tiết Deployment
kubectl delete deployment <deployment-name>         # Xóa Deployment

kubectl scale deployment <deployment-name> --replicas=<number>   # Thay đổi số lượng replica
kubectl rollout history deployment <deployment-name>             # Xem lịch sử cập nhật
kubectl rollout undo deployment <deployment-name> --to-revision=<revision-number>  # Quay lại phiên bản trước
kubectl rollout restart deployment <deployment-name>             # Restart Deployment
```

---

# 6. ConfigMap – Quản Lý Cấu Hình Dễ Dàng

**ConfigMap** là đối tượng dùng để lưu trữ cấu hình dưới dạng cặp key-value, giúp tách riêng cấu hình khỏi container
image. Điều này làm cho việc thay đổi cấu hình trở nên đơn giản hơn mà không cần rebuild image.

Ví dụ file YAML tạo ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  key1: value1
  key2: value2
  key3: value3
```

Ví dụ file YAML mount ConfigMap vào Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: app-container
      image: myapp:latest
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config   # Mount vào thư mục /etc/config trong container
  volumes:
    - name: config-volume
      configMap:
        name: example-config     # Liên kết với ConfigMap đã tạo
```

📝 **Các lệnh quản lý ConfigMap:**

```bash
kubectl apply -f configmap.yaml               # Tạo hoặc cập nhật ConfigMap
kubectl get configmaps                        # Liệt kê các ConfigMap
kubectl describe configmap <configmap-name>   # Xem chi tiết một ConfigMap
kubectl delete configmap <configmap-name>     # Xóa ConfigMap
```

---

# 7. StatefulSet – Quản Lý Ứng Dụng Stateful

Khi bạn cần triển khai các ứng dụng yêu cầu **trạng thái bền vững**, như cơ sở dữ liệu hoặc hệ thống cache, *
*StatefulSet** là đối tượng phù hợp. Khác với Deployment, StatefulSet đảm bảo rằng mỗi Pod được tạo ra có:

- **Hostname và DNS ổn định:** Mỗi Pod có tên duy nhất như `nginx-0`, `nginx-1`…
- **Storage riêng biệt:** Sử dụng volumeClaimTemplates để mỗi Pod nhận một PersistentVolumeClaim riêng, giúp dữ liệu
  được lưu trữ độc lập và không bị mất đi khi Pod tái khởi động.
- **Quá trình tạo và xoá theo thứ tự:** Đảm bảo quá trình khởi tạo hoặc loại bỏ Pod diễn ra có trật tự, rất quan trọng
  với các ứng dụng phân tán đòi hỏi sự nhất quán.

### Ví dụ file YAML cho StatefulSet:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  serviceName: "nginx"              # Yêu cầu có một Headless Service đi kèm
  replicas: 3                       # Số lượng Pod mong muốn
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # Tạo PVC riêng cho từng Pod
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```

*Giải thích:*

- **serviceName:** Cần định nghĩa một Headless Service (không cấp IP ClusterIP) để cung cấp DNS cho các Pod.
- **volumeClaimTemplates:** Mỗi Pod sẽ có PVC riêng, giúp lưu trữ dữ liệu bền vững qua các lần tái khởi động.
- **Quá trình triển khai theo thứ tự:** Pod được tạo theo thứ tự (nginx-0, nginx-1, …) và cũng xoá theo thứ tự nếu cần.

---

# 8. DaemonSet – Triển Khai Pod Trên Mọi Node

**DaemonSet** đảm bảo rằng một bản sao của Pod luôn chạy trên mỗi Node hoặc trên một tập hợp Node chỉ định. Đây là giải
pháp lý tưởng cho các tác vụ cần chạy trên tất cả các Node như:

- **Giám sát và thu thập log:** Các ứng dụng như Fluentd, Prometheus Node Exporter có thể được triển khai dưới dạng
  DaemonSet để thu thập log và số liệu từ mỗi Node.
- **Network plugins hoặc các agent hệ thống:** Cài đặt plugin mạng, bảo mật và các nhiệm vụ cấu hình hệ thống.

### Ví dụ file YAML cho DaemonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluent/fluentd:v1.12-debian-1.0
          ports:
            - containerPort: 24224
          volumeMounts:
            - name: config-volume
              mountPath: /fluentd/etc
      volumes:
        - name: config-volume
          configMap:
            name: fluentd-config   # ConfigMap chứa cấu hình của Fluentd
```

*Giải thích:*

- **Selector và Labels:** Giúp Kubernetes xác định các Pod thuộc DaemonSet.
- **DaemonSet đảm bảo:** Mỗi Node (hoặc tập các Node được chọn) đều chạy một bản sao của Pod triển khai.
- **Ứng dụng thông dụng:** Thu thập log, giám sát hệ thống, hoặc triển khai các agent cấu hình.

---

# Tổng Kết

- **Node:** Máy vật lý/ảo cung cấp tài nguyên cho Kubernetes Cluster.
- **Pod:** Đơn vị triển khai cơ bản chứa các container chia sẻ tài nguyên.
- **Volume:** Cung cấp lưu trữ bền vững cho dữ liệu ứng dụng.
- **ReplicaSet:** Đảm bảo số lượng Pod chạy theo mẫu định sẵn.
- **Deployment:** Quản lý triển khai, cập nhật và scaling ứng dụng một cách linh hoạt.
- **ConfigMap:** Tách riêng cấu hình ứng dụng khỏi container image, dễ thay đổi và quản lý.
- **StatefulSet:** Quản lý ứng dụng stateful với hostname, lưu trữ và quá trình triển khai theo thứ tự.
- **DaemonSet:** Đảm bảo một Pod chạy trên mỗi Node (hoặc tập Node chọn lọc) cho các nhiệm vụ giám sát, logging và quản
  lý hệ thống.

Tài liệu trên cung cấp cái nhìn tổng hợp về các thành phần thiết yếu trong Kubernetes. Nếu cần tìm hiểu thêm về các đối
tượng khác như **Job, CronJob, hay các chiến lược triển khai CI/CD** trên Kubernetes, bạn có thể mở rộng thêm theo nhu
cầu cụ thể của dự án.

Hy vọng tài liệu này giúp ích cho bạn trong việc nắm bắt kiến thức và triển khai các ứng dụng trên Kubernetes một cách
hiệu quả!