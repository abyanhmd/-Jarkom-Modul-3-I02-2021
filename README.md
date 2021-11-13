# Jarkom-Modul-3-I02-2021
## Group IUP 2
- Gede Yoga Arisudana (05111942000009)
- Abyan Ahmad (05111942000013)
- Zulfiqar Rahman Aji (05111942000019)

## Problem 1
First of all, install all the requirement for each non-client node.

**Enieslobby - DNS Server**
```shell
apt-get install bind9 -y
apt-get install dnsutils -y
```

**Jipangu - DHCP Server**
```shell
apt-get install isc-dhcp-sever -y
```

After installing the `isc-dhcp-server`, don't forget to make the interface of Jipangu in /etc/default/isc-dhcp-server to be `eth0` because `eth0` connects Jipangu to the router above it.

**Water7 - Proxy Server**
```shell
apt-get install squid
```

<br>

## Problem 2
Install isc-dhcp-relay in Foosha.
```shell
apt-get install isc-dhcp-relay -y
```

Make the server as `192.209.2.4` or the IP of Jipangu and the interface as `eth1 eth2 eth3` because Foosha is connected to those three ethernet. After the installation is completed, restart the `isc-dhcp-relay`.
```shell
service isc-dhcp-relay restart
```

In Jipangu, add this script in */etc/dhcp/dhcpd.conf*. This script basically points out the subnet of Jipangu that is connected with its router that is also connected with Foosha, its relay.
```shell
subnet 192.209.2.0 netmask 255.255.255.0 {
    option routers 192.209.2.1;
}
```

<br>

## Problem 3
Set dynamic IP address in /etc/network/interfaces of each client node.
```properties
auto eth0
iface eth0 inet dhcp
```

Add this script in */etc/dhcp/dhcpd.conf*.
```shell
subnet 192.209.1.0 netmask 255.255.255.0 {
    range 192.209.1.20 192.209.1.99;
    range 192.209.1.150 192.209.1.169;
    option routers 192.209.1.1;
    option broadcast-address 192.209.1.255;
    default-lease-time 360;
    max-lease-time 7200;
}
```

After adding the script, restart `isc-dhcp-server` in Jipangu.
```shell
service isc-dhcp-server restart
```

Testing it with one of the client.

![3_Client-IP](/Screenshot/3_Client-IP.png)

<br>

## Problem 4
Add this script in */etc/dhcp/dhcpd.conf*.
```shell
subnet 192.209.3.0 netmask 255.255.255.0 {
    range 192.209.3.30 192.209.3.50;
    option routers 192.209.3.1;
    option broadcast-address 192.209.3.255;
    default-lease-time 720;
    max-lease-time 7200;
}
```

After adding the script, restart `isc-dhcp-server` in Jipangu.
```shell
service isc-dhcp-server restart
```

Testing it with one of the client.

![4_Client-IP](/Screenshot/4_Client-IP.png)

<br>

## Problem 9
In Skypie, add this script in */etc/squid/squid.conf* under the conf that was added before.
```shell
http_port 5000
visible_hostname jualbelikapal.i02.com

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED

http_access allow USERS
```

After adding the script, store the username and password with this command.
```shell
htpasswd -c /etc/squid/passwd [username]
```

The username and password will be stored in */etc/squid/passwd* and the password will be encrypted.
```shell
luffybelikapali02:$apr1$ATTNwxoF$u5ZW6noc1cFuvNgavtB6/1
zorobelikapali02:$apr1$tK/mphqV$JNz2tSuTrB/uGgRDiZ9D2.
```

Restart `isc-dhcp-server` in Jipangu.
```shell
service isc-dhcp-server restart
```
- Luffy

![GIF Luffy](/Screenshot/9_GIF-Luffy.gif)

- Zoro

![GIF Zoro](/Screenshot/9_GIF-Zoro.gif)

<br>

