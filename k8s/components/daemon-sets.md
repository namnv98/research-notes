DaemonSets
Đây là bản dịch tiếng Việt của đoạn văn trên về DaemonSet trong Kubernetes, kèm theo một số diễn giải thêm cho rõ về các
khái niệm liên quan:

DaemonSet là các operator được thiết kế để quản lý các tác nhân (agent) trên node. Chúng giống với các operator
ReplicaSet và Deployment khi quản lý việc tạo bản sao Pod và cập nhật ứng dụng, nhưng DaemonSet có một đặc điểm riêng:
đảm bảo chỉ có một bản sao (replica) của Pod được đặt trên mỗi node, trên tất cả các node. Ngược lại, ReplicaSet và
Deployment theo mặc định không kiểm soát việc lên lịch (scheduling) và bố trí (placement) nhiều bản sao Pod trên cùng
một node.

Các loại tác vụ thông thường sử dụng DaemonSet:

Thu thập dữ liệu giám sát từ tất cả các node: DaemonSet đảm bảo một pod giám sát luôn chạy trên tất cả các node

Chạy các daemon lưu trữ, mạng hoặc proxy trên tất cả các node: Chẳng hạn, các daemon này giúp thiết lập mạng liên lạc
giữa các pod trên các node khác nhau.

Các tác vụ quan trọng khác, cần đảm bảo chạy trên tất cả các node

Khả năng tự động hóa của DaemonSet:

Khi một node mới được thêm vào cụm, một Pod từ một DaemonSet nhất định sẽ tự động được đặt trên node đó. Quá trình này
được điều khiển tự động bởi DaemonSet thay vì bởi Scheduler (bộ lên lịch) mặc định.

Khi bất kỳ một node nào bị lỗi hoặc bị xóa khỏi cụm, các Pod tương ứng được vận hành bởi DaemonSet cũng sẽ được dọn
dẹp (garbage collected).

Nếu DaemonSet bị xóa, tất cả các bản sao Pod mà nó đã tạo cũng bị xóa theo.

Chú ý về scheduling đối với DaemonSet: Việc bố trí (placement) các DaemonSet Pod vẫn bị chi phối bởi các thuộc tính lên
lịch, có thể giới hạn việc đặt Pod chỉ trên một tập hợp con của các node trong cụm. Điều này có thể đạt được với sự trợ
giúp của các thuộc tính lên lịch Pod như nodeSelectors, các quy tắc node affinity (thân thiện node), taint và
toleration. Các cơ chế này đảm bảo rằng Pod của DaemonSet chỉ được đặt trên các node cụ thể, chẳng hạn chỉ trên các node
worker nếu được yêu cầu. Tuy nhiên, Scheduler mặc định có thể tiếp quản quá trình lên lịch nếu một tính năng tương ứng
được kích hoạt, cho phép sử dụng các quy tắc node affinity.

Ví dụ về định nghĩa DaemonSet: Dưới đây là một ví dụ về bản khai báo định nghĩa đối tượng DaemonSet ở định dạng YAML:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-agent
  namespace: kube-system
  labels:
    k8s-app: fluentd-agent
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-agent
  template:
    metadata:
      labels:
        k8s-app: fluentd-agent
    spec:
      containers:
        - name: fluentd-agent
          image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```
