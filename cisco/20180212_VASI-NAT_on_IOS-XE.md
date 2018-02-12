# VASIでNATを設定する

    scenario 2を使用

    https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/200255-Configure-VRF-Aware-Software-Infrastruct.html#anc12

## VRFを定義

LAN Segment を VRF Trust_leftに設定して、Untrust側は Untrust_rightとして定義

```
vrf definition Trust_left
 address-family ipv4
 exit-address-family

vrf definition Untrust_right
 address-family ipv4
 exit-address-family

```

## interface をVRF割り当てる

```
interface GigabitEthernet 4
 vrf forwarding Trust_left
 ip add 192.168.10.254 255.255.255.0
 ip nat inside

interface GigabitEthernet2
 vrf forwarding Untrust_right
 ip address 10.0.10.1 255.255.255.252
```

## vasi interface の設定

```
interface vasiright 1
 vrf forwarding Untrust_right
 ip add 10.10.0.2 255.255.255.252

interface vasileft 1
 vrf forwarding Trust_left
 ip address 10.10.0.1 255.255.255.252
 ip nat outside
```

## NAT変換用Loopback の設定

```
interface loopback 1
 vrf forwarding Trust_left
 ip address 200.0.0.1 255.255.255.255
```

## NAT 設定

```
ip access-list standard NAT_Trust
 permit 192.168.10.0 0.0.0.255

ip nat inside source list NAT_Trust interface loopback 1 vrf Trust_left overload
```

## Routing 設定

    今回は VASI-Leftを NAT Outside Inteface として設定しているので
    VRF Trust_left では default route を vasileft 1 を出力インターフェイスとして next-hopを vasi-right 1に設定
    VRF Untrust_right では default route は Router-2 の connected
    VRF Trust_left で NAT変換されたアドレス宛てに vasiright 1 を出力インターフェイスとして next-hopに vasi-left 1を設定

```
ip route vrf Trust_left 0.0.0.0 0.0.0.0 vasileft1 10.10.0.2
ip route vrf Untrust_right 0.0.0.0 0.0.0.0 GigabitEthernet2 10.0.10.2
ip route vrf Untrust_right 200.0.0.1 255.255.255.255 vasiright1 10.10.0.1
```

## 実行結果

```

router1# sh ip nat translations
Pro  Inside global         Inside local          Outside local         Outside global
icmp 200.0.0.1:3           192.168.10.1:61840    192.168.20.254:61840  192.168.20.254:3
icmp 200.0.0.1:4           192.168.10.1:62096    192.168.20.254:62096  192.168.20.254:4
icmp 200.0.0.1:1           192.168.10.1:61328    192.168.20.254:61328  192.168.20.254:1
icmp 200.0.0.1:5           192.168.10.1:62352    192.168.20.254:62352  192.168.20.254:5
icmp 200.0.0.1:2           192.168.10.1:61584    192.168.20.254:61584  192.168.20.254:2
Total number of translations: 5

```