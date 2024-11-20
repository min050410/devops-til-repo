## wireguard 연결 문제 해결

### 환경

-   Windows + WireGuard client (Stable)
-   Synology DSM 7.2.2 + WireGuard (VPN Server)

### 문제 상황

WireGuard 클라이언트에서 VPN 서버와의 Handshake는 성공했으나, 다음과 같은 문제가 발생함.

1. 클라이언트가 이더넷 네트워크에 연결되지 않음.
2. 클라이언트가 VPN을 통해 Synology 서버의 로컬 네트워크에 접근할 수 없음. (Ping 테스트 실패.)

### 해결 과정

#### 1. NAT (Masquerading) 설정

WireGuard 클라이언트가 Synology 서버의 로컬 네트워크에 접근하려면, Synology가 들어오는 VPN 트래픽을 적절히 라우팅해야 합니다.  
이를 위해 NAT(Masquerading) 설정을 추가했습니다.

#### NAT의 역할

NAT를 적용하면 WireGuard 클라이언트의 IP(예: 10.0.0.x)가 Synology 로컬 네트워크의 IP(예: 172.30.1.x)로 변환됩니다.  
이로 인해 Synology의 로컬 네트워크는 VPN 트래픽을 내부 요청처럼 처리하게 됩니다.

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

여기서 eth0은 Synology 서버의 LAN 인터페이스 이름입니다. 네트워크 인터페이스 이름은 ip addr 명령으로 확인 가능합니다.

#### 2. 클라이언트 설정 수정

클라이언트의 WireGuard 설정에서 AllowedIPs에 Synology의 로컬 네트워크 대역(172.30.1.0/24)이 포함되어야 합니다.

클라이언트의 설정 파일을 다음 처럼 수정하였습니다.

기존 설정:

```
AllowedIPs = 10.0.0.0/24
```

수정된 설정:

```
AllowedIPs = 10.0.0.0/24, 172.30.1.0/24
```

-   10.0.0.0/24: VPN 대역 (WireGuard 인터페이스)
-   172.30.1.0/24: Synology의 로컬 네트워크 대역
