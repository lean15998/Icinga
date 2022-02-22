# 1. Host và Service

- Icinga2 có thể được sử dụng để theo dõi tính khả dụng của các Host và Service. Host và Service hầu như có thể là bất kỳ thứ gì có thể được kiểm tra theo một cách nào đó:
  <ul>
   <ul>
      <li> Dịch vụ mạng (HTTP, SMTP, SNMP, SSH, v.v.)
      <li> Máy in
      <li> Switch và Router
      <li> Các dịch vụ mạng hoặc cục bộ khác có thể truy cập được
      <li> v.v..
      </ul>
 </ul>
 
 ### Trạng thái của Host
 
  - UP: Host khả dụng
  - DOWN : Host không khả dụng
  
  ### Trạng thái của Service
  
- OK : Service đang hoạt động tốt
- WARNING : Service đang có 1 số vấn đề nhưng vẫn trong tình trạng làm việc.
- CRITICAL : Service đang trong trạng thái nguy hiểm
- UNKNOW : Không thể kiểm tra trạng thái của Service

###  Trạng thái Hard và Soft

- Khi có vấn đề xảy ra với host/service, Icinga2 sẽ không lập tức gửi cảnh báo đi ngay, mà sẽ tiến hành kiểm tra lại trước khi gửi cảnh báo, nhằm tránh việc những cảnh báo không quan trọng được gửi đi liên tục.
- Trong thời gian kiểm tra này, object rơi vào trạng thái SOFT
- Sau khi tất cả re-check được tiến hành, và object được kiểm tra vẫn trong trạng thái non-OK, thì host/service sẽ chuyển sang trạng thái HARD và cảnh báo được gửi đi.
- Để thay đổi thời gian và số lần kiểm tra, hãy tìm đến /etc/icinga2/conf.d/templates.conf
- Template Icinga2 cung cấp sẵn các template chứa các thông số cơ bản cho host, service, hay việc gửi cảnh báo. Ta cũng có thể tự tạo template để tùy chỉnh theo nhu cầu và import template mới vào các object.



```sh
root@quynv:~# tail -5 /etc/icinga2/conf.d/templates.conf
template Host "Host01" {
  max_check_attempts = 3
  check_interval = 1m
  retry_interval = 30s
}
```


### Kiểm tra host và Service

Đây là các service được icinga cung cấp sẵn để monitor

| Service(s) | Applied on host(s) |
| --- | --- |
|  load, procs, swap, users, icinga | The NodeName host only |
|  ping4, ping6 | All hosts with address resp. address6 attribute |
| ssh | All hosts with address and vars.os set to Linux |
| http, optional: Icinga Web 2 | All hosts with custom attribute http_vhosts defined as dictionary |
| disk, disk / | All hosts with custom attribute disks defined as dictionary |


```sh
root@quynv:~# tail -5 /etc/icinga2/conf.d/hosts.conf

object Host "local" {
  address = "10.0.0.51"
  check_command = "hostalive"
}
```

```sh
root@quynv:~# tail -5 /etc/icinga2/conf.d/services.conf
object Service "http" {
  host_name = "local"
  check_command = "ping4"
}
```


# 2. Template

  - Template có thể được sử dụng để áp dụng một tập hợp các thuộc tính giống hệt nhau cho nhiều đối tượng:

  ```sh
  root@quynv:~# cat /etc/icinga2/conf.d/templates.conf
  template Service "generic-service" {
  max_check_attempts = 3
  check_interval = 5m
  retry_interval = 1m
  enable_perfdata = true
}

apply Service "ping4" {
  import "generic-service"

  check_command = "ping4"

  assign where host.address
}

apply Service "ping6" {
  import "generic-service"

  check_command = "ping6"

  assign where host.address6
}
```

 - Import template
 
```sh
root@quynv:~# tail -6 /etc/icinga2/conf.d/services.conf
object Service "http" {
  import = "generic-service"
  host_name = "local"
  check_command = "ping4"
}
```

# 3. Biến tuỳ chỉnh

 - Có thể xác định các thuộc tính tuỳ chỉnh bằng cách dùng biến **vars**.

```sh
object Host "localhost" {
  check_command = "ssh"
  vars.ssh_port = 2222
}
```

```
vars["ssh_port"] = 2222
```

```sh
  vars.disks["disk /"] = {
    disk_partitions = "/"
  }
  
```
Hoặc có thể viết như sau:

```sh
  vars = {
    disks = {
      "disk /" = {
        disk_partitions = "/"
      }
    }
  }
```
- Hàm và biến tuỳ chỉnh

```sh
object CheckCommand "random-value" {
  command = [ PluginDir + "/check_dummy", "0", "$text$" ]

  vars.text = {{ Math.random() * 100 }}
}

object CheckCommand "my-ping" {
  command = [ PluginDir + "/check_ping" ]

  arguments = {
    "-H" = "$ping_address$"
    "-w" = "$ping_wrta$,$ping_wpl$%"
    "-c" = "$ping_crta$,$ping_cpl$%"
    "-p" = "$ping_packets$"
  }

  // Resolve from a host attribute, or custom variable.
  vars.ping_address = "$address$"

  // Default values
  vars.ping_wrta = 100
  vars.ping_wpl = 5

  vars.ping_crta = 250
  vars.ping_cpl = 10

  vars.ping_packets = 5
}

```

# 4. Runtime macro



# 5. Apply Rules


###  Giám sát tất cả các host có host.address được chỉ định

```sh
apply Service "ping4" {
  check_command = "ping4"
  assign where host.address
}
```

### Áp dụng dịch vụ cho host

- Áp dụng dịch vụ cho host có host.address chỉ định và host os là Linux.

```sh
apply Service "ssh" {
  import "generic-service"

  check_command = "ssh"

  assign where host.address && host.vars.os == "Linux"
}

```

### Áp dụng Notifications cho Hosts và Services

- `mail-noc` thông báo sẽ được tạo dưới dạng đối tượng cho tất cả các dịch vụ có biến tuỳ chỉnh `notification.mail` được xác định. Lệnh thông báo được đặt thành mail-service-notification và tất cả các thành viên của nhóm người dùng noc sẽ nhận được thông báo.

```sh
apply Notification "mail-noc" to Service {
  import "mail-service-notification"

  user_groups = [ "noc" ]

  assign where host.vars.notification.mail
}
```

- Template Notìication `mail-host-notification` chứa tất cả các cài đặt thông báo có liên quan. Apply rule được áp dụng trên tất cả các host nơi quy tắc `host.address` được xác định.

Nếu host có một tập biến tùy chỉnh cụ thể, giá trị của nó sẽ được kế thừa vào phạm vi đối tượng thông báo cục bộ, ví dụ host.vars.notification_interval: host.vars.notification_periodvà host.vars.notification_type. Điều này ghi đè các thuộc tính đã được chỉ định trong template `mail-host-notification` đã nhập.

```sh
apply Notification "host-mail-noc" to Host {
  import "mail-host-notification"

  // replace interval inherited from `mail-host-notification` template with new notfication interval set by a host custom variable
  if (host.vars.notification_interval) {
    interval = host.vars.notification_interval
  }

  // same with notification period
  if (host.vars.notification_period) {
    period = host.vars.notification_period
  }

  // Send SMS instead of email if the host's custom variable `notification_type` is set to `sms`
  if (host.vars.notification_type == "sms") {
    command = "sms-host-notification"
  } else {
    command = "mail-host-notification"
  }

  user_groups = [ "noc" ]

  assign where host.address
}
```

Host tương ứng

```sh
object Host "host1" {
  import "host-linux-prod"
  display_name = "host1"
  address = "192.168.1.50"
  vars.notification_interval = 1h
  vars.notification_period = "24x7"
  vars.notification_type = "sms"
}
```


# 6. Groups
 - Tập hợp các đối tượng giống nhau

```sh
object HostGroup "windows" {
  display_name = "Windows Servers"
}
```

- Thêm host vào group

```sh
template Host "windows-server" {
  groups += [ "windows" ]
}

object Host "mssql-srv1" {
  import "windows-server"

  vars.mssql_port = 1433
}

object Host "mssql-srv2" {
  import "windows-server"

  vars.mssql_port = 1433
}
```

- Có thể áp dụng cho nhóm dịch vụ và nhóm người dùng

```sh
object UserGroup "windows-mssql-admins" {
  display_name = "Windows MSSQL Admins"
}

template User "generic-windows-mssql-users" {
  groups += [ "windows-mssql-admins" ]
}

object User "win-mssql-noc" {
  import "generic-windows-mssql-users"

  email = "noc@example.com"
}

object User "win-mssql-ops" {
  import "generic-windows-mssql-users"

  email = "ops@example.com"
}
```

### Chỉ định tư cách thành viên nhóm

