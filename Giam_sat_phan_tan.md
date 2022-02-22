#  Roles

- Một nút master nằm trên cùng của hệ thống phân cấp.
- Một nút satellite là con của một nút master.
- Một nút agent được kết nối với nút satellite và nút master.

<img src= "https://github.com/lean15998/Icinga/blob/main/image/4.01.png" >

- Một nút master không có nút cha.
<ul>
  <ul>
  <li> Nút master là nơi bạn thường cài đặt Icinga Web 2.
  <li> Một nút master có thể kết hợp các kiểm tra đã thực thi từ các nút con thành phần phụ trợ và thông báo.
 </ul>
</ul>
  
- Một nút satellite có một nút cha và một nút con.
<ul>
  <ul>
  <li> Một nút satellite có thể tự thực hiện kiểm tra hoặc ủy quyền thực thi kiểm tra cho các nút con.
  <li> Một nút satellite có thể nhận cấu hình cho các máy chủ / dịch vụ, v.v. từ nút cha.
  <li> Một nút satellite tiếp tục chạy ngay cả khi nút master tạm thời không khả dụng.
  </ul>
</ul>
    
- Một nút agent chỉ có một nút cha.
<ul>
  <ul>
    <li> Một nút agent sẽ chạy các kiểm tra được cấu hình của chính nó hoặc nhận các sự kiện thực thi lệnh từ nút cha.
  </ul>
</ul>

Client có thể là master, satellite hoặc agent. Nó thường yêu cầu một cái gì đó từ nút master hoặc nút cha.


# Zone

- Hệ thống phân cấp Icinga2 bao gồm cái gọi là các zone object . Các zone phụ thuộc vào mối quan hệ cha parent-child tin tưởng nhau.

<img src = "https://github.com/lean15998/Icinga/blob/main/image/4.02.png">

 - Cấu hình các nút satellite có nút master là zone parent:

```sh
object Zone "master" {
   //...
}

object Zone "satellite region 1" {
  parent = "master"
  //...
}

object Zone "satellite region 2" {
  parent = "master"
  //...
}
```

- Hạn chế nhất định đối với các zone con, ví dụ như các thành viên của chúng không được phép gửi các lệnh cấu hình cho các thành viên zone cha. Ngược lại, hệ thống phân cấp tin cậy cho phép ví dụ vùng master gửi các tệp cấu hình đến vùng satellite.
- Agent node cũng có zone duy nhất của riêng chúng.

# Endpoint

- Các node là thành viên của một zone được gọi là các đối tượng endpoint.

<img src="https://github.com/lean15998/Icinga/blob/main/image/4.03.png">

- File cấu hình của 2 endpoint ở các vùng khác nhau

```sh
object Endpoint "master1" {
  host = "192.168.56.101"
}

object Endpoint "atellite1" {
  host = "192.168.56.105"
}

object Zone "master" {
  endpoints = [ "master1" ]
}

object Zone "satellite" {
  endpoints = [ "satellite1" ]
  parent = "master"
}
```
# Apilistener

- Trong trường hợp bạn đang sử dụng các lệnh CLI sau này, bạn không cần phải viết cấu hình này từ đầu trong trình soạn thảo văn bản. Đối tượng ApiListener được sử dụng để tải chứng chỉ TLS và chỉ định các hạn chế, ví dụ: chấp nhận các lệnh cấu hình.
- Nó cũng được sử dụng cho Icinga 2 REST API chia sẻ cùng một máy chủ và cổng với giao thức Icinga 2 Cluster.
- Cấu hình object được lưu trữ trong tệp `/etc/icinga2/features-enabled/api.conf`. Tùy thuộc vào chế độ cấu hình, các thuộc tính `accept_commands` và `accept_config` có thể được cấu hình ở đây.
Để sử dụng tính năng này, bạn cần kích hoạt nó và khởi động lại Icinga2.

```sh
icinga2 feature enable api
```

# Quy ước

Theo quy ước, tất cả các nút phải được cấu hình bằng FQDN của chúng.

Hơn nữa, bạn phải đảm bảo rằng các tên sau đây hoàn toàn giống nhau trong tất cả các tệp cấu hình:
<ul>
  <ul>
  <li> Tên chung của chứng chỉ máy chủ (CN).
  <li> Đối tượng cấu hình endpoint cho host.
  <li> Hằng số NodeName cho localhost.
  </ul>
</ul>

# Master Setup










