## Problem 10
In order to limit the proxy access time, make a file named **acl.conf** in */etc/squid/acl.conf*.
```shell
acl AVAILABLE_WORKING_1 time MTWH 07:00-11:00
acl AVAILABLE_WORKING_2 time TWHF 17:00-23:59
acl AVAILABLE_WORKING_3 time WHFA 00:00-03:00
```

After making the file, add this script in */etc/squid/squid.conf*.
```shell
include /etc/squid/acl.conf

http_port 5000
visible_hostname jualbelikapal.i02.com

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED

http_access allow USERS AVAILABLE_WORKING_1
http_access allow USERS AVAILABLE_WORKING_2
http_access allow USERS AVAILABLE_WORKING_3
```

![GIF](/Screenshot/10_GIF.gif)

<br>

## Problem 11
In order to make **super.franky.i02.com** domain name, we need to set it up in Enieslobby.
```shell
# /etc/bind/named.conf.local

zone "super.franky.i02.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.i02.com";
};

zone "3.209.192.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/3.209.192.in-addr.arpa";
};
```

```shell
# /etc/bind/kaizoku/super.franky.i02.com

$TTL    604800
@       IN      SOA     super.franky.i02.com. root.super.franky.i02.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.i02.com.
@       IN      A       192.209.3.69    ; IP Skypie
www     IN      CNAME   super.franky.i02.com.
```

```shell
# /etc/bind/kaizoku/3.209.192.in-addr.arpa

$TTL    604800
@       IN      SOA     super.franky.i02.com. root.super.franky.i02.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.209.192.in-addr.arpa. IN      NS      super.franky.i02.com.
69                      IN      PTR     192.209.3.69    ; IP Skypie
```

After setting the domain name in Enieslobby, we also need to set apache2 in Skypie.
```properties
# /etc/apache2/sites-available/super.franky.i02.com.conf

<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName super.franky.i02.com
        ServerAlias www.super.franky.i02.com
        DocumentRoot /var/www/super.franky.i02.com

        <Directory /var/www/super.franky.i02.com>
                Options +Indexes
                AllowOverride All
        </Directory>

        <Directory /var/www/super.franky.i02.com/public>
                Options +Indexes
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

The file in the DocumentRoot is the file from super.franky.zip which has public folder and error folder. After the config in the apache2 is set up, a2ensite the domain name.
```shell
a2ensite super.franky.i02.com
```

In Skypie the resolver configuration should points to Enieslobby's IP, because if it is still Foosha's IP, the website that will be lynx-ed will be the website outside this project. In order to redirect the website from google.com to super.franky.i02.com, we need to add this script.
```shell
acl BLACKLIST dstdomain [-n] .google.com
deny_info http://www.super.franky.i02.com BLACKLIST
http_access deny BLACKLIST
http_reply_access deny BLACKLIST
http_access deny all
```

![GIF](/Screenshot/11_GIF.gif)

<br>

## Problem 12
In order to download jpg and png and limit the bandwidth to 10 kbps, we can add this script.
```shell
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
acl luffy proxy_auth luffybelikapali02
acl download url_regex -i \.jpg$ \.png$

delay_pools 1

delay_class 1 1
delay_parameters 1 1250/1250
delay_access 1 allow luffy
delay_access 1 deny zoro
delay_access 1 allow download
delay_access 1 deny all
```

The first line is the location of the password file for proxy authentication. The second and third line is to declare luffy username and jpg/png download. The parameter is 1250/1250 because the limit is 10kbps = 10000bps = 1250 bit/s. We need to allow luffy and deny zoro. We also need to allow download jpg/png and after all the requirement is fulfilled, deny all the remaining possibility to avoid any problem.

![GIF](/Screenshot/12_GIF.gif)

<br>

## Problem 13
This script is the addition to previous problem's script.
```shell
delay_class 2 1
delay_parameters 2 -1/-1
delay_access 2 allow zoro
delay_access 2 deny luffy
delay_access 2 deny all
```
In order to make the bandwidth is unlimited, we can make the parameters to -1/-1. We don't include the picture or gif for this problem because it didn't state the rate speed of downloading, it was just saved immediately.
