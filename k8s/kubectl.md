các câu lệnh `kubectl` phổ biến** cùng giải thích chi tiết, phân loại theo nhóm chức năng:

---

## **📋 1. Lệnh cơ bản**

| Lệnh                        | Mô tả                                             | Ví dụ                     |
|-----------------------------|---------------------------------------------------|---------------------------|
| **`kubectl version`**       | Hiển thị phiên bản client và server               | `kubectl version --short` |
| **`kubectl cluster-info`**  | Hiển thị thông tin cluster (API server, DNS, ...) | `kubectl cluster-info`    |
| **`kubectl config view`**   | Xem cấu hình kubeconfig hiện tại                  | `kubectl config view`     |
| **`kubectl api-resources`** | Liệt kê tất cả resource types được hỗ trợ         | `kubectl api-resources`   |

---

## **🛠️ 2. Quản lý Pod**

| Lệnh                                           | Mô tả                                   | Ví dụ                                        |
|------------------------------------------------|-----------------------------------------|----------------------------------------------|
| **`kubectl get pods`**                         | Liệt kê Pod trong namespace hiện tại    | `kubectl get pods -n default`                |
| **`kubectl describe pod <pod-name>`**          | Xem chi tiết Pod (events, logs, status) | `kubectl describe pod nginx`                 |
| **`kubectl logs <pod-name>`**                  | Hiển thị logs của Pod                   | `kubectl logs nginx -f` (theo dõi real-time) |
| **`kubectl exec -it <pod-name> -- <command>`** | Chạy lệnh trong Pod                     | `kubectl exec -it nginx -- /bin/bash`        |
| **`kubectl delete pod <pod-name>`**            | Xóa Pod                                 | `kubectl delete pod nginx`                   |

---

## **🚀 3. Quản lý Deployment**

| Lệnh                                                      | Mô tả                       | Ví dụ                                       |
|-----------------------------------------------------------|-----------------------------|---------------------------------------------|
| **`kubectl get deployments`**                             | Liệt kê Deployments         | `kubectl get deployments`                   |
| **`kubectl create deployment <name> --image=<image>`**    | Tạo Deployment              | `kubectl create deploy nginx --image=nginx` |
| **`kubectl scale deployment <name> --replicas=<number>`** | Thay đổi số replicas        | `kubectl scale deploy nginx --replicas=3`   |
| **`kubectl rollout status deployment/<name>`**            | Kiểm tra trạng thái rollout | `kubectl rollout status deploy/nginx`       |
| **`kubectl rollout undo deployment/<name>`**              | Rollback về phiên bản trước | `kubectl rollout undo deploy/nginx`         |

---

## **🔗 4. Quản lý Service**

| Lệnh                                                               | Mô tả                     | Ví dụ                                                   |
|--------------------------------------------------------------------|---------------------------|---------------------------------------------------------|
| **`kubectl get services`**                                         | Liệt kê Services          | `kubectl get svc`                                       |
| **`kubectl expose deployment <name> --port=<port> --type=<type>`** | Tạo Service từ Deployment | `kubectl expose deploy nginx --port=80 --type=NodePort` |
| **`kubectl describe service <service-name>`**                      | Xem chi tiết Service      | `kubectl describe svc nginx`                            |

---

## **🔐 5. Quản lý ConfigMap & Secret**

| Lệnh                                                     | Mô tả                 | Ví dụ                                                 |
|----------------------------------------------------------|-----------------------|-------------------------------------------------------|
| **`kubectl create configmap <name> --from-file=<file>`** | Tạo ConfigMap từ file | `kubectl create cm my-config --from-file=config.yaml` |
| **`kubectl get secrets`**                                | Liệt kê Secrets       | `kubectl get secrets`                                 |
| **`kubectl describe secret <secret-name>`**              | Xem chi tiết Secret   | `kubectl describe secret my-secret`                   |

---

## **📂 6. Quản lý Namespace**

