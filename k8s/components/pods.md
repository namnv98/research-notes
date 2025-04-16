# Pods

Pod là đối tượng đơn vị công việc (workload object) nhỏ nhất của Kubernetes.
Nó là đơn vị triển khai (deployment) trong Kubernetes, đại diện cho một phiên bản duy nhất của ứng dụng.
Pod là một tập hợp logic của một hoặc nhiều container, bao bọc và cô lập chúng để đảm bảo rằng chúng:

- Được lên lịch (schedule) cùng nhau trên cùng một máy chủ (host) với Pod..
- Chia sẻ chung network namespace (không gian tên mạng), nghĩa là chúng dùng chung một địa chỉ IP duy nhất được gán cho
  Pod.
- Có quyền truy cập để gắn kết (mount) cùng một bộ nhớ ngoài (volume) và các phụ thuộc chung khác.

![pods.png](../images/pods.png)

Pods bản chất là tạm thời (ephemeral) và không có khả năng tự sửa chữa (self-heal).  
Đó là lý do tại sao chúng được sử dụng với các bộ điều khiển (controller), hoặc các operator (hai khái niệm này có thể
dùng tương đương nhau), có nhiệm vụ xử lý việc sao chép (replication), chịu lỗi (fault tolerance), tự sửa chữa cho
Pods.  
Ví dụ về các bộ điều khiển điển hình: Deployment, ReplicaSet, DaemonSet, Job, v.v.

Khi một operator được sử dụng để quản lý ứng dụng, thông số cấu hình (specification) của Pod được lồng bên trong định
nghĩa của bộ điều khiển thông qua Pod Template (khuôn mẫu Pod).

Bên dưới là một ví dụ về bản khai báo định nghĩa đối tượng Pod độc lập ở định dạng YAML, không sử dụng operator:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    run: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:1.22.1
      ports:
        - containerPort: 80
