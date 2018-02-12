# Getting Started with Cisco Zone Based Firewall.
## Zone の定義

    Firewallは Juniper SSGを最初に設定したせいか、Trust/Untrustがとてもなじみ深い

```
zone security Trust
zone security Untrust
```

## zone-pair 作成

    zone-pair は定義したZoneから Source Zone, Destination Zoneをペアとして定義する
    後ほど、zone-pairの方向性で許可・拒否する Policy-mapを適用する

```
zone-pair security Trust_to_Untrust source Trust destination Untrust
zone-pair security Untrust_to_Trust source Untrust destination Trust
```

## Class-mapを作成して、許可するプロトコルを設定する

    policy-map に適用する Protocolを定義する。
    ここでは Trustは全てのプロトコルを許可して、Untrustは TCP/UDPのみ許可する

```
class-map type inspect match-any ClassMap_Trust
 match protocol tcp
 match protocol udp
 match protocol icmp

class-map type inspect match-any ClassMap_Untrust
 match protocol tcp
 match protocol udp
```

## zone-pair に設定する Policy-mapを定義する

    先ほど作成した Clapp-map は type inspectとして適用する
    Trust -> Untrust は tcp/udp/icmpが許可され、 Untrust -> Trust は tcp/udpが許可される。
    class-default に drop log を適用すると、ログが出力される 

```
policy-map type inspect PolicyMap_Trust_to_Untrust
 class type inspect ClassMap_Trust
  inspect
 class class-default
  drop log

policy-map type inspect PolicyMap_Untrust_to_Trust
 class type inspect ClassMap_Untrust
  inspect
 class class-default
  drop log
```

## zone-pair に Policy-mapを適用する

    作成済みの zone-pairに policy-map を適用する

```
zone-pair security Trust_to_Untrust source Trust destination Untrust
 service-policy type inspect PolicyMap_Trust_to_Untrust

zone-pair security Untrust_to_Trust source Untrust destination Trust
 service-policy type inspect PolicyMap_Untrust_to_Trust

```

## interface に zone を割り当てる


```
interface GigabitEthernet2
 zone-member security Untrust

interface GigabitEthernet4
 zone-member security Trust

```