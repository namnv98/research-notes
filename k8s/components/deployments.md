# Deployments

Đối tượng Deployment cung cấp các bản cập nhật mang tính khai báo (declarative) cho Pod và ReplicaSet.  
DeploymentController là một phần của trình quản lý bộ điều khiển (controller manager) trên control plane node, và với
vai trò là một bộ điều khiển, nó đảm bảo rằng trạng thái hiện tại luôn khớp với trạng thái mong muốn của ứng dụng dạng
container đang chạy.  
Nó cho phép cập nhật và khôi phục (rollback) ứng dụng liền mạch, được biết đến như chiến lược RollingUpdate mặc định,
thông qua các hoạt động rollout (triển khai) và rollback (khôi phục), đồng thời trực tiếp quản lý các ReplicaSet của nó
để mở rộng quy mô ứng dụng.   
Nó cũng hỗ trợ một chiến lược cập nhật khác ít phổ biến hơn, được gọi là Recreate (tái tạo).

Dưới đây là một ví dụ về bản khai báo định nghĩa đối tượng Deployment ở định dạng YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
        - name: nginx
          image: nginx:1.20.2
          ports:
            - containerPort: 80
```

- apiVersion: Trường bắt buộc đầu tiên này chỉ định điểm cuối API (API endpoint) trên API Server mà chúng ta muốn
  kết nối; nó phải khớp với phiên bản API hiện có cho loại đối tượng được định nghĩa.

- kind: Trường thứ hai bắt buộc, chỉ định loại đối tượng – trong trường hợp của chúng ta, nó là Deployment, nhưng
  nó cũng có thể là Pod, ReplicaSet, Namespace, Service, v.v.

- metadata: Trường thứ ba bắt buộc, chứa thông tin cơ bản của đối tượng, như tên (name), chú thích (annotation),
  nhãn (label), namespace, v.v.

- spec: Trường thứ tư bắt buộc này đánh dấu phần đầu của khối định nghĩa trạng thái mong muốn của đối tượng
  Deployment. Trong ví dụ, chúng ta đang yêu cầu 3 replica, tức là 3 phiên bản của Pod, đang chạy tại bất kỳ thời điểm
  nào.
  Các Pod được tạo bằng cách sử dụng khuôn mẫu Pod (Pod Template) được định nghĩa trong spec.template.
- Một đối tượng được lồng bên trong (nested object), chẳng hạn như Pod là một phần của Deployment, sẽ giữ lại metadata
  và spec của nó,
  và bỏ đi apiVersion và kind – cả hai đều được thay thế bằng template (khuôn mẫu).
  Trong spec.template.spec, chúng ta định nghĩa trạng thái mong muốn của Pod. Pod của chúng ta tạo ra một container đơn
  lẻ chạy image nginx:1.20.2 từ Docker Hub.

Khi đối tượng Deployment được tạo, hệ thống Kubernetes sẽ đính kèm trường status vào đối tượng và điền vào đó tất cả các
trường trạng thái cần thiết.

Ví dụ về Deployment và ReplicaSet:

Trong ví dụ sau, một Deployment mới tạo ra ReplicaSet A, sau đó tạo ra 3 Pod, với mỗi Pod Template được cấu hình để chạy
một container image nginx:1.20.2.  
Trong trường hợp này, ReplicaSet A được liên kết với nginx:1.20.2 đại diện cho một trạng thái của Deployment.  
Trạng thái cụ thể này được ghi lại là Bản sửa đổi (Revision) 1.

![Deployment.png](../images/Deployment.png)

Theo thời gian, chúng ta cần cập nhật ứng dụng được quản lý bởi đối tượng Deployment. Giả sử chúng ta muốn thay đổi Pod
Template và cập nhật ảnh container từ nginx:1.20.2 lên nginx:1.21.5. Deployment sẽ kích hoạt một ReplicaSet B mới cho
phiên bản ảnh container mới 1.21.5 và sự liên kết này đại diện cho trạng thái mới được ghi lại của Deployment, Revision
2 (Bản sửa đổi 2). Quá trình chuyển đổi liền mạch giữa hai ReplicaSet, từ ReplicaSet A với ba Pod phiên bản 1.20.2 sang
ReplicaSet B mới với ba Pod phiên bản 1.21.5, hay từ Revision 1 sang Revision 2, được gọi là cập nhật rolling update của
Deployment.

Một vài điểm cần lưu ý thêm về cập nhật rolling update:

- Deployment theo dõi cả ReplicaSet A và ReplicaSet B cho đến khi tất cả các Pod mong muốn của ReplicaSet B (thường là
  tất
  cả Pod) đang chạy và ổn định. Khi đó, Deployment sẽ loại bỏ các Pod cũ từ ReplicaSet A theo cách được kiểm soát (
  rolling
  update) để đảm bảo tính khả dụng cao của ứng dụng.

- Cập nhật rolling update là chiến lược mặc định của Deployment và nó giúp giảm thiểu thời gian chết (downtime) của ứng
  dụng trong suốt quá trình cập nhật.

Rolling update (cập nhật luân phiên) được kích hoạt khi chúng ta cập nhật các thuộc tính cụ thể của Pod Template trong
Deployment.
Trong khi các thay đổi có chủ đích như cập nhật image container, cổng container (container port), volume và
điểm gắn kết (mount) sẽ kích hoạt một Revision (Bản sửa đổi) mới, các hoạt động khác có bản chất động, ví dụ như mở rộng
quy mô hoặc thêm label cho Deployment, không kích hoạt rolling update, do đó không làm thay đổi số Revision.  

Khi rolling update đã hoàn tất, Deployment sẽ hiển thị cả ReplicaSet A và ReplicaSet B, trong đó A được mở rộng xuống
0 (zero) Pod và B được mở rộng lên 3 Pods.  
Đây là cách Deployment ghi lại các cấu hình trạng thái trước đó của nó dưới dạng các Revision (Bản sửa đổi).  

![deployment2.png](../images/deployment2.png)

Một khi ReplicaSet B và 3 Pods phiên bản 1.21.5 của nó đã sẵn sàng, Deployment bắt đầu quản lý chúng một cách chủ động.  
Tuy nhiên, Deployment vẫn giữ lại cấu hình các trạng thái trước đó được lưu dưới dạng Revision (bản sửa đổi), đóng vai
trò then chốt trong khả năng rollback (khôi phục) của Deployment – cho phép quay trở về trạng thái cấu hình được biết
đến trước đó. Trong ví dụ của chúng ta, nếu hiệu suất của nginx:1.21.5 không đạt yêu cầu, Deployment có thể được
rollback (khôi phục) về Revision trước đó, trong trường hợp này là từ Revision 2 quay lại Revision 1 đang chạy nginx:
1.20.2.  

![deployment3.png](../images/deployment3.png)


