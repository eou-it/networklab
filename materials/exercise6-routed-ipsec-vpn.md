# Lab Exercise 6 - (Static) Routed IPSEC VPN

## Outcomes

When completed with this lab, you will have accomplished the following basic routing tasks:
* Configure VPN tunnels from your firewall device to at least one other firewall on the lab network
* Configure static route through the VPN tunnel or tunnels for each firewall's networks
* Verify the configuration
  * Observe routing information on the firewall
  * Observe ARP information of devices connected to the firewall
  * PING and TRACEROUTE from each firewall to any/all of the private addresses

## Configuration

### Firewall Addresses
| Firewall | VPN Endpoint IP |
| --- | ---|
| lab-fw-1 | 192.168.7.154 |
| lab-fw-2 | 172.30.44.146 |
| lab-fw-3 | 172.16.254.22 |
| lab-fw-4 | 192.168.72.74 |


### Install IKE an IPSEC Proposals and Policies

From configuration mode, issue the `load merge terminal` command, then paste the following block of configuation. hit `[ENTER]` and then press `[CTRL] + [D]`.

```
security {
    ike {
        proposal lab-ike-proposal {
            authentication-method pre-shared-keys;
            dh-group group20;
            encryption-algorithm aes-256-gcm;
            lifetime-seconds 28800;
        }
        policy lab-ike-policy {
            proposals lab-ike-proposal;
            pre-shared-key ascii-text "labvpnpresharedkey";
        }
    }
    ipsec {
        proposal lab-ipsec-proposal {
            protocol esp;
            encryption-algorithm aes-256-gcm;
            lifetime-seconds 3600;
        }
        policy lab-ipsec-policy {
            perfect-forward-secrecy {
                keys group20;
            }
            proposals lab-ipsec-proposal;
        }
    }
}
```

# Install IKE Gateway and IPSEC VPN Configuration Blocks, and Enable the IKE service on the untrust zone

Note: These can be installed on each Firewall. Notice that certain sections contain the "inactive: " flag. We will need to activate the appropriate blocks later.

```
security {
    ike {
        gateway lab-fw-1 {
            ike-policy lab-ike-policy;
            address 192.168.7.154;
            dead-peer-detection {
                always-send;
                interval 60;
                threshold 5;
            }
            external-interface ge-0/0/0.0;
            version v2-only;
        }
        gateway lab-fw-2 {
            ike-policy lab-ike-policy;
            address 172.30.44.146;
            dead-peer-detection {
                always-send;
                interval 60;
                threshold 5;
            }
            external-interface ge-0/0/0.0;
            version v2-only;
        }
        gateway lab-fw-3 {
            ike-policy lab-ike-policy;
            address 172.16.254.22;
            dead-peer-detection {
                always-send;
                interval 60;
                threshold 5;
            }
            external-interface ge-0/0/0.0;
            version v2-only;
        }
        gateway lab-fw-4 {
            ike-policy lab-ike-policy;
            address 192.168.72.74;
            dead-peer-detection {
                always-send;
                interval 60;
                threshold 5;
            }
            external-interface ge-0/0/0.0;
            version v2-only;
        }
    }
    ipsec {
        inactive: vpn lab-fw-1 {
            bind-interface st0.0;
            df-bit clear;
            ike {
                gateway lab-fw-1;
                ipsec-policy lab-ipsec-policy;
            }
            establish-tunnels immediately;
        }
        inactive: vpn lab-fw-2 {
            bind-interface st0.1;
            df-bit clear;
            ike {
                gateway lab-fw-2;
                ipsec-policy lab-ipsec-policy;
            }
            establish-tunnels immediately;
        }
        inactive: vpn lab-fw-3 {
            bind-interface st0.2;
            df-bit clear;
            ike {
                gateway lab-fw-3;
                ipsec-policy lab-ipsec-policy;
            }
            establish-tunnels immediately;
        }
        inactive: vpn lab-fw-4 {
            bind-interface st0.3;
            df-bit clear;
            ike {
                gateway lab-fw-4;
                ipsec-policy lab-ipsec-policy;
            }
            establish-tunnels immediately;
        }
    }
    zones {
        security-zone untrust {
            host-inbound-traffic {
                system-services {
                    ike;
                }
            }
        }
    }
}
```

### Create interfaces and assign them to a security zone,

Note that we have not set IP addresses for any of these interfaces yet. We'll do that in the next step.


```
interfaces {
    lo0 {
        unit 0 {
            family inet;
        }
    }
    st0 {
        unit 0 {
            family inet;
        }
        unit 1 {
            family inet;
        }
        unit 2 {
            family inet;
        }
        unit 3 {
            family inet;
        }
    }
}
security {
    zones {
        security-zone trust {
            interfaces {
                lo0.0;
            }
        }
        security-zone lab-vpn {
            host-inbound-traffic {
                system-services {
                    all;
                }
                protocols {
                    ospf;
                }
            }
            interfaces {
                inactive: st0.0;
                inactive: st0.1;
                inactive: st0.2;
                inactive: st0.3;
            }
        }
    }
```

### Configure IP addresses on the newly created interfaces and set static routes for the networks on the other end of the VPN tunnels, activate VPN configuration for the appropriate tunnels



<details><summary>lab-fw-1</summary>

#### lab-fw-1

From configuration mode, issue the `load set terminal` command, then paste the following block of configuation. hit `[ENTER]` and then press `[CTRL] + [D]`.

