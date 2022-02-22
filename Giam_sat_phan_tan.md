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

- Cách cài đặt một nút chính trung tâm bằng lệnh `node wizard`.

- Chạy lệnh CLI `node wizard`. Trước đó, hãy đảm bảo thu thập thông tin cần thiết:

| Tham số |	Miêu tả |
| -- | -- |
| Tên thường gọi (CN)	| Yêu cầu. Theo quy ước, đây phải là FQDN của máy chủ. Mặc định là FQDN. |
| Master zone name |	(Không bắt buộc) Cho phép chỉ định tên vùng master. Mặc định là master |
| Global zones	| (Không bắt buộc) Cho phép chỉ định nhiều zone global hơn ngoài global-templates và director-global. Mặc định là `N`. |
| API bind host |	(Không bắt buộc) Cho phép chỉ định địa chỉ mà ApiListener bị ràng buộc. Chỉ dành cho mục đích sử dụng nâng cao. |
| API bind port |	(Không bắt buộc) Cho phép chỉ định cổng mà ApiListener bị ràng buộc. Chỉ để sử dụng nâng cao (cổng mặc định 5665). |
| Disable conf.d |	(Không bắt buộc) Cho phép vô hiệu hóa include_recursive "conf.d"chỉ thị ngoại trừ tệp api-users.conf trong tệp icinga2.conf. Mặc định là `Y`. |


Trình hướng dẫn thiết lập sẽ đảm bảo rằng các bước sau được thực hiện:


<ul>
  <ul>
  <li> Bật tính năng api.
  <li> Tạo một tổ chức phát hành chứng chỉ (CA) mới /var/lib/icinga2/ca nếu nó không tồn tại.
  <li> Tạo chứng chỉ cho nút này được ký bởi khóa CA.
  <li> Cập nhật tệp zone.conf với cấu trúc phân cấp vùng mới.
  <li> Cập nhật cấu hình ApiListener và hằng số .
  <li> Cập nhật icinga2.conf để tắt tệp conf.d và thêm tệp api-users.conf.



# Signing Certificates on the Master 

- Tất cả các chứng chỉ phải được ký bởi cùng một tổ chức phát hành chứng chỉ (CA). Điều này đảm bảo rằng tất cả các nút tin cậy lẫn nhau trong một môi trường giám sát phân tán.

- CA này được tạo trong quá trình thiết lập chính và phải giống nhau trên tất cả các phiên bản chính.

- Bạn có thể tránh ký và triển khai chứng chỉ theo cách thủ công bằng cách sử dụng các phương pháp tích hợp sẵn cho các yêu cầu ký chứng chỉ tự động ký (CSR):

    <ul>
      <ul>
    <li> CSR Auto-Signing  sử dụng vé máy khách (đại lý hoặc vệ tinh) được tạo trên thẻ chính làm định danh tin cậy.
    <li> On-Demand CSR Signing cho phép ký các yêu cầu chứng chỉ đang chờ xử lý trên bản chính.
      </ul>
    </ul>

### CSR Auto-Signing 

- Một client có thể là master, satellite hoặc agent. Nó gửi một yêu cầu ký chứng chỉ (CSR) và phải tự xác thực theo cách đáng tin cậy. Master tạo một thẻ khách hàng được bao gồm trong yêu cầu này. Bằng cách đó, tổng thể có thể xác minh rằng yêu cầu khớp với vé tin cậy trước đó và ký vào yêu cầu.

Ưu điểm:

- Các nút (master, satellite, agent) có thể được cài đặt bởi những người dùng khác nhau đã nhận được ticket client.
- Không cần tương tác thủ công trên nút master.

Nhược điểm:

- Các ticket cần được tạo trên master và được sao chép vào các trình wizard của client.
- Không trung tâm quản lý chữ ký.
    
### SCR Auto-Signing : Chuẩn bị
- Trước khi sử dụng chế độ này, hãy đảm bảo rằng các bước sau được thực hiện ký trên master:

   <ul>
     <ul>
      <li> Thiết lập master đã được chạy thành công. Điêu nay bao gôm:(Đã tạo một cặp khóa CA), (Đã tạo salt ticket bí mật được lưu trữ trong hằng số TicketSalt , được đặt làm thuộc tính ticket_salt bên trong tính năng api .)
      <li> Khởi động lại cá thể chính.
     </ul>
   </ul>
    
### CSR Auto-Signing: On the master
    
- Trình wizard cho các nút satellite/agent sẽ yêu cầu bạn cung cấp ticket client.

- Có hai cách có thể để lấy lại ticket:

       <ul>
     <ul>
      <li> Lệnh CLI được thực thi trên nút master.
      <li> Yêu cầu API REST đối với nút master.
     </ul>
   </ul>

- Thông tin bắt buộc
    
| Parameter	| Description |
| -- | -- |
| Common name (CN) |	Required. The common name for the agent/satellite. By convention this should be the FQDN. |
    
    
- Tạo một ticket trên nút master(master1) cho agent(agent1):

```sh
  [root@master1 /]# icinga2 pki ticket --cn "agent1"
```
- Truy vấn Icinga2 API trên master yêu cầu đối tượng ApiUser có ít nhất quyền actions/generate-ticket.

