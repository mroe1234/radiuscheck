This script provides both a nagios/icinga check as well as a telegraf exec plugin.
This tool depends on eapol_test.  It can be found:

http://deployingradius.com/scripts/eapol_test/

I build this script based on eapol_test v2.8-hostap_2_8. If they change the tool, this may break  
I used jq (https://stedolan.github.io/jq/) to do float math since bash only supports integer math.
If you use the telegraf option, you will need to install jq.

Here is my telegraf plugin config.  I am querying two radius servers thus the two commands:

```
[[inputs.exec]]
  commands = [
    "/usr/local/bin/radiuscheck -t -h nps1",
    "/usr/local/bin/radiuscheck -t -h nps2",
  ]
  timeout = "5s"
  data_format = "influx"
```

Here is an example nrpe config for the check:
```
command[check_radius_cert_nps1]=/usr/local/bin/radiuscheck -n -h 192.168.0.201 -w 30 -c 0
command[check_radius_cert_nps2]=/usr/local/bin/radiuscheck -n -h 192.168.0.239 -w 30 -c 0
```

<strong>OPTIONS:</strong>  
  -h host <required>  
  -n nagios  
  -t telegraf  
  -w warning (only makes sense when invoking with -n)  
  -c critical (only makes sense when invoking with -n)  

if neither -n nor -t are provided the tool does nothing.  


<strong>Example output:</strong>  

Telegraf:  
```
$ ./radiuscheck -t -h nps1
radius,host=nps1 authtime=0.037
```
Nagios:  
```
$ /usr/local/bin/radiuscheck -n -h nps1 -w 30 -c 0
OK;Certificate expires on Jul  8 15:10:35 2021 GMT
```