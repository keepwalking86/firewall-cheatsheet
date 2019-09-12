# iptables cheatsheet

## Delete (Flush) existing rules

`iptables -F`

## Set the default chain policies

Khởi tạo default policy (mặc định sẽ block all traffic)

```
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP
```

## Show status of your firewall

Xem nhanh các policy đang được thiết lập

`iptables -L -n -v --line-numbers`

## Block an IP address

Block IP hoặc network được chỉ định

vd: Block IP 1.2.3.4

`iptables -A INPUT -s 1.2.3.4 -j DROP`

vd: Block dải IP từ China (reference [https://lite.ip2location.com/china-ip-address-ranges](https://lite.ip2location.com/china-ip-address-ranges) )

`iptables -A INPUT -s 1.8.0.0/16 -j DROP`

## Block access to remote site

Cái này dùng để chặn truy cập ra ngoài đến site chỉ định

vd: block access to xxx.com site

`
iptables -A OUTPUT -p tcp -d xxx.com -j DROP
`

## Allow HTTP web traffic

Dùng để cho phép truy cập từ bên ngoài vào hệ thống web server và từ trong ra ngoài qua port 80/443

```
iptables -A INPUT -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

## Port forwarding

Dùng để forward (NAT) từ external IP đến private IP

`iptables -t nat -A PREROUTING -p tcp -d 1.2.3.4 --dport 1122 -j DNAT --to 192.168.0.100:22`

## Specifying Multiple Ports with multiport

`iptables -A INPUT -i eth0 -p tcp -m state --state NEW -m multiport --dports http,https,ftp -j ACCEPT`

## Saving Rules

`iptables-save`

## Prevent DoS Attack

Hạn chế truy cập với 25 request/minute đến website

`iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT`


reference:

[https://www.unix-ninja.com/p/An_iptables_cheat-sheet](https://www.unix-ninja.com/p/An_iptables_cheat-sheet)
[https://www.andreafortuna.org/2019/05/08/iptables-a-simple-cheatsheet/](https://www.andreafortuna.org/2019/05/08/iptables-a-simple-cheatsheet/)
