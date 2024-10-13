# LAB: SPAN/RSPAN/ERSPAN
Where to use each:
- SPAN: used when packet analyzer is on the same switch as the traffic to be captured
- RSPAN: used when packet analyzer is on another switch as the traffic to be captured
- ERSPAN: used when packet analyzer is on a remote location as the traffic to be caputred

## SPAN
SW2 ports 2 and 3 are on vlan 50. Declare the source for the monitor session and destination. 
```cisco
config#)monitor session 1 source interface g0/3
config#)monitor session 1 destination interface g0/2
config#)exit
>show monitor session 1
```
Observe traffic being captured by right clicking the link on g0/2 and clicking packet capture. Traffic from G0/3 is now being sent to G0/2. Switches by default would not allow this. SPAN allows for the traffic to be mirrored to another port.

## RSPAN
SW1 port g0/1 and g0/3 are on vlan 1. Configure a remote-span vlan then configure monitor sessions to capture the traffic from g0/1, g0/3. Note: Devices should not be configured on the remote-span vlan. MAC-addresses are not learned. Ensure no loops are on the remote-span vlan. Traffic is flooded out the remote-span vlan. Monitor session number is locally significant but best to keep the same to avoid confusion

SW2 port g0/1 is the destination for the packet analyzer. Configure remote-span vlan then configure monitor sessions to g0/1. Ensure traffic is trunked for the remote-span vlan between switches.
```cisco
SW1)config# vlan 99
SW1)config-vlan# name RSPAN_VLAN
SW1)config-vlan# remote-span
SW1)config# monitor session 1 source interface g1, g3
SW1)config# monitor session 1 destination remote vlan 99

SW2) config# vlan 99
SW2) config-vlan# name RSPAN_VLAN
SW2) config-vlan# remote-span
SW2) config# monitor session 1 source remote vlan 99
SW2) config# monitor session 1 destination interface g0/1

show monitor session 1
show monitor remote
```
Observe traffic being captured by right clicking the link on g0/1 and clicking packet capture. Traffic from SW1 g1,g3 is now being sent to SW2 G0/1.

## ERSPAN
IOS-VL image does not have capability for ERSPAN. On R1 configure source monitor session for erspan. On R2 configure destination monitor session for erspan. Capture traffic received from G3 on R1 and send it to PC-F.

```cisco
! Define ACL for filtering
R1) config# ip access-list extended PC_E_TRAFFIC
R1) config-acl# 10 permit ip host 10.100.0.11 any
R1) config-acl# exit

R1)config# monitor session 1 type erspan-source
R1)config-monitor# description SOURCE-PC-E-TRAFFIC
R1)config-monitor# source interface g1/0/4 rx
R1)config-monitor# filter access-group PC_E_TRAFFIC
R1)config-monitor# no shutdown
R1)config-monitor# destination
R1)config-monitor-destination# erspan-id 2
R1)config-monitor# destination# ip address 172.16.0.11
R1)config-monitor# destination# origin ip address 10.100.0.254
R1)config-monitor# end

R2)config# monitor session 2 type erspan-destination
R2)config-monitor#  destination interface G2
R2)config-monitor#  source
R2)config-monitor-source# erspan-id 2
R2)config-monitor-source# ip address 10.100.0.11
```
Observe traffic being captured by right clicking the link on SW4 g0/1 and clicking packet capture. Traffic received on R1 G3 is now being sent to PC-F.