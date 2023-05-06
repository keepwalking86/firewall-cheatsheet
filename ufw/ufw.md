**Open ports/service**

- enable/disable port 8080

```
sudo ufw allow 8080 
```

- Open ports for specified IPs

```
sudo ufw allow from 192.168.1.10 to any port 22
sudo ufw allow from 172.16.2.0/24 to any port 21
```

- check port/services allow**

```
sudo ufw ...
```

**Remove port/service**

```
sudo ufw ...
```

**Remove rich rule**

`sudo ufw ..`


**whitelist source ip address**

```
sudo ufw allow from 192.168.1.10
sudo ufw allow from 172.16.1.0/24
```

**Deny from specified IPs**
```
sudo ufw deny from 192.168.1.10
sudo ufw deny from 172.16.1.0/24
```

**List rules**

`sudo ufw status numbered`
