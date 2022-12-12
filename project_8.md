
## **PROJECT 8: Load Balancer Solution With Apache**
---
---
</br>
</br>

### **CONFIGURE APACHE AS A LOAD BALANCER**
</br>

Configure Apache As A Load Balancer

Create an Ubuntu Server 20.04 EC2 instance and name it `Project-8-apache-lb`

</br>

![EC2](./images-project8/EC2.PNG)

</br>

1. Open `TCP port 80` on Project-8-apache-lb by creating an Inbound Rule in Security Group.

2. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:
   
   </br>

   
```py
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```

#Restart apache2 service

</br>

`sudo systemctl restart apache2`

`sudo systemctl status apache2`

</br>


![status-apache2](./images-project8/status-apache2.PNG)

---
---
</br>

### **Configure load balancing**


</br>


`sudo vi /etc/apache2/sites-available/000-default.conf`

</br>

```py
#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
```

</br>

![load-balancer](./images-project8/load-balancer.PNG)

</br>

#Restart apache server

`sudo systemctl restart apache2`

`bytraffic balancing` method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by `loadfactor parameter`.

We can also study and try other methods, like: `bybusyness, byrequests, heartbeat`

Try to access the LB’s public IP address or Public DNS name from your browser:

`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

</br>

![lb](./images-project8/lb.PNG)


Note: If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.


`sudo umount /var/log/httpd`

Open two ssh/Putty consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

Try to refresh your browser page http://Load-Balancer-Public-IP-Address-or-Public-DNS-Name/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file

</br>

![log](./images-project8/get-request.PNG)

---
---
</br>

### **Optional Step – Configure Local DNS Names Resolution**

</br>

configure local domain name resolution. The easiest way is to use `/etc/hosts` file

#Open this file on your LB server

    sudo vi /etc/hosts

#Add 2 records into this file with `Local IP address` and arbitrary `name` for both of your Web Servers

`<WebServer1-Private-IP-Address> Web1`
`<WebServer2-Private-IP-Address> Web2`

Now you can update your LB config file with those names instead of IP addresses.

BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1

</br>

![hosts](./images-project8/hosts.PNG)

Now we can update our LB config file with those names instead of IP addresses.

BalancerMember `http://Web1:80` loadfactor=5 timeout=1

BalancerMember `http://Web2:80` loadfactor=5 timeout=1

</br>

![lb-config](./images-project8/lb-config.PNG)

</br>

Curl the Web Servers from LB locally `curl http://Web1` or `curl http://Web2` 

Remember, this is only internal configuration and it is also local to your LB server, these names will neither be ‘resolvable’ from other servers internally nor from the Internet.

Targed Architecture
Now your set up looks like this:

</br>

![architact](./images-project8/architectur.PNG)



