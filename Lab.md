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

```sh
root@quynv:~# icinga2 pki ticket --cn "satellite"
78d86f2e08e3f84f653f3b769d6181f114245ff0
root@quynv:~# icinga2 pki ticket --cn "agent"
575ec25b3b96b892cee1ec0312907e0a3956634d

root@quynv:~# systemctl restart icinga2.service

```


## 2. Thiết lập trên node satellite

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








