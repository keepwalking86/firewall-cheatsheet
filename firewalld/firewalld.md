**Open ports/service**

- enable/disable port 8080

```
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent
```

- Open ports for specified IPs

```
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.10.0/24" port port="27017" protocol="tcp" accept'
sudo firewall-cmd --reload
```

- check port/services allow**

```
firewall-cmd --list-ports
firewall-cmd --list-services
```

**Remove port/service**

```
firewall-cmd --zone=public --remove-port=8080/tcp
firewall-cmd --permanent --remove-port=8080/tcp
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --reload
```

**Remove rich rule**

`firewall-cmd --permanent --remove-rich-rule 'rule family="ipv4" source address="1.1.1.1/32" port protocol="tcp" port="27017" accept`

**manage zones**

- Show zones
```
firewall-cmd --get-zones
firewall-cmd --get-active-zones
firewall-cmd --get-default-zone
```
- Change specify interface to a zone

```
firewall-cmd --zone=work --change-interface=eno3
firewall-cmd --get-active-zones
```
work
  interfaces: eno3
public
  interfaces: eno2

**whitelist source ip address**

```
firewall-cmd --permanent --zone=public --add-source=192.168.1.0/24
firewall-cmd --permanent --zone=public --add-source=192.168.1.111/32