```sh
    [root@master1 /]# vim /etc/icinga2/conf.d/api-users.conf

object ApiUser "client-pki-ticket" {
  password = "bea11beb7b810ea9ce6ea" //change this
  permissions = [ "actions/generate-ticket" ]
}

[root@icinga2-master1 /]# systemctl restart icinga2

Retrieve the ticket on the master node `icinga2-master1.localdomain` with `curl`, for example:

 [root@master1 /]# curl -k -s -u client-pki-ticket:bea11beb7b810ea9ce6ea -H 'Accept: application/json' \
 -X POST 'https://localhost:5665/v1/actions/generate-ticket' -d '{ "cn": "icinga2-agent1.localdomain" }'
 ```

- Lưu trữ ticket đó cho satellite/agent để thiết lập sau này.



### On-Demand CSR Signing

.....................
    
    
    
# Agent/Satellite Setup

- Tạo một vé trên nút master(master1) cho agent(agent1):
 ```sh
[root@master1 /]# icinga2 pki ticket --cn "agent1"
4f75d2ecd253575fe9180938ebff7cbca262f96e
```
- Chạy "node wizard" và nhập theo yêu cầu
    
    ```sh
    [root@agent1 /]# icinga2 node wizard
    ```
    
<ul>
  <ul>
    <li> Common name (CN)
    <li>  Master/Satellite Common Name
    <li> Master/Satellite endpoint host
    <li> Master/Satellite endpoint port
    <li> Ticket 
  </ul>
  </ul>
        
 - Restart dịch vụ icinga2
    
```sh
[root@agent1 /]# systemctl restart icinga2
```
      
- Tổng quan về tất cả các thông số:
    
| Tham số | Mô tả |
|  -- | -- |
| Common name (CN)	| Yêu cầu. Theo quy ước, đây phải là FQDN của máy chủ. Mặc định là FQDN. |
| Master common name	| Yêu cầu. Sử dụng tên chung mà bạn đã chỉ định cho nút chính của mình trước đây. |
| Establish connection to the parent node	| Không bắt buộc. Liệu nút có cố gắng kết nối với nút cha hay không. Mặc định là y. |
| Master/Satellite endpoint host |	Bắt buộc nếu đại lý cần kết nối với tổng thể / vệ tinh. Địa chỉ IP của điểm cuối chính hoặc FQDN. Thông tin này được bao gồm trong Endpointcấu hình đối tượng trong zones.conftệp. |
| Master/Satellite endpoint port	| Tùy chọn nếu đại lý cần kết nối với chính / vệ tinh. Cổng lắng nghe của thiết bị đầu cuối chính. Thông tin này được bao gồm trong Endpointcấu hình đối tượng. |
| Add more master/satellite endpoints	| Không bắt buộc. Nếu bạn đã định cấu hình nhiều nút chính / nút vệ tinh, hãy thêm chúng vào đây. |
| Parent Certificate information	| Yêu cầu. Xác minh rằng máy chủ kết nối thực sự là nút chính được yêu cầu. |
| Request ticket	| Không bắt buộc. Thêm vé được tạo trên cái chính. |
| API bind host	| Không bắt buộc. Cho phép chỉ định địa chỉ mà ApiListener bị ràng buộc. Chỉ dành cho mục đích sử dụng nâng cao. |
| API bind port	| Không bắt buộc. Cho phép chỉ định cổng mà ApiListener bị ràng buộc. Chỉ để sử dụng nâng cao (yêu cầu thay đổi cổng mặc định 5665 ở mọi nơi). |
| Accept config	| Không bắt buộc. Liệu nút này có chấp nhận đồng bộ cấu hình từ nút chính (bắt buộc đối với chế độ đồng bộ cấu hình ) hay không. Vì lý do bảo mật , điều này được mặc định thành n. |
| Accept commands	| Không bắt buộc. Liệu nút này có chấp nhận thông báo thực thi lệnh từ nút chính hay không (bắt buộc đối với chế độ điểm cuối lệnh ). Vì lý do bảo mật , điều này được mặc định thành n. |
| Local zone name	| Không bắt buộc. Cho phép chỉ định tên cho vùng cục bộ. Điều này rất hữu ích khi trường hợp này là một vệ tinh, không phải một tác nhân. Mặc định là FQDN.  |
| Parent zone name	| Không bắt buộc. Cho phép chỉ định tên cho vùng mẹ. Điều này quan trọng nếu tác nhân có một cá thể vệ tinh là mẹ, không phải là chính. Mặc định là master. |
| Global zones	| Không bắt buộc. Cho phép chỉ định nhiều khu vực toàn cầu hơn ngoài global-templatesvà director-global. Mặc định là n. |
| Disable conf.d	| Không bắt buộc. Cho phép vô hiệu hóa việc bao gồm thư mục conf.dchứa cấu hình ví dụ cục bộ. Khách hàng nên truy xuất cấu hình của họ từ nút cha hoặc hoạt động như cầu nối thực thi điểm cuối lệnh. Mặc định là y. |
      
      
      
      
      
      
      
      
      


