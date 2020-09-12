=================================  IPTABLES FIREWALL  =================================

# Table of Contents
- [I. About iptables](#about)
  - [1. Types of tables](#types-of-tables)
  - [2. Types of chains](#types-of-chains)
  - [3. Actions of rules ](#actions-of-rules)
- [II. Cheatsheet IPv4](#cheatsheet-IPv4)
  - [1. Default rules](#default-rules)
  - [2. Allow incoming connections](#allow-incoming)
  - [3. Drop/Reject](#drop-reject)
  - [4. Prevent DoS Attack](#prevent-dos)
  - [5. Delete Firewall Rules](#delete-rule)
  - [6. Define chain](#define-chain)
  - [7. Displaying firewall status](#display-status)
  - [8. Saving/Backup/Restore](#backup-restore)
  - [9. NAT/Routing](#nat-route)
- [III. Cheatsheet IPv6](#cheatsheet-IPv6)
  - [1. Default rules](#default-rules-ipv6)

==========================================================

# <a name="about">I. About iptables</a>

iptables là firewall mà sử dụng để quản lý các rules cho việc lọc (filter) hoặc NAT các gói tin (packages)

Cấu trúc của iptables là iptables -> tables --> chains --> rules. Nghĩa là có thể chứa nhiều tables. Tables có thể chứa nhiều chains. Mà chains có thể là có sẵn hoặc định nghĩa. Chains có thể chứa nhiều rules

## <a name="types-of-tables">1. Types of tables</a>

iptables gồm các tables sau:

**filter** : Đây là table mặc định. Nó chứa các chain có sẵn INPUT, OUPUT, FORWARD

- INPUT: chain này dùng cho các rule incoming vào firewall

- OUTPUT: chain này dùng cho các rule từ firewall outgoing

- FORWARD: chain này dùng route các gói đi qua firewall

**nat**: table này gồm chains có sẵn sau PREROUTING, POSTROUTING, OUTPUT

- PREROUTING: chain này thay đổi gói tin trước khi routing. Chain này được dùng cho DNAT (Destination NAT).

- POSTROUTING: chain này thay đổi gói tin sau khi routing. Chain này được dùng cho SNAT (Source NAT) 

- OUPUT: NAT các gói trên chính firewall

**mangle**: table này được sử dụng thay đổi các gói đặc biệt. Nó gồm các chains có sẵn sau INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING

**raw**: table này dùng cho trường hợp cấu hình thô. table này gồm 2 chains có sẵn PREROUTING và OUTPUT

Ngoài ra còn có table **security**. table này dùng cho các rules network cho MAC (Mandatory Access Control)

## <a name="types-of-chains">2. Types of chains</a>

iptables sử dụng 03 chains có sẵn: INPUT, OUTPUT và FORWARD

- INPUT: dùng cho kiểm soát các hành động kết nối incomming. Ví dụ kết nối từ bên ngoài vào web ports

- OUTPUT: dùng cho kiểm soát các hành động outgoing. Ví dụ kết nối từ host ra ngoài internet

- FORWARD: Thay vì kết nối trực tiếp từ bên ngoài vào hệ thống, sử dụng chain FORWARD cho kiểm soát hành động trước khi chuyển tiếp vào hệ thống. Nó đóng vai trò như router để NAT vào hệ thống. Ví dụ forward port 80 từ host firewall vào host web server.

Ngoài ra, có thể định nghĩa chain mới theo nhu cầu sử dụng.

## <a name="actions-of-rules"> 3. Actions of rules</a>

Với bất kỳ rule nào được thiết lập đều phải đưa ra hành động để thực hiện. iptables gồm có 03 actions sau:

- ACCEPT: Hành động cho phép thực hiện kết nối

- REJECT: Không cho phép kết nối, và gửi phản hồi đến client. Thông báo client nhận được giống như "Destination port unreachable"

- DROP: Không cho phép kết nối, và không gửi thông tin đến client. Thông báo đến client là "Request time out".

# <a name="cheatsheet-IPv4"> II. Cheatsheet IPv4</a>

## <a name="default-rules">1. Default rules</a>

Sau khi cài đặt xong gói iptables-services, thì nội dung cấu hình mặc định của iptables như sau:

```
# sample configuration for iptables service
# you can edit this manually or use system-config-firewall
# please do not ask us to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

Rules ở cấu hình trên cho phép ping và truy cập ssh từ bên ngoài vào hệ thống và từ hệ thống có thể truy cập ra bên ngoài, còn lại thì reject.

## <a name="allow-incoming">2. Allow incoming connections</a>

- Allow any traffic from specify IP/Network addresses

Ví dụ: Cho phép Network 192.168.1.0/24 được phép truy cập all traffic tcp qua port eth1. Hay cho phép một public IP được phép truy cập vào server.

```
iptables -A INPUT -i eth1 -s 192.168.1.0/24 -p tcp -j ACCEPT
iptables -A INPUT -s 27.72.72.27/32 -j ACCEPT
```

- Allow all to access port/service

ex: Allow HTTP/HTTPS web traffic

```
iptables -A INPUT -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
```
or

```
iptables -A INPUT -p tcp --dport http -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport https -m state --state NEW,ESTABLISHED -j ACCEPT
```

or using match `-m multiport`

`iptables -A INPUT -p tcp -m multiport --dports 80,443`

- Insert one or more rules in the selected chain as the given rule number

Using option -I (insert)

example insert rule with line 5 in chain INPUT

`iptables -I INPUT 5 -p tcp -m tcp --dport 443 -j ACCEPT`

## <a name="drop-reject">3. Drop/Reject</a>

- Drop ping (icmp)

`iptables -A INPUT -p icmp --icmp-type echo-request -j DROP`

- Block an IP address

Blocking an IP, multi IP, IP range or CIDR

```
iptables -A INPUT -s 1.2.3.4 -j DROP
iptables -A INPUT -s 1.2.1.2,1.2.2.3 -j DROP
iptables -A INPUT -m iprange --src-range 1.2.3.0-1.2.3.255 -j DROP
iptables -A INPUT -s 111.111.111.0/24 -j DROP
```

## <a name="prevent-dos">4. Prevent DoS Attack</a>

```
iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 50 -j REJECT --reject-with tcp-reset
iptables -A INPUT -p tcp --dport 443 -m connlimit --connlimit-above 50 -j REJECT --reject-with tcp-reset
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

Limit is 50 concurrent connections. If > 50 connections has been reject

## <a name="delete-rule">5. Delete Firewall Rules</a>

- Delete rule by chain and number line

`iptables -D INPUT 5`

mean delete rule number line 5 in chain `INPUT`

- Delete rule by specification

```
iptables -D INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -D INPUT -s 1.2.3.4/32 -j DROP
```

## <a name="define-chain">6. User defined chain</a>

Using `-N` to create new chain

example: Define chain to allow  source addresses

```
iptables -N crm-ips
iptables -A INPUT -j crm-ips
iptables -A crm-ips -s 113.190.113.190 -j ACCEPT
iptables -A crm-ips -s 123.123.123.123 -j ACCEPT
iptables -A crm-ips -j DROP
```

## <a name="display-status">7. Displaying firewall status</a>

`iptables -L -n -v`

- Display with lines

`iptables -L -n -v --line-numbers`

## <a name="backup-restore">8. Saving/Backup/Restore</a>

- Saving rules

`iptables-save >/etc/sysconfig/iptables`

- Backup rules

`iptables-save >/path/to/backup/iptables-yymmdd`

- Retore rules

```
iptables-restore </path/to/backup/iptables-yymmdd
iptables-save >/etc/sysconfig/iptables
systemctl restart iptables
```

## <a name="nat-route">9. NAT/Routing</a>

- DNAT

`iptables -t nat -A PREROUTING -p tcp -d 1.2.3.4 --dport 1122 -j DNAT --to 192.168.0.100:22`

- Postrouting and Masquerade

Allow private IP addresses in LAN (internal) to communicate with external public networks

`iptables -t nat -A POSTROUTING -o eth* -j MASQUERADE`

DNAT and SNAT/MASQUERADE request to enable IP forwarding

`sysctl -w net.ipv4.ip_forward=1`

# <a name="cheatsheet-ipv6">III. Cheatsheet for IPv6</a>

# <a name="default-rules-ipv6">1. Default rules</a>

```
# sample configuration for ip6tables service
# you can edit this manually or use system-config-firewall
# please do not ask us to add additional ports/services to this default configuration
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p ipv6-icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --tcp-flags SYN SYN -m tcpmss --mss 1:500 -j DROP
-A INPUT -j REJECT --reject-with icmp6-adm-prohibited
-A FORWARD -j REJECT --reject-with icmp6-adm-prohibited
COMMIT
```