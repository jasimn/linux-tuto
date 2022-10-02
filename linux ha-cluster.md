
Implementing HA cluster in centos 7

Server1 

yum install pacemaker pcs resource-agents httpd -y

echo 'hacluster:Password1' | chpasswd

firewall-cmd --permanent --add-service=high-availability
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
systemctl enable pcsd
systemctl start pcsd

Server 2 

yum install pacemaker pcs resource-agents httpd -y

echo 'hacluster:Password1' | chpasswd

firewall-cmd --permanent --add-service=high-availability
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

systemctl enable pcsd
systemctl start pcsd
systemctl enable corosync pacemaker


Server1

pcs cluster auth server1.example.com server2.example.com -u hacluster Password1
pcs cluster setup --name mycluster server1.example.com server2.example.com
pcs cluster start --all
pcs status

systemctl enable corosync pacemaker 
pcs property set stonith-enabled=false
pcs property set no-quorum-policy=ignore

pcs status
pcs config

pcs resource create cluster_ip ocf:heartbeat:IPaddr2 ip=192.168.56.115 cidr_netmask=24 op monitor interval=20s

pcs status
pcs config

firewall-cmd --permanent --add-service=http
firewall-cmd --reload

systemctl disable httpd
systemctl stop httpd

vim /etc/httpd/conf/httpd.conf
    
apachectl configtest

vim /etc/httpd/conf.d/status.conf

<Location /server-status>
    SetHandler server-status
    Require ip 127.0.0.1
</Location>


apachectl configtest

Vim /var/www/html/index.html
<h1>Welcome</h1>
<br>
<hr>
Server1

pcs resource create web-server ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl=http://127.0.0.1/server-status op monitor interval=20s
pcs constraint colocation add web-server cluster_ip INFINITY
 pcs constraint show --full


Server2

pcs cluster standby server1.example.com
pcs cluster unstandby server1.example.com

vim /etc/httpd/conf.d/status.conf # paste below connent 

<Location /server-status>
    SetHandler server-status
    Require ip 127.0.0.1
</Location>

apachectl configtest

vim /var/www/html/index.html

<h1>Welcome</h1>
<br>
<hr>
Server2

pcs cluster standby server1.example.com