```
set interfaces lo0.0 family inet address 10.1.0.1/32
set interfaces st0.1 family inet address 10.0.12.1/30
set routing-options static route 10.0.0.0/8 discard
set routing-options static route 10.0.23.0/30 next-hop st0.1
set routing-options static route 10.0.34.0/30 next-hop st0.1
set routing-options static route 10.2.0.0/16 next-hop st0.1
set routing-options static route 10.3.0.0/16 next-hop st0.1
set routing-options static route 10.4.0.0/16 next-hop st0.1
activate security zones security-zone lab-vpn interfaces st0.1
activate security ipsec vpn lab-fw-2
```

</details>
<details><summary>lab-sw-2</summary>

#### lab-fw-2

From configuration mode, issue the `load set terminal` command, then paste the following block of configuation. hit `[ENTER]` and then press `[CTRL] + [D]`.

```
set interfaces lo0.0 family inet address 10.2.0.1/32
set interfaces st0.0 family inet address 10.0.12.2/30
set interfaces st0.2 family inet address 10.0.23.1/30
set routing-options static route 10.0.0.0/8 discard
set routing-options static route 10.0.34.0/30 next-hop st0.2
set routing-options static route 10.1.0.0/16 next-hop st0.0
set routing-options static route 10.3.0.0/16 next-hop st0.2
set routing-options static route 10.4.0.0/16 next-hop st0.2
activate security zones security-zone lab-vpn interfaces st0.0
activate security zones security-zone lab-vpn interfaces st0.2
activate security ipsec vpn lab-fw-1
activate security ipsec vpn lab-fw-3
```

</details>
<details><summary>lab-sw-3</summary>

#### lab-fw-3

From configuration mode, issue the `load set terminal` command, then paste the following block of configuation. hit `[ENTER]` and then press `[CTRL] + [D]`.

```
set interfaces lo0.0 family inet address 10.3.0.1/32
set interfaces st0.1 family inet address 10.0.23.2/30
set interfaces st0.3 family inet address 10.0.34.1/30
set routing-options static route 10.0.0.0/8 discard
set routing-options static route 10.0.12.0/30 next-hop st0.3
set routing-options static route 10.1.0.0/16 next-hop st0.1
set routing-options static route 10.2.0.0/16 next-hop st0.1
set routing-options static route 10.4.0.0/16 next-hop st0.3
activate security zones security-zone lab-vpn interfaces st0.0
activate security zones security-zone lab-vpn interfaces st0.2
activate security ipsec vpn lab-fw-2
activate security ipsec vpn lab-fw-4
```

</details>
<details><summary>lab-sw-4</summary>

#### lab-fw-4

From configuration mode, issue the `load set terminal` command, then paste the following block of configuation. hit `[ENTER]` and then press `[CTRL] + [D]`.

```
set interfaces lo0.0 family inet address 10.4.0.1/32
set interfaces st0.2 family inet address 10.0.34.2/30
set routing-options static route 10.0.0.0/8 discard
set routing-options static route 10.0.12.0/30 next-hop st0.2
set routing-options static route 10.0.23.0/30 next-hop st0.2
set routing-options static route 10.2.0.0/16 next-hop st0.2
set routing-options static route 10.3.0.0/16 next-hop st0.2
set routing-options static route 10.4.0.0/16 next-hop st0.2
activate security zones security-zone lab-vpn interfaces st0.2
activate security ipsec vpn lab-fw-3
```

</details>

### Configure security policies and firewall filters to allow desired traffic

```
security {
    policies {
        from-zone lab-vpn to-zone junos-host {
            policy lab-vpn-to-trust {
                match {
                    source-address any;
                    destination-address any;
                    application any;
                }
                then {
                    permit;
                }
            } 
        }
        from-zone lab-vpn to-zone trust {
            policy lab-vpn-to-trust {
                match {
                    source-address any;
                    destination-address any;
                    application any;
                }
                then {
                    permit;
                }
            } 
        }
    }
}
```


### check the configuration we have added for errors and commit.

We'll want to `show | compare`, and review the changes we are making before we  `commit check`, and `commit`.

Use the `show security ike security-associations`, `show security ipsec security-associations`, `show route`, `ping <destination>` and `traceroute <destination` commands to see if the VPNs are up and passing traffic.

If everything is working as expected, let's harden our security a little in the next section.


### Harden the public IKE and IPSEC services on the device using security policies

```
security {
    address-book {
        global {
            address lab-fw-1 192.168.7.154/32;
            address lab-fw-2 172.30.44.146/32;
            address lab-fw-3 172.16.254.22/32;
            address lab-fw-4 192.168.72.74/32;
        }
    }
    policies {
        from-zone untrust to-zone junos-host {
            policy ipsec {
                match {
                    source-address [ lab-fw-1 lab-fw-2 lab-fw-3 lab-fw-4 ];
                    destination-address any;
                    source-address-excluded;
                    application [ junos-ike junos-ike-nat ah esp ];
                }
                then {
                    deny;
                    count;
                }
            }
        }
    }
}
applications {
    application esp protocol esp;
    application ah protocol ah;
}
```


### Harden the public IKE and IPSEC services on the device using a firewall filter

```
policy-options {
    prefix-list vpn-endpoints {
        172.16.254.22/32
        172.30.44.146/32
        192.168.7.154/32
        192.168.72.74/32
    }
}
firewall{
    family inet {
        filter default-accept {
            term accept {
                then {
                    accept;
                }
            }
        }
        filter vpn-endpoints {
            term ike {
                from {
                    source-prefix-list {
                        vpn-endpoints except;
                    }
                    protocol udp;
                    destination-port [ 500 4500 ];
                }
                then {
                    discard;
                }
            }
            term ipsec {
                from {
                    source-prefix-list {
                        vpn-endpoints except;
                    }
                    protocol [ esp ah ];
                }
                then {
                    discard;
                }
            }
        }
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                filter {
                    input-list [ vpn-endpoints default-accept ];
                }
            }
        }
    }
}
```

