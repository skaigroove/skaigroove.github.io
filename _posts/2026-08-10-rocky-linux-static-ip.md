---
layout: post
title: Rocky Linux 고정 IP 설정
subtitle: Rocky 8 / Rocky 9 버전별 네트워크 설정 방법
tags: [linux, network, rocky]
---

## 1. 현재 OS 정보 확인

```bash
cat /etc/rocky-release
```

## 2. 네트워크 인터페이스 확인

```bash
ifconfig
```

랜선이 물리적으로 꽂혀 있는 인터페이스 확인 (UP 상태인 것이 연결된 인터페이스):

```bash
ip -br link
```

출력 예시:
```
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
ens192           UP             00:0c:29:xx:xx:xx <BROADCAST,MULTICAST,UP,LOWER_UP>
ens224           DOWN           00:0c:29:xx:xx:xx <BROADCAST,MULTICAST>
```

## 3. 버전별 설정 방법

OS 버전마다 설정 파일의 위치와 형식이 다르다.

---

### Rocky 8

**설정 파일 위치**: `/etc/sysconfig/network-scripts/ifcfg-<인터페이스명>`

#### 파일 편집

```bash
vi /etc/sysconfig/network-scripts/ifcfg-<인터페이스명>
```

```bash
TYPE=Ethernet
PROXY_METHOD=none
BOOTPROTO=none
BROWSER_ONLY=no
DEFROUTE=yes                 # 다중 인터페이스 시 디폴트 게이트웨이 설정
NAME=ensxxx
DEVICE=ensxxx
ONBOOT=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
IPADDR=192.168.0.xxx
PREFIX=24
GATEWAY=192.168.0.1
DNS1=8.8.8.8
UUID=생성되어있는값
```

#### 적용

```bash
systemctl restart NetworkManager
```

---

### Rocky 9

**설정 파일 위치**: `/etc/NetworkManager/system-connections/<인터페이스명>.nmconnection`

#### 파일 편집

```bash
vi /etc/NetworkManager/system-connections/ens224.nmconnection
```

```ini
[connection]           # 연결 메타정보
id=ens224              # 연결 이름 (nmcli에서 보이는 이름)
type=ethernet          # 연결 타입
interface-name=ens224  # 바인딩할 실제 인터페이스

[ethernet]             # L2 (물리 계층) 설정
# MAC 고정 등 필요시 추가

[ipv4]                 # L3 (IP 계층) 설정
method=manual          # manual=고정, auto=DHCP
addresses=192.168.0.251/24
gateway=192.168.0.1
dns=8.8.8.8;

[ipv6]                 # IPv6 설정
method=disabled        # 사용 안 하면 disabled
```

#### 적용

```bash
nmcli connection reload
nmcli connection up ens224
```
