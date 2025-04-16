Ứng dụng dạng container được triển khai trong cụm Kubernetes có thể cần kết nối đến các ứng dụng container khác, hoặc có
thể cần phải truy cập được bởi các ứng dụng khác, hoặc thậm chí bởi các client bên ngoài. Điều này trở thành khó khăn
bởi vì container không tự mở cổng (port) ra mạng của cụm, và nó cũng không thể tự được khám phá (discoverable). Một giải
pháp đơn giản là ánh xạ cổng (port mapping) như được cung cấp bởi các máy chủ container thông thường. Tuy nhiên, do sự
phức tạp của framework Kubernetes, việc ánh xạ cổng đơn giản như vậy lại không “đơn giản” tí nào. Giải pháp thực tế
trong Kubernetes phức tạp hơn nhiều, với sự tham gia của tác nhân kube-proxy node, IP tables, các quy tắc định tuyến,
máy chủ DNS của cụm; tất cả các thành phần này cùng nhau thực hiện một cơ chế cân bằng tải cấp vi mô (micro-load
balancing) để mở cổng của container ra mạng lưới của cụm, thậm chí ra thế giới bên ngoài nếu cần. Cơ chế này được gọi là
Service (Dịch vụ), và nó là phương thức được khuyến nghị để mở ứng dụng dạng container ra mạng Kubernetes. Lợi ích của
Kubernetes Service trở nên rõ ràng hơn khi mở một ứng dụng với nhiều bản sao (replica), khi nhiều container chạy cùng
một image cần mở cùng một cổng. Đây là tình huống mà việc ánh xạ cổng đơn giản của một máy chủ container sẽ không hoạt
động, nhưng Service lại không gặp vấn đề gì khi triển khai một yêu cầu phức tạp như vậy.