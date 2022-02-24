# Mô hình triển khai

                                                         | eth0
                            +----------------------------+----------------------------+
                            |                            |                            |
                            |10.0.0.51                   |10.0.0.52                   |10.0.0.53 
                +-----------+-----------+    +-----------+-----------+    +-----------+-----------+
                |        [master]       |    |      [satellite]      |    |        [agent]        |
                |        icinga2        +----+         icinga2       +----+         icinga2       |
                |        maria-db       |    |                       |    |                       |
                |        mailutils      |    |                       |    |                       |    
                |         ssmtp         |    |                       |    |                       |    
                |                       |    |                       |    |                       |
                +-----------+-----------+    +-----------------------+    +-----------------------+
                       
                       
## 1. Thiết lập trên node master 

- Chạy `winzard node`

```sh
root@quynv:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup)                                                                                                              [Y/n]: n

Starting the Master setup routine...

Please specify the common name (CN) [quynv]: quynv
Reconfiguring Icinga...
Checking for existing certificates for common name 'quynv'...
Certificate '/var/lib/icinga2/certs//quynv.crt' for CN 'quynv' already existing.                                                                                                              Skipping certificate generation.
Generating master configuration for Icinga 2.
'api' feature already enabled.

Master zone name [master]: master

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]: n
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Do you want to disable the inclusion of the conf.d directory [Y/n]: y
Disabling the inclusion of the conf.d directory...
Checking if the api-users.conf file exists...

Done.

Now restart your Icinga 2 daemon to finish the installation!
```

- Tạo ticket client cho node satellite và agent

```sh
root@quynv:~# icinga2 pki ticket --cn "satellite"
78d86f2e08e3f84f653f3b769d6181f114245ff0
root@quynv:~# icinga2 pki ticket --cn "agent"
575ec25b3b96b892cee1ec0312907e0a3956634d

root@quynv:~# systemctl restart icinga2.service

```


## 2. Thiết lập trên node satellite

- Chạy `wizard node`

```sh
root@satellite:~# systemctl restart icinga2
root@satellite:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]: y

Starting the Agent/Satellite setup routine...

Please specify the common name (CN) [satellite]: satellite

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): quynv

Do you want to establish a connection to the parent node from this node? [Y/n]: y
Please specify the master/satellite connection information:
Master/Satellite endpoint host (IP address or FQDN): 10.0.0.51
Master/Satellite endpoint port [5665]: 5665

Add more master/satellite endpoints? [y/N]: n
Parent certificate information:

 Version:             3
 Subject:             CN = quynv
 Issuer:              CN = Icinga CA
 Valid From:          Feb 17 07:57:36 2022 GMT
 Valid Until:         Feb 13 07:57:36 2037 GMT
 Serial:              cb:ad:a4:bf:0e:df:d0:6f:86:f2:16:0e:19:f9:30:d6:9b:03:34:71

 Signature Algorithm: sha256WithRSAEncryption
 Subject Alt Names:   quynv
 Fingerprint:         CF 6C E6 50 CA C2 FD 81 90 B5 C5 02 2D 42 31 FD 58 80 BC E1 E3 E1 A0 AE 18 E7 8E 84 06 FE 25 12

Is this information correct? [y/N]: y

Please specify the request ticket generated on your Icinga 2 master (optional).
 (Hint: # icinga2 pki ticket --cn 'satellite'): 78d86f2e08e3f84f653f3b769d6181f114245ff0
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Accept config from parent node? [y/N]: y
Accept commands from parent node? [y/N]: y

Reconfiguring Icinga...

Local zone name [satellite]:
Parent zone name [master]: master

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]: n

Do you want to disable the inclusion of the conf.d directory [Y/n]: y
Disabling the inclusion of the conf.d directory...

Done.

Now restart your Icinga 2 daemon to finish the installation!

root@satellite:~# systemctl restart icinga2.service
```



## 3. Thiết lập trên node agent

- Chạy `wizard node`

