**Dokumentácia k projektu VLAN a DHCP konfigurácie**
## DHCP Pools


```other
Router# configure terminal

Router(config)# ip dhcp pool Voice
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.1
Router(dhcp-config)# option 150 ip 192.168.10.1
Router(dhcp-config)# exit

Router(config)# ip dhcp pool Data
Router(dhcp-config)# network 192.168.20.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.20.1
Router(dhcp-config)# exit

Router(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.2
Router(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.2

Router(config)# interface gigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

Router(config)# interface gigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
```


```other
Router#show ip dhcp pool
  
Pool Voice :
Utilization mark (high/low) : 100 / 0
Subnet size (first/next) : 0 / 0
Total addresses : 254
Leased addresses : 0
Excluded addresses : 2
Pending event : none
  
1 subnet is currently in the pool
Current index IP address range Leased/Excluded/Total
192.168.10.1 192.168.10.1 - 192.168.10.254 0 / 2 / 254
  
Pool Data :
Utilization mark (high/low) : 100 / 0
Subnet size (first/next) : 0 / 0
Total addresses : 254
Leased addresses : 0
Excluded addresses : 2
Pending event : none
  
1 subnet is currently in the pool
Current index IP address range Leased/Excluded/Total
192.168.20.1 192.168.20.1 - 192.168.20.254 0 / 2 / 254
```


## VLAN


| Number | Name  | Usage     | Pool                          |
| ------ | ----- | --------- | ----------------------------- |
| 10     | Voice | IP Phones | 192.168.10.1 - 192.168.10.254 |
| 20     | Data  | Computers | 192.168.20.1 - 192.168.20.254 |

## Setup Office Switch


```other
Switch>enable
Switch#
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#
Switch(config)#vlan 10
Switch(config-vlan)# name Voice
Switch(config-vlan)#vlan 20
Switch(config-vlan)# name Data
Switch(config-vlan)#end
Switch(config)#interface FastEthernet0/3
Switch(config-if)#switchport mode trunk

Switch(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
switchport trunk allowed vlan 10,20
Switch(config-if)#end

```


## Overenie, ze vsetko je ok


```other
Switch#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4, Fa0/5, Fa0/6, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
10   Voice                            active    Fa0/2
20   Data                             active    Fa0/1
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
Switch#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/3       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/3       10,20

Port        Vlans allowed and active in management domain
Fa0/3       10,20

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/3       10,20

Switch#
```


## Setup Pocitacov

- OS Nabootovany
- IP Config
- DHCP
- niekedy po zmene na DHCP failne request pre IP. pockat 5s opakovat znova

## Narazam na error


```other
%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
%SPANTREE-2-RECV_PVID_ERR: Received BPDU with inconsistent peer vlan id 10 on FastEthernet0/2 VLAN1.

%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking FastEthernet0/2 on VLAN0001. Inconsistent local vlan.
```


`643A00` 

# telephony-service


```other
Router(config)# telephony-service
Router(config-telephony)# max-dn 10
Router(config-telephony)# max-ephones 10
Router(config-telephony)# ip source-address 192.168.10.1 port 2000
Router(config-telephony)# auto assign 1 to 10
Router(config-telephony)# create cnf-files
Router(config-telephony)# exit

Router(config)# ephone-dn 1
Router(config-ephone-dn)# number 1001
Router(config-ephone-dn)# exit

...
```

