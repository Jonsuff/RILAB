## Raspberry Pi4에 Ubuntu Mate 설치하기

## 1. Ubuntu Image 마운트 및 기본 계정설정

### 1.1 Ubuntu Servar img 다운 및 마운트

 https://ubuntu.com/download/raspberry-pi 에 접속합니다.

 Raspberry Pi용 LTS 버전 선택하기 (Raspberry Pi4 Ubuntu18.04.4 LTS Download 64-bit)

\- 자신의 Raspberry pi에 맞는 OS를 선택합니다.

받은 image를 Balena etcher: https://www.balena.io/etcher/ 를 사용하여 SD카드에 기록합니다.

### 1.2 기본계정 설정하기

처음 실행하면 기본 계정은(ID/PASSWORD : ubuntu/ubuntu)이다. 패스워드를 치라 나오는데 이때 패스워드는 ubuntu를 입력하면 아래와 같은 문구가 나오고 

```
You are required to change your password immediately(root enforced)
Changing password for ubuntu.
```

그후 current(현재)비밀번호를 입력 후 새로운 비밀 번호 설정을 하면됩니다.



## 2. 네트워크 설정 (Wi-Fi 설정)

### 2.1 netplan 설정파일 확인하기

root로 사용자 전환합니다.

```
ubuntu@ubuntu:~$ sudo su
```

/etc/netplan으로 이동합니다.

```
root@ubuntu:/home/ubuntu# cd /etc/netplan/
```

설정파일 확인합니다.

```
root@ubuntu:/etc/netplan# ls -l
totla 4
-rw-r--r-- 1 root root 416 Aug 6 22:52 50-cloud-init.yaml
```

### 2.2 확인된 yaml파일 편집하기

확인된  yaml파일을 vim 을 사용하여 편집을 시작합니다.

```
root@ubuntu:/etc/netplan# vim 50-cloud-init.yaml
```

시작하게 되면 아래와 같은 문구가 보일것입니다.

```
...
network:
   ethernets:
      eth0:
         dhcp4: true
         optinal: true
   version: 2
...
```

아래와 같이 수정한다.  이때 wifiname과 password를 본인의 wifi의 이름과 패스워드로 입력합니다.

```
...
network:
   ethernets:
      eth0:
         dhcp4: true
         optinal: true
   wifis:
      wlan0:
         dhcp4: true
         optinal: true
         access-points:
            "wifiname"
               password: "password"
               
   version: 2
...
```

netplan 설정을 적용합니다.

```
root@ubuntu:/etc/netplan# netplan generate
root@ubuntu:/etc/netplan# netplan apply
```

wifi 설정이 잘 됬는지 확인합니다.

```
root@ubuntu:/etc/netplan# ifconfig

...
wlan0: flags=4099<UP,BROADCAST,MULTICAST> mtu 1500
       inet 192,168.0.44
       ...
```

이와 같이 wlan0에 ip가 잘 나오는지 확인합니다.

### 2.3 시간대 변경하기

설치한 ubuntu의 기본 시간대는 UTC+0입니다.

우리나라 시간대는 Korea Standard Time(KST)로 UTC+9 입니다.

시간대를 알맞게 설정하지 않은 경우, 부팅로그나 커널 로그 및 기타 로그를 확인할때 번거로움이 생길 수 있습니다.  

아래 명령어를 실행한 후

```
ubuntu@ubuntu:~$ sudo dpkg-reconfigure tzdata
```

 Asia / Seoul 을 선택하면

```
Current default time zone: 'Asia/Seoul'
Local time is now: Thu Aug 27 17:02:27 KST 2020.
Universal Time is now: Thu Aug 27 08:02:27 UTC 2020.
```

이와 같은 문구를 확인할 수 있습니다.



## 3. Ubuntu Mate 설치

### 3.1 apt update & apt upgrade

먼저 apt update 와 upgrade를 진행합니다.

```
ubuntu@ubuntu:~$ sudo apt update
ubuntu@ubuntu:~$ sudo apt upgrade
```



### 3.2 Install Ubuntu-mate-desktop

이제 Mate를 설치하기 위해 다음 명령어를 실행합니다.

```
ubuntu@ubuntu:~$ sudo apt install ubuntu-mate-desktop
```

완료가 되면 재부팅합니다.



## 4. Ubuntu 환경에서 WI-FI 연결이 되지 않는 경우

먼저 아래 명령어를 실행한다.

```
ubuntu@ubuntu:~$ sudo apt install network-manager
```

2.1과 2.2를 다시 하여 아래처럼 renderer: NetworkManager를 추가한다.

```

...
      wlan0:
         renderer: NetworkManager
         dhcp4: true

...
```

이후 아래 명령어를 실행하면 Wi-Fi가 연결이 된다.

```
root@ubuntu:/etc/netplan# netplan generate
root@ubuntu:/etc/netplan# netplan apply 
```







