```sh
root@agent:~# icinga2 node wizard
Welcome to the Icinga 2 Setup Wizard!

We will guide you through all required configuration details.

Please specify if this is an agent/satellite setup ('n' installs a master setup) [Y/n]: y

Starting the Agent/Satellite setup routine...

Please specify the common name (CN) [agent]: agent

Please specify the parent endpoint(s) (master or satellite) where this node should connect to:
Master/Satellite Common Name (CN from your master/satellite node): satellite

Do you want to establish a connection to the parent node from this node? [Y/n]: y
Please specify the master/satellite connection information:
Master/Satellite endpoint host (IP address or FQDN): 10.0.0.52
Master/Satellite endpoint port [5665]: 5665

Add more master/satellite endpoints? [y/N]: n
Parent certificate information:

 Version:             3
 Subject:             CN = satellite
 Issuer:              CN = Icinga CA
 Valid From:          Feb 23 03:54:22 2022 GMT
 Valid Until:         Feb 19 03:54:22 2037 GMT
 Serial:              3e:f6:a5:93:0e:3a:a6:14:72:27:98:1c:fa:a2:06:b0:b0:38:5d:fc

 Signature Algorithm: sha256WithRSAEncryption
 Subject Alt Names:   satellite
 Fingerprint:         66 40 76 FF CF 97 E0 EB 17 77 86 27 8F C5 D4 C2 6E 74 4F 3F BB E7 CF C9 D3 44 99 BB 5D 47 F0 68

Is this information correct? [y/N]: y

Please specify the request ticket generated on your Icinga 2 master (optional).
 (Hint: # icinga2 pki ticket --cn 'agent'): 575ec25b3b96b892cee1ec0312907e0a3956634d
Please specify the API bind host/port (optional):
Bind Host []:
Bind Port []:

Accept config from parent node? [y/N]: y
Accept commands from parent node? [y/N]: y

Reconfiguring Icinga...
Disabling feature notification. Make sure to restart Icinga 2 for these changes to take effect.
Enabling feature api. Make sure to restart Icinga 2 for these changes to take effect.

Local zone name [agent]:
Parent zone name [master]: satellite

Default global zones: global-templates director-global
Do you want to specify additional global zones? [y/N]: n

Do you want to disable the inclusion of the conf.d directory [Y/n]: y
Disabling the inclusion of the conf.d directory...

Done.

Now restart your Icinga 2 daemon to finish the installation!
root@agent:~# systemctl restart icinga2.service
```

```sh
root@agent:~# vim /etc/icinga2/features-enabled/api.conf
object ApiListener "api" {
  accept_config = true
  accept_commands = true
}
```


## 4. Cấu hình giám sát node agent trên node master

- Thêm thông tin cấu hình endpoint và zone của `satellite`

```sh
root@quynv:~# vim /etc/icinga2/zones.conf

object Endpoint "quynv" {
}

object Zone "master" {
        endpoints = [ "quynv" ]
}

object Zone "global-templates" {
        global = true
}

object Zone "director-global" {
        global = true
}

object Zone "satellite" {
  endpoints = [ "satellite" ]
  parent = "master"
}


object Endpoint "satellite" {
  host = "10.0.0.52"
  log_duration = 0 // Disable the replay log for command endpoint agents
}

```

- Tạo thư mục satellite và tạo file cấu hình zone cho `agent`


```sh
root@quynv:~# mkdir -p /etc/icinga2/zones.d/satellite
root@quynv:~# cd /etc/icinga2/zones.d/satellite/
root@quynv:/etc/icinga2/zones.d/satellite# vim agent.conf
object Zone "agent" {
  endpoints = [ "agent" ]
  parent = "satellite"
}

object Endpoint "agent" {
  host = "10.0.0.53"
  log_duration = 0 // Disable the replay log for command endpoint agents
}
```

- Tạo file cấu hình host cho `agent`

```sh
root@quynv:/etc/icinga2/zones.d/satellite# vim hosts.conf

object Host "agent" {
  check_command = "hostalive"
  address = "10.0.0.53"
  vars.agent_endpoint = name
}
```

- Tạo file cấu hình dịch vụ cho  `agent`

```sh
root@quynv:/etc/icinga2/zones.d/satellite# vim service.conf
apply Service "ping4" {
  check_command = "ping4"
  assign where host.zone == "satellite" && host.address
}

apply Service "disk" {
  check_command = "disk"
  // Execute the check on the remote command endpoint
  command_endpoint = host.vars.agent_endpoint

  // Assign the service onto an agent
  assign where host.zone == "satellite" && host.address
}

```
- Kiểm tra cấu hình và restart dịch vụ

```sh
root@quynv:/etc/icinga2/zones.d/satellite# icinga2 daemon -C
[2022-02-23 08:42:36 +0000] information/cli: Icinga application loader (version: r2.13.2-1)
[2022-02-23 08:42:36 +0000] information/cli: Loading configuration file(s).
[2022-02-23 08:42:36 +0000] information/ConfigItem: Committing config item(s).
[2022-02-23 08:42:36 +0000] information/ApiListener: My API identity: quynv
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 IcingaApplication.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 Host.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 FileLogger.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 CheckerComponent.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 ApiListener.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 IdoMysqlConnection.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 5 Zones.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 3 Endpoints.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 2 ApiUsers.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 245 CheckCommands.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 1 NotificationComponent.
[2022-02-23 08:42:36 +0000] information/ConfigItem: Instantiated 3 Services.
[2022-02-23 08:42:36 +0000] information/ScriptGlobal: Dumping variables to file '/var/cache/icinga2/icinga2.vars'
[2022-02-23 08:42:36 +0000] information/cli: Finished validating the configuration file(s).
root@quynv:/etc/icinga2/zones.d/satellite# systemctl restart icinga2.service
```

- Kiểm tra trên Dashboard

<img src = "https://github.com/lean15998/Icinga/blob/main/image/5.06.PNG">



## 5. Cảnh báo

- Mở tính năng `notification`

```sh
root@quynv:~# icinga2 feature list
Disabled features: command compatlog debuglog elasticsearch gelf graphite icingadb influxdb influxdb2 livestatus opentsdb perfdata statusdata syslog
Enabled features: api checker ido-mysql mainlog notification
```

- Cài đặt mailutils và sSMTP