| Lệnh                                                          | Mô tả              | Ví dụ                                               |
|---------------------------------------------------------------|--------------------|-----------------------------------------------------|
| **`kubectl get namespaces`**                                  | Liệt kê namespaces | `kubectl get ns`                                    |
| **`kubectl create namespace <name>`**                         | Tạo namespace mới  | `kubectl create ns staging`                         |
| **`kubectl config set-context --current --namespace=<name>`** | Chuyển namespace   | `kubectl config set-context --current --ns=staging` |

---

## **🔌 7. Port Forwarding & Proxy**

| Lệnh                                                          | Mô tả                                | Ví dụ                                |
|---------------------------------------------------------------|--------------------------------------|--------------------------------------|
| **`kubectl port-forward <pod-name> <local-port>:<pod-port>`** | Chuyển tiếp cổng từ Pod sang local   | `kubectl port-forward nginx 8080:80` |
| **`kubectl proxy`**                                           | Tạo proxy để truy cập Kubernetes API | `kubectl proxy --port=8001`          |

---

## **🐞 8. Debugging & Troubleshooting**

| Lệnh                                                           | Mô tả                              | Ví dụ                                                         |
|----------------------------------------------------------------|------------------------------------|---------------------------------------------------------------|
| **`kubectl top pod`**                                          | Hiển thị CPU/memory usage của Pods | `kubectl top pod --sort-by=memory`                            |
| **`kubectl get events --sort-by=.metadata.creationTimestamp`** | Xem events theo thời gian          | `kubectl get events --field-selector=involvedObject.kind=Pod` |
| **`kubectl apply -f <file.yaml>`**                             | Áp dụng cấu hình từ file YAML      | `kubectl apply -f deployment.yaml`                            |
| **`kubectl explain <resource>`**                               | Hiển thị tài liệu về resource      | `kubectl explain pod.spec.containers`                         |

---

## **🗑️ 9. Xóa Resource**

| Lệnh                                | Mô tả                            | Ví dụ                                                |
|-------------------------------------|----------------------------------|------------------------------------------------------|
| **`kubectl delete -f <file.yaml>`** | Xóa resource từ file YAML        | `kubectl delete -f deployment.yaml`                  |
| **`kubectl delete all --all`**      | Xóa mọi resource trong namespace | `kubectl delete all --all -n staging` (⚠️ Cẩn thận!) |

---

## **🎯 10. Lệnh Nâng Cao**

| Lệnh                                                         | Mô tả                           | Ví dụ                                                    |
|--------------------------------------------------------------|---------------------------------|----------------------------------------------------------|
| **`kubectl apply -k <kustomize-directory>`**                 | Áp dụng Kustomize configuration | `kubectl apply -k ./overlays/prod`                       |
| **`kubectl get pods --field-selector=status.phase=Running`** | Lọc Pod theo trạng thái         | `kubectl get pods --field-selector=status.phase=Pending` |
| **`kubectl auth can-i <verb> <resource>`**                   | Kiểm tra quyền                  | `kubectl auth can-i delete pods`                         |

---

## **💡 Tips & Tricks**

- **Output format**: Sử dụng `-o wide`, `-o yaml`, `-o json` để xem chi tiết hơn.  
  Ví dụ: `kubectl get pods -o wide`.
- **Watch mode**: Thêm `--watch` để theo dõi real-time.  
  Ví dụ: `kubectl get pods --watch`.
- **Alias**: Thêm `alias k=kubectl` vào `.bashrc`/`.zshrc` để gõ lệnh nhanh hơn.
- **Auto-completion**: Bật auto-completion cho kubectl:
  ```bash
  source <(kubectl completion bash)  # Bash
  source <(kubectl completion zsh)   # Zsh
  ```

---

## **🎯 Advanced Logging**

```
# Aggregate logs từ tất cả Pod có label cụ thể
kubectl logs -l app=nginx --tail=50 --all-containers=true

# Xem logs từ Pod đã bị terminated
kubectl logs -p <pod-name> 

```

---
