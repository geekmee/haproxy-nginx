# haproxy-nginx
To demo how HAProxy works with Nginx



## On master, install and configure up haproxy
```vagrant ssh master
sudo su

vi /etc/hosts
- 192.168.10.104 nginx1
- 192.168.10.105 nginx2

yum -y install haproxy
cd /etc/haproxy
vi haproxy.cfg #copy/paste haproxy-nginx/haproxy.cfg
vi /etc/rsyslog.conf #uncomment
 - $ModLoad imudp
 - $UDPServerRun 514
vi /etc/rsyslog.d/haproxy.conf #adding
local2.=info     /var/log/haproxy-access.log    #For Access Log
local2.notice    /var/log/haproxy-info.log      #For Service Info - Backend, loadbalancer

systemctl restart rsyslog
systemctl start haproxy
systemctl enable haproxy
```

## On nodes
```
vagrant ssh nginx1
sudo su
vi /etc/hosts
- 192.168.10.104 nginx1
- 192.168.10.105 nginx2
- 192.168.10.102 master
yum -y install epel-release
yum -y install nginx
cd /usr/share/nginx/html/
echo "<h1>echo from nginx1</h1>" > index.html 
echo "<h1>echo from nginx2</h1>" > index.html (for nginx2)
systemctl enable nginx
systemctl start nginx
```

## test
curl 192.168.10.102

## statistics
http://192.168.1.102:8080/stats

## Ref
- https://www.howtoforge.com/tutorial/how-to-setup-haproxy-as-load-balancer-for-nginx-on-centos-7/
- CentOS Vagrant Box https://app.vagrantup.com/centos/boxes/7