```

apiVersion: Trường apiVersion phải được thiết lập thành v1 cho định nghĩa đối tượng Pod.

kind: Trường bắt buộc thứ hai là kind chỉ định loại đối tượng là Pod.

metadata: Trường bắt buộc thứ ba metadata chứa tên của đối tượng và các nhãn (label) và chú thích (annotation) tùy chọn.

spec: Trường bắt buộc thứ tư spec đánh dấu phần đầu của khối định nghĩa trạng thái mong muốn của đối tượng Pod – cũng được gọi là PodSpec. 
Pod của chúng ta tạo ra một container đơn lẻ chạy image nginx:1.22.1 được kéo từ registry chứa image container, trong trường hợp này là từ Docker Hub. 
Trường containerPort chỉ định cổng container sẽ được Kubernetes resources (tài nguyên Kubernetes) mở ra để truy cập giữa các ứng dụng hoặc truy cập của client bên ngoài – sẽ được đề cập chi tiết hơn trong chương Services (Dịch vụ). 
Nội dung của spec được đánh giá cho mục đích lên lịch (scheduling), sau đó kubelet của nút được chọn sẽ chịu trách nhiệm chạy image container với sự trợ giúp của container runtime trên nút đó. 
Tên và nhãn của Pod được sử dụng cho mục đích theo dõi khối lượng công việc.

Labels:
Nhãn (label) là các cặp khóa-giá trị (key-value) được gắn với các đối tượng Kubernetes (ví dụ: Pod, ReplicaSet, Node, Namespace, Persistent Volume). 
Nhãn được sử dụng để tổ chức và chọn một tập hợp con của các đối tượng, dựa trên các yêu cầu có sẵn. 
Nhiều đối tượng có thể có chung Nhãn (label). 
Các nhãn không đảm bảo tính duy nhất cho các đối tượng. 
Bộ điều khiển (controller) sử dụng nhãn để nhóm các đối tượng tách rời một cách hợp lý, thay vì sử dụng tên hoặc ID của đối tượng.

![lables.png](../images/lables.png)

Trong hình ảnh trên, chúng ta đã sử dụng hai khóa nhãn: app và env. 
Dựa trên yêu cầu của mình, chúng ta đã cung cấp các giá trị khác nhau cho bốn Pods. 
Nhãn env=dev về mặt logic sẽ chọn và nhóm hai Pod trên cùng, trong khi nhãn app=frontend về mặt logic sẽ chọn và nhóm hai Pod bên trái. 
Chúng ta có thể chọn một trong bốn Pods – phía dưới bên trái, bằng cách chọn hai nhãn: app=frontend AND env=qa.


Label Selectors:
Các bộ điều khiển (controller), hoặc các operator, và Services (Dịch vụ) sử dụng bộ chọn nhãn (label selector) để chọn một tập hợp con của các đối tượng. 
Kubernetes hỗ trợ hai loại Bộ chọn:
- Bộ chọn dựa trên phép so sánh bằng (Equality-Based Selectors):  
  Cho phép lọc các đối tượng dựa trên khóa và giá trị của nhãn (Label). 
  Sự trùng khớp được thực hiện bằng cách sử dụng các toán tử =, == (có thể dùng thay thế cho nhau) hoặc != (không bằng). 
  Ví dụ: với env==dev hoặc env=dev chúng ta chọn các đối tượng có khóa nhãn env được đặt thành giá trị dev.

- Bộ chọn dựa trên tập hợp (Set-Based Selectors):  
  Cho phép lọc các đối tượng dựa trên một tập hợp các giá trị. 
- Chúng ta có thể sử dụng các toán tử in (trong tập), notin (không trong tập) cho các giá trị nhãn (Label value) và các toán tử exists (tồn tại), does not exist (không tồn tại) cho khóa nhãn (Label key). 
  Ví dụ:
     + Với env in (dev,qa) chúng ta chọn các đối tượng có nhãn env được đặt thành dev hoặc qa.
     + Với !app chúng ta chọn các đối tượng không có khóa nhãn app.

![selector.png](../images/selector.png)

# ReplicationControllers
Mặc dù không còn được khuyến nghị sử dụng, ReplicationController là một operator phức tạp, đảm bảo một số lượng bản
sao (replica) cụ thể của Pod đang chạy tại bất kỳ thời điểm nào, bằng cách liên tục so sánh trạng thái thực tế với trạng
thái mong muốn của ứng dụng được quản lý. Nếu có nhiều Pod hơn số lượng mong muốn, ReplicationController ngẫu nhiên loại
bỏ số lượng Pod vượt quá, và, nếu có ít Pods hơn số lượng mong muốn, thì ReplicationController yêu cầu tạo thêm Pod cho
đến khi số lượng thực tế khớp với số lượng mong muốn. Nói chung, chúng ta không triển khai Pod độc lập, vì nó sẽ không
thể tự khởi động lại nếu bị chấm dứt do lỗi vì Pod thiếu tính năng tự sửa lỗi (self-healing) mà Kubernetes hứa hẹn.
Phương pháp được khuyến nghị là sử dụng một số loại operator (bộ điều khiển) để chạy và quản lý Pod.

Ngoài việc sao chép (replication), operator ReplicationController còn hỗ trợ cập nhật ứng dụng.

Tuy nhiên, bộ điều khiển được khuyến nghị mặc định là Deployment, cấu hình nên một bộ điều khiển ReplicaSet để quản lý
vòng đời của Pod ứng dụng.


# ReplicaSets (1)
ReplicaSet là một phần kế thừa từ ReplicationController thế hệ tiếp theo, vì nó thực hiện các chức năng sao chép (
replication) và tự sửa lỗi (self-healing) của ReplicationController. ReplicaSet hỗ trợ cả bộ chọn dựa trên phép so sánh
bằng (equality-based) và bộ chọn dựa trên tập hợp (set-based), trong khi ReplicationController chỉ hỗ trợ bộ chọn dựa
trên phép so sánh bằng.

Khi chỉ chạy một phiên bản duy nhất của ứng dụng, luôn có nguy cơ ứng dụng đó gặp sự cố bất ngờ, hoặc toàn bộ máy chủ
chứa ứng dụng đó bị lỗi. Nếu chỉ dựa vào một phiên bản ứng dụng duy nhất, sự cố như vậy có thể ảnh hưởng xấu đến các ứng
dụng, dịch vụ hoặc máy khách khác. Để tránh các sự cố hỏng hóc (failure) tiềm ẩn đó, chúng ta có thể chạy nhiều phiên
bản của ứng dụng song song, do đó đạt được tính khả dụng cao. Vòng đời của ứng dụng được xác định bởi Pod sẽ được giám
sát bởi một bộ điều khiển (controller) – ReplicaSet. ReplicaSet giúp chúng ta mở rộng (scale) số lượng Pod chạy một
image container ứng dụng cụ thể. Việc mở rộng có thể được thực hiện thủ công hoặc bằng cách sử dụng bộ tự động mở rộng (
autoscaler).

Bên dưới chúng ta mô tả hình ảnh một ReplicaSet, với số lượng bản sao (replica) được đặt thành 3 cho một khuôn mẫu Pod
cụ thể. Pod-1, Pod-2, và Pod-3 giống hệt nhau, chạy cùng một image container ứng dụng, được nhân bản từ cùng một khuôn
mẫu Pod. Hiện tại, trạng thái thực tế khớp với trạng thái mong muốn. Tuy nhiên, hãy nhớ rằng mặc dù ba bản sao Pod được
cho là giống hệt nhau – chạy một thể hiện (instance) của cùng một ứng dụng, cùng cấu hình, chúng vẫn khác biệt nhau về
danh tính – tên Pod, địa chỉ IP, và đối tượng Pod đảm bảo rằng ứng dụng có thể được đặt riêng lẻ vào bất kỳ nút worker
nào của cụm do kết quả của quá trình lên lịch (scheduling).

![replicaset.webp](../images/replicaset.webp)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
        - name: php-redis
          image: gcr.io/google_samples/gb-frontend:v3
```

ReplicaSets (2)
Hãy cùng tiếp tục với ví dụ ReplicaSet trước đó và giả sử rằng một trong các Pod bị buộc phải chấm dứt bất ngờ (do không đủ tài nguyên, timeout, node chứa nó gặp sự cố, v.v…), 
khiến trạng thái thực tế không còn khớp với trạng thái mong muốn.

![replicaSet1.png](../images/replicaSet1.png)

ReplicaSet sẽ phát hiện ra trạng thái thực tế không còn khớp với trạng thái mong muốn và kích hoạt yêu cầu tạo thêm một Pod, do đó đảm bảo trạng thái thực tế một lần nữa khớp với trạng thái mong muốn.
![replicaSet2.png](../images/replicaSet2.png)

ReplicaSet có thể được sử dụng độc lập như bộ điều khiển Pod nhưng chúng chỉ cung cấp một tập hợp các tính năng hạn chế. 
Deployments, loại bộ điều khiển được khuyến nghị sử dụng cho việc dàn xếp (orchestration) các Pod, cung cấp một tập hợp các tính năng bổ sung. 
Deployment quản lý việc tạo, xóa và cập nhật Pod. Deployment sẽ tự động tạo ra một ReplicaSet, sau đó ReplicaSet này mới tạo ra Pod. 
Chúng ta không cần quản lý ReplicaSet và Pod một cách riêng biệt, Deployment sẽ quản lý chúng thay cho ta.

Chúng ta sẽ xem xét kỹ hơn về Deployment ở phần tiếp theo.







