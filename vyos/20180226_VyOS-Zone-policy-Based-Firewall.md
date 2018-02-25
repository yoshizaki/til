# VyOS Zone-policy Based Firewall


router-1に Zone-policy based firewall設定

```
[Internet] -- [HostOS] -- [172.16.1/0/24] -- [router-1] -- [10.0.0.0/29] --[router-2] --- [192.168.x.0/24]
```

## system 設定

```
set system host-name router-1
set service ssh
```
## Interface 設定

eth0 : HostOS経由Physical Router DHCP払い出し
eth1 : router-2接続、router-2にPCを接続

```
set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth1 address 10.0.0.1/29
```

## routing 設定

192.168.x.0/24 は router-2 の下にいるサブネット

```
set protocols static route 192.168.1.0/24 next-hop 10.0.0.2
set protocols static route 192.168.2.0/24 next-hop 10.0.0.2
```

## DNS 設定

```
set system name-server 172.16.1.1
```

### DNS Forwarder 設定

```
set service dns forwarding cache-size 0
set service dns forwarding listen-on eth1
set service dns forwarding name-server 172.16.1.1
```

## NAT 設定

数字の小さいルール順に処理される(source addressにACLを使いたくなる)

10.0.0.0/29 : router-2 connected
192.168.1.0/24 : router-2 Subnet-1
192.168.2.0/24 : router-2 Subnet-2

```
set nat source rule 100 outbound-interface eth0
set nat source rule 100 source address 10.0.0.0/29
set nat source rule 100 translation address masquerade

set nat source rule 110 source address 192.168.1.0/24
set nat source rule 110 outbound-interface eth0
set nat source rule 110 translation address masquerade

set nat source rule 120 source address 192.168.2.0/24
set nat source rule 120 outbound-interface eth0
set nat source rule 120 translation address masquerade
```

## Zone Policy Firewall

ゾーンの定義
router-2 に接続している eth1は Trust Zone
HostOS に接続している eth1 は Untrust Zone

```
set zone-policy zone Untrust interface eth0
set zone-policy zone Trust interface eth1
```

UntrustからTrustへの通信は Established/Related(all protocol)を許可

```
set firewall name Untrust_to_Trust rule 10 action accept
set firewall name Untrust_to_Trust rule 10 state established enable
set firewall name Untrust_to_Trust rule 10 state related enable
set firewall name Untrust_to_Trust rule 10 protocol all
set zone-policy zone Trust from Untrust firewall name Untrust_to_Trust
```

TrustからUntrustへの通信は全て許可

```
set firewall name Trust_to_Untrust rule 10 action accept
set zone-policy zone Untrust from Trust firewall name Trust_to_Untrust
```

## commit/save

commitして反映、saveで保存

```
commit
save
```