- Thay vì chỉ định thủ công từng đối tượng cho một nhóm, bạn cũng có thể gán các đối tượng cho một nhóm dựa trên các thuộc tính của chúng:

```sh
object HostGroup "prod-mssql" {
  display_name = "Production MSSQL Servers"

  assign where host.vars.mssql_port && host.vars.prod_mysql_db
  ignore where host.vars.test_server == true
  ignore where match("*internal", host.name)
}
```

 Tất cả các host có thuộc tính `mssql_port` sẽ được thêm làm thành viên của hostgroup `mssql`. Tuy nhiên, tất cả các host phù hợp với chuỗi thuộc tính `internal` hoặc với `test_server` không được thêm vào nhóm này.

# 7. Thông báo

- Thông báo cho các vấn đề về service và host là một phần không thể thiếu trong thiết lập giám sát.
- Khi service và host đang gặp sự cố thì thông báo sẽ được gửi đến cho người dùng.
- Có nhiều cách để gửi thông báo, ví dụ như qua email, XMPP, IRC, Twitter, v.v. .Icinga2 không biết cách gửi thông báo. Thay vào đó, nó dựa vào các cơ chế bên ngoài như shell script để thông báo cho người dùng.

```sh
object User "icingaadmin" {
  display_name = "Icinga 2 Admin"
  enable_notifications = true
  states = [ OK, Warning, Critical ]
  types = [ Problem, Recovery ]
  email = "icinga@localhost"
}
```

- User `icingaadmin` trong ví dụ dưới đây sẽ chỉ nhận được thông báo về các trạng thái `Warning` và `Critical`. Thêm vào đó các Recovery thông báo được gửi .
- Nếu không set `states` và `types` thì tất cả các thông báo về trạng thái sẽ được gửi.

- Ví dụ một notification template 

```sh
template Notification "generic-notification" {
  interval = 15m

  command = "mail-service-notification"

  states = [ Warning, Critical, Unknown ]
  types = [ Problem, Acknowledgement, Recovery, Custom, FlappingStart,
            FlappingEnd, DowntimeStart, DowntimeEnd, DowntimeRemoved ]

  period = "24x7" //Khoảng thơi gian 24*7
}
```

- Apply keyword tạo notification cho service

```sh
apply Notification "notify-cust-xy-mysql" to Service {
  import "generic-notification"
  users = [ "noc-xy", "mgmt-xy" ]
  assign where match("*has gold support 24x7*", service.notes) && (host.vars.customer == "customer-xy" || host.vars.always_notify == true
  ignore where match("*internal", host.name) || (service.vars.priority < 2 && host.vars.is_clustered == true)
}
```

### Thông báo : User từ host/service

 - Sử dụng các quy tắc `mail` và `sms` cho các đối tượng khác nhau

```sh

object Host "icinga2-agent1.localdomain" {
  [...]

  vars.notification["mail"] = {
    groups = [ "icingaadmins" ]
    users = [ "icingaadmin" ]
  }
  vars.notification["sms"] = {
    users = [ "icingaadmin" ]
  }
}
```

- Sử dụng chi tiết hơn trên service

```sh
apply Service "http" {
  [...]

  vars.notification["mail"] = {
    groups = [ "icingaadmins" ]
    users = [ "icingaadmin" ]
  }

  [...]
}
```
- Dịch vụ thông báo chó user và group được kế thừa từ service và nếu không được thiết lập, từ host. Người dùng mặc định cũng được đặt.


```sh
apply Notification "mail-service-notification" to Service {
  [...]

  if (service.vars.notification.mail.users) {
    users = service.vars.notification.mail.users
  } else if (host.vars.notification.mail.users) {
    users = host.vars.notification.mail.users
  } else {
    /* Default user who receives everything. */
    users = [ "icingaadmin" ]
  }

  if (service.vars.notification.mail.groups) {
    user_groups = service.vars.notification.mail.groups
  } else if (host.vars.notification.mail.groups) {
    user_groups = host.vars.notification.mail.groups
  }

  assign where ( host.vars.notification.mail && typeof(host.vars.notification.mail) == Dictionary ) || ( service.vars.notification.mail && typeof(service.vars.notification.mail) == Dictionary )
}
```

### Độ trễ thông báo và thời gian re-notification

```sh
apply Notification "mail" to Service {
  import "generic-notification"

  command = "mail-notification"
  users = [ "icingaadmin" ]

  interval = 5m  // time renotification
  
  times.begin = 15m // delay notification window

  assign where service.name == "ping4"
}





