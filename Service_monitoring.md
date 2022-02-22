
# 1. plugin
 
- Tất cả các plugin Icinga hoặc Nagios hiện có đều hoạt động với Icinga2. Ví dụ, bạn có thể tìm thấy các plugin cộng đồng trên Icinga Exchange .

- Cách được khuyến nghị để thiết lập các plugin này là sao chép chúng vào thư mục PluginDir.

### Thiết lập plugin

- Đảm bảo rằng plugin đang hoạt động bình thường bằng cách cố gắng chạy nó trên bảng điều khiển bằng cách sử dụng bất kỳ người dùng nào mà Icinga2 đang chạy như:

```sh
sudo -u nagios /usr/lib/nagios/plugins/check_mysql_health --help
 ```
 
 # 2. CheckCommand
 
- Mỗi plugin yêu cầu một object CheckCommand trong cấu hình, object này có thể được sử dụng trong định nghĩa object service hoặc host .

Ví dụ về kiểm tra kích thước CSDL với check_mysql_health.

```sh
/usr/lib64/nagios/plugins/check_mysql_health --hostname '127.0.0.1' --username root --password icingar0xx --mode sql --name 'select sum(data_length + index_length) / 1024 / 1024 from information_schema.tables where table_schema = '\''icinga'\'';' '--name2' 'db_size' --units 'MB' --warning 4096 --critical 8192
```

 ### Tích hợp file cấu hình icinga
 
 ```sh
 object Host "icinga2-master1.localdomain" {
  check_command = "hostalive"
  address = "..."

  // Database listens locally, not external
  vars.mysql_health_hostname = "127.0.0.1"

  // Basic database size checks for Icinga DBs
  vars.databases["icinga"] = {
    mysql_health_warning = 4096 //MB
    mysql_health_critical = 8192 //MB
  }
  vars.databases["icingaweb2"] = {
    mysql_health_warning = 4096 //MB
    mysql_health_critical = 8192 //MB
  }
}
```

- Host object chuẩn bị các ngưỡng và chi tiết của CSDL để áp dụng cho rule. Nó cũng sử dụng các điều kiện để tìm nạp các giá trị được chỉ định trên host hoặc đặt các giá trị mặc định.

```sh
apply Service "db-size-" for (db_name => config in host.vars.databases) {
  check_interval = 1m
  retry_interval = 30s

  check_command = "mysql_health"

  if (config.mysql_health_username) {
    vars.mysql_healt_username = config.mysql_health_username
  } else {
    vars.mysql_health_username = "root"
  }
  if (config.mysql_health_password) {
    vars.mysql_healt_password = config.mysql_health_password
  } else {
    vars.mysql_health_password = "icingar0xx"
  }

  vars.mysql_health_mode = "sql"
  vars.mysql_health_name = "select sum(data_length + index_length) / 1024 / 1024 from information_schema.tables where table_schema = '" + db_name + "';"
  vars.mysql_health_name2 = "db_size"
  vars.mysql_health_units = "MB"

  if (config.mysql_health_warning) {
    vars.mysql_health_warning = config.mysql_health_warning
  }
  if (config.mysql_health_critical) {
    vars.mysql_health_critical = config.mysql_health_critical
  }

  vars += config
}
```


### Checkcommand mới

-  Các quy ước sau khi thêm định nghĩa đối tượng lệnh mới:

`Sử dụng các đối số lệnh bất cứ khi nào có thể. Thuộc tính command phải là một mảng [ ... ] để thoát shell.`
```<command name>_<parameter name>```

- Xem mô tả đối số

```sh
./check_systemd.py --help

usage: check_systemd.py [-h] [-c SECONDS] [-e UNIT | -u UNIT] [-v] [-V]
                        [-w SECONDS]

...

optional arguments:
  -h, --help            show this help message and exit
  -c SECONDS, --critical SECONDS
                        Startup time in seconds to result in critical status.
  -e UNIT, --exclude UNIT
                        Exclude a systemd unit from the checks. This option
                        can be applied multiple times. For example: -e mnt-
                        data.mount -e task.service.
  -u UNIT, --unit UNIT  Name of the systemd unit that is beeing tested.
  -v, --verbose         Increase output verbosity (use up to 3 times).
  -V, --version         show program's version number and exit
  -w SECONDS, --warning SECONDS
                        Startup time in seconds to result in warning status.
 ```
 
 
 - CheckCommand không dối só

```sh
object CheckCommand "systemd" { // Plugin name without 'check_' prefix
  command = [ PluginContribDir + "/check_systemd.py" ] // Use the 'PluginContribDir' constant, see the contributed ITL commands
}
```

Chạy lệnh xác thực cấu hình để xem có hoạt động không
`icinga2 daemon -C`
 
- Phân tích các thông số plugin. Các plugin có đầu ra trợ giúp tốt hiển thị các thông số tùy chọn trong dấu ngoặc vuông. Đây là trường hợp cho tất cả các tham số cho plugin này. Nếu có các tham số bắt buộc, hãy sử dụng required khóa bên trong đối số.

Thuộc tính `arguments` là một từ điển lấy các tham số làm khóa.

```sh
 arguments = {
    "--unit" = { ... }
  }
```
- Bản thân giá trị đối số là một từ điển phụ có các khóa bổ sung:

<ul>
  <ul>
    <li> value tham chiếu đến chuỗi macro đang chạy.
     <li> description nơi bạn sao chép văn bản trợ giúp tham số plugin vào
     <li> required, set_if .v.. để biết các tham số nâng cao.
  </ul>
  </ul>

  
  






