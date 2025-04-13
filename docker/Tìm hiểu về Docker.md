Với việc phát triển ứng dụng thông thường, chúng ta thường chạy trực tiếp trên máy thật (host) để build ứng dụng. Và một
ứng dụng chúng ta không chỉ có một service mà cần nhiều service hay package gồm các ngôn ngữ khác nhau. Với từng
service, package đó chúng ta phải cài đặt tỉ mỉ đảm bảo ghi rõ từng version của package hay ngôn ngữ sử dụng để có thể
tương thích và hoạt động tốt được.

Mỗi khi bàn giao cho khách hàng hay cho các developer khác để bảo trì ứng dụng, họ sẽ phải cài lại từng package đó,
nhưng không thể tránh được trường hợp cài sai version hay confict, và phải mất quá nhiều thời gian cho việc set-up môi
trường. Docker ra đời đã giải quyết được bài toán đó, công cụ này mang lại cho chúng ta sự tiện lợi khi tất cả những thứ
cần thiết của ứng dụng được đóng gói (images) và containerization (theo cách triển khai container của docker) để việc
maintain, build ứng dụng trở nên nhanh chóng hơn bao giờ hết.

### Docker là gì

Docker là một nền tảng cho developers và sysadmin cho phép triển khai và chạy các ứng dụng dưới dạng container. Nó cho
phép tạo các môi trường độc lập và tách biệt để khởi chạy và phát triển ứng dụng và môi trường này được gọi là
Container. Khi cần triển khai lên bất kỳ server nào chỉ cần run Container của Docker thì application của bạn sẽ được
khởi chạy ngay lập tức, nó đảm bảo ứng dụng chạy được và giống nhau ở các máy khác nhau (Linux, Windows, Desktop,
Server ...).

### Container và Virtual machine

Khi so sánh Container và Virtual Machine là đang đề cập đến công nghệ ảo hoá đằng sau chúng hay còn được gọi là
Container-Based Virtualization và VM-Based Virtualization.

Cả Container-Based và VM-Based đều được sử dụng để ảo hóa, tức là chia sẻ và quản lý tài nguyên phần cứng như RAM, CPU,
Network, ... từ một máy chủ đơn lẻ. Sự khác biệt chính giữa hai kỹ thuật này là:

VM-Based: Ảo hóa tài nguyên ở cấp phần cứng. Mỗi máy ảo hoạt động như một hệ thống độc lập với hệ điều hành riêng biệt.
Để thực hiện việc này, cần sử dụng một phần mềm gọi là hypervisor. Ví dụ, một máy chủ vật lý có thể chạy nhiều máy ảo,
mỗi máy ảo có hệ điều hành riêng như Windows, Linux.
Container-Based: Ảo hóa tài nguyên ở trên cấp (phía trên) hệ điều hành. Tất cả các container trên một máy chủ chia sẻ
cùng một hệ điều hành chủ, nhưng mỗi container vẫn hoạt động như một ứng dụng độc lập. Để thực hiện việc này, cần sử
dụng container engine như Docker. Ví dụ, nhiều ứng dụng có thể chạy trong các container khác nhau trên cùng một hệ điều
hành Linux, mà không cần mỗi container có hệ điều hành riêng.

### Image và Container

Images là một file read-only, không thể thay đổi được, nó chứa các libraries, dependencies, và file, những cấu hình cần
thiết chạy container. Chúng ta không thể start hoặc run images giống như container nhưng có thể tạo được image cho chính
mình hoặc lấy images đã public trên registry để về và customize thành một image gồm những công cụ cần thiết cho riêng
mình.

Một container cuối cùng chỉ là một image đang chạy. Docker container là một run-time environment mà ở đó người dùng có
thể chạy một ứng dụng độc lập. Những container này rất gọn nhẹ và cho phép bạn chạy ứng dụng trong đó rất nhanh chóng và
dễ dàng. Container hoạt động độc lập, nó đảm bảo không làm ảnh hưởng xấu đến các container khác, cũng như server mà nó
đang chạy trong đó. Docker được cho là "tạo ra sự độc lập tuyệt vời". Vì vậy, bạn sẽ không cần lo lắng việc máy tính của
bạn bị xung đột do ứng dụng đang được phát triển được chạy trong container.

### Registry

Docker registry là một kho lưu trữ nơi bạn có thể lưu trữ các Docker images và chia sẻ chúng với người khác. Docker
registry có thể được tổ chức thành các Docker repositories, và trong mỗi repository, bạn có thể duy trì các phiên bản cụ
thể của một Docker image. Ví dụ Docker Hub là một Public Registries do Docker cung cấp, cho phép tìm kiếm và chia sẻ các
Docker image.

### Dockerfile

Để tạo ra một image, chúng ta cần tạo một Docker File. Docker dựa vào file này để sử dụng và build ra images. Docker
file bao gồm tất cả các instruction (hướng dẫn) để docker build ra một image.

Hãy xem ví dụ về dockerfile của 1 ứng dụng .Net:

```
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY ./app .
COPY ./appsettings.json appsettings.json
EXPOSE 8012
ENTRYPOINT ["dotnet", "Demo.QLTS.Api.dll"]
```

- FROM: đây là nơi khai báo ra image base, mình lấy image aspnet với tag là 6.0 để sử dụng.
- WORKDIR: Là chỉ thị dùng để thiết lập thư mục làm việc. Nó giống home directory, trong trường hợp này là home
  directory
  của container. Khi gọi WORKDIR nó sẽ tạo ra thư mục ngay lần gọi đầu và truy cập đến nó như home directory. Ở đây
  thiết
  lập thư mục làm việc hiện tại ở container là /app
- COPY: Chỉ thị này dùng để copy tệp tin hoặc thư mục mới từ source (src) và chuyển đến destination (dest) trong hệ
  thống
  tệp tin của container. Ở đây copy thư mục ./app ở máy host chuyển đến thư mục làm việc hiện tạo ở container.
- EXPOSE: Lệnh EXPOSE thông báo cho Docker cho container lắng nghe trên các cổng mạng được chỉ định khi chạy. Ở đây
  container lắng nghe ở cổng 8012.
- ENTRYPOINT: thường dùng để thực thi nhiều câu lệnh trong quá trình start container. Ở đây là chạy file thực thi
  Demo.QLTS.Api.dll.

### Các lệnh docker cơ bản

docker --version // Kiểm tra phiên bản Docker

docker image ls // Liệt kê danh sách các image đang có ở máy host

docker image rm <<IMAGE_ID>> // Xóa 1 image có id là image_id

docker pull <<IMAGE_NAME>>:<<IMAGE_TAG>> // Download 1 image có tên image_name và tag là image_tag từ registry về

docker push <<IMAGE_NAME>>:<<IMAGE_TAG>> // Upload 1 image có tên image_name và tag là image_tag lên registry

docker build [OPTIONS] PATH | URL | - // Build 1 image từ 1 Dockerfile với PATH là đường dẫn đến docker file

docker image tag <<SOURCE_IMAGE>>:<<TAG>> <<TARGET_IMAGE>>:<<TAG>> // Tạo 1 tag mới có tên source_image tham chiếu đến
target_image

docker run [OPTIONS] <<IMAGE_ID>> [COMMAND] [ARG...]  // Chạy 1 container mới từ image có id là IMAGE_ID

docker container ls [OPTIONS] // Liệt kê các container

docker exec [OPTIONS] <<CONTAINER_ID>> COMMAND [ARG...] // Chạy một lệnh trên container đang chạy
