```sh
root@quynv:~# apt install -y mailutils ssmtp
```

- Cấu hình ssmtp

```sh
root@quynv:~# vim /etc/ssmtp/ssmtp.conf
root=quy15091998@gmail.com

mailhub=smtp.gmail.com:587
UseSTARTTLS=YES
AuthUser=quy15091998@gmail.com
AuthPass=Abc123456789
rewriteDomain=gmail.com
hostname=quynv
FromLineOverride=YES
```

- Cấu hình gửi cảnh báo cho người dùng `agent`

```sh
root@quynv:/etc/icinga2/zones.d/satellite# vim user.conf

object User "agent" {
  display_name = "agent"
  enable_notifications =true
  states = [Down, Up, OK, Warning, Unknown]
  types = [Problem, Recovery]
  email = "quy15091998@gmail.com"
}
```

- Gửi cảnh báo cho host `agent`

```sh
object Host "agent" {
  check_command = "hostalive"
  address = "10.0.0.53"
  vars.agent_endpoint = name
  vars.notification["mail"] = {
  users = ["agent"]
}
}
```

- Kiểm tra cấu hình và khởi động lại dịch vụ

```sh
root@quynv:/etc/icinga2/zones.d/satellite# icinga2 daemon -C
[2022-02-24 01:32:40 +0000] information/cli: Icinga application loader (version: r2.13.2-1)
[2022-02-24 01:32:40 +0000] information/cli: Loading configuration file(s).
[2022-02-24 01:32:40 +0000] information/ConfigItem: Committing config item(s).
[2022-02-24 01:32:40 +0000] information/ApiListener: My API identity: quynv
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 16 Notifications.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 IcingaApplication.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 2 HostGroups.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 2 NotificationCommands.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 2 Hosts.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 Downtime.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 FileLogger.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 CheckerComponent.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 ApiListener.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 IdoMysqlConnection.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 5 Zones.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 3 Endpoints.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 2 ApiUsers.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 245 CheckCommands.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 NotificationComponent.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 UserGroup.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 2 Users.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 3 TimePeriods.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 3 ServiceGroups.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 1 ScheduledDowntime.
[2022-02-24 01:32:40 +0000] information/ConfigItem: Instantiated 14 Services.
[2022-02-24 01:32:40 +0000] information/ScriptGlobal: Dumping variables to file '/var/cache/icinga2/icinga2.vars'
[2022-02-24 01:32:40 +0000] information/cli: Finished validating the configuration file(s).

root@quynv:/etc/icinga2/zones.d/satellite# systemctl restart icinga2.service
```

- Trên dashboard

<img src = "https://github.com/lean15998/Icinga/blob/main/image/5.07.PNG" >

- Check mail cảnh báo

<img src = "https://github.com/lean15998/Icinga/blob/main/image/5.04.PNG">


## 6.Thêm plugin giám sát

- Tải plugin trên mạng và lưu vào thư mục `/usr/lib/nagios/plugins/`

- Thêm cấu hình Checkcommand trên node `master`

```sh
root@quynv:~# vim /usr/share/icinga2/include/plugins-contrib.d/operating-system.

object CheckCommand "mem" {
        command = [ PluginDir + "/check_mem.pl", "-w 20", "-c 10" ]
}

object CheckCommand "cpu" {
        command = [ PluginDir + "/check_cpu", "-w 20", "-c 10" ]
}

}
```
- Thêm cấu hình service vào file cấu hình

```sh
root@quynv:~# vim /etc/icinga2/zones.d/satelite/services.conf

/.........

apply Service "Memory" {
  check_command = "mem"
  command_endpoint = host.vars.agent_endpoint
  assign where host.zone == "satelite" && host.address
}

apply Service "Cpu" {
  check_command = "cpu"
  command_endpoint = host.vars.agent_endpoint
  assign where host.zone == "satelite" && host.address
}

```

- Kiểm tra cấu hình và khởi động lại icinga2

```sh
root@quynv:~# icinga2 daemon -C
[2022-02-23 10:13:20 +0000] information/cli: Icinga application loader (version: r2.13.2-1)
[2022-02-23 10:13:20 +0000] information/cli: Loading configuration file(s).
[2022-02-23 10:13:20 +0000] information/ConfigItem: Committing config item(s).
[2022-02-23 10:13:20 +0000] information/ApiListener: My API identity: quynv
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 IcingaApplication.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 Host.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 FileLogger.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 CheckerComponent.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 ApiListener.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 IdoMysqlConnection.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 5 Zones.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 3 Endpoints.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 2 ApiUsers.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 245 CheckCommands.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 1 NotificationComponent.
[2022-02-23 10:13:20 +0000] information/ConfigItem: Instantiated 4 Services.
[2022-02-23 10:13:20 +0000] information/ScriptGlobal: Dumping variables to file '/var/cache/icinga2/icinga2.vars'
[2022-02-23 10:13:20 +0000] information/cli: Finished validating the configuration file(s).
root@quynv:~# systemctl restart icinga2.service
```

- Kiểm tra trên dashboard

<img src = "https://github.com/lean15998/Icinga/blob/main/image/5.05.PNG">

