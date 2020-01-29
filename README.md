# haproxy-nginx
To demo how HAProxy works with Nginx



## On master
configure hosts
```vagrant ssh master
sudo su

vi /etc/hosts
  192.168.10.104 nginx1
  192.168.10.105 nginx2
```

install and configure haproxy 
`yum -y install haproxy`

vi /etc/haproxy/haproxy.cfg (we can just copy/paste haproxy-nginx/haproxy.cfg)

```
listen haproxy3-monitoring *:8080                #Haproxy Monitoring run on port 8080
    mode http
    option forwardfor
    option httpclose
    stats enable
    stats show-legends
    stats refresh 5s
    stats uri /stats                             #URL for HAProxy monitoring
    stats realm Haproxy\ Statistics
    stats auth admin:admin            #User and Password for login to the monitoring dashboard
    stats admin if TRUE
    default_backend app-main                    #This is optionally for monitoring backend
    
frontend main
    bind *:80
    option http-server-close
    option forwardfor
    default_backend app-main
 
backend app-main
    balance roundrobin                                     #Balance algorithm
    option httpchk HEAD / HTTP/1.1\r\nHost:\ localhost    #Check the server application is up and healty - 200 status code
    server nginx1 192.168.10.104:80 check                 #Nginx1
    server nginx2 192.168.10.105:80 check                 #Nginx2
```

configure rsyslog for HAProxy. We will configure the rsyslog daemon to log the HAProxy statistics. Edit the rsyslog.conf file to enable the UDP port 514 to be used by rsyslog.
```
vi /etc/rsyslog.conf (uncoment following lines)
 $ModLoad imudp
 $UDPServerRun 514
```
If you want to use a specific IP, you can add a new line like the one below:

```
$UDPServerAddress 127.0.0.1
```

Then create new haproxy configuration file for rsyslog.
```
vi /etc/rsyslog.d/haproxy.conf #adding
local2.=info     /var/log/haproxy-access.log    #For Access Log
local2.notice    /var/log/haproxy-info.log      #For Service Info - Backend, loadbalancer
```

start haproxy
```
systemctl restart rsyslog
systemctl start haproxy
systemctl enable haproxy
```

## On nodes
```
vagrant ssh nginx1
sudo su
vi /etc/hosts
  192.168.10.104 nginx1
  192.168.10.105 nginx2
  192.168.10.102 master

yum -y install epel-release
yum -y install nginx

echo "<h1>echo from nginx1</h1>" > /usr/share/nginx/html/index.html 
echo "<h1>echo from nginx2</h1>" > /usr/share/nginx/html/index.html (for nginx2)

systemctl enable nginx
systemctl start nginx
```

## test
curl 192.168.10.102

load test using JMeter
vagrant ssh master
./load-test.sh


## statistics
HAProxy web monitoring (username/password: admin/admin, set in file /etc/haproxy/haproxy.cfg)
http://192.168.10.102:8080/stats

## Ref
- https://www.howtoforge.com/tutorial/how-to-setup-haproxy-as-load-balancer-for-nginx-on-centos-7/
- CentOS Vagrant Box https://app.vagrantup.com/centos/boxes/7
