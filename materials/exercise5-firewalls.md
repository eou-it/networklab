# Lab Exercise 5 - Juniper Firewalls

## Outcomes

When completed with this lab, you will have accomplished the following basic routing tasks:
* Configure your Firewall with a unique public static IP address and connect it to the Internet
  * interface ge-0/0/0.0
* Verify the configuration
  * Observe routing information on the firewall
  * Observe ARP information of devices connected to the firewall
  * PING and TRACEROUTE from each firewall to any/all of the addresses from the table below

## Configuration

| Firewall | Subnet | Gateway Address | Firewall Address |
| === | === | === | === |
| lab-fw-1 | 192.168.7.152/30 | 192.168.7.153 | 192.168.7.154 |
| lab-fw-2 | 172.30.44.144/30 | 172.30.44.145 | 172.30.44.146 |
| lab-fw-3 | 172.16.254.20/30 | 172.16.254.21 | 172.16.254.22 |
| lab-fw-4 | 192.168.72.72/30 | 192.168.72.73 | 192.168.72.74 |