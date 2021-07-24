# Basic-MPLS

One of the important use case for using MPLS in Service Provider networks is that the Intermedite/Provider (P) routers need not have the whole IPv4/IPv6 routes in their routing table. Instead, they only need to have labels to reach the remote end PE (Provider Edge) routers. 

To illustrate this, let's assume that we are a service provider (ABC Networks) where our customer needs reachability betwen CE1 and CE2. 

![](images/1.png)

We are running ISIS IGP throughout our network core. 

If we were to use an IP based network only, then P3 and P4 routers will also have to install customer prefix 10.1.1.0/24 and 20.1.1.0/24 in the routing table. Imagine, multiple customers connected to our PE routers. This will become difficult to scale our network. 

Let us now introduce MPLS to our network to avoid installing customer prefixes throughout our core network. In Cisco IOS XE, we would need to - 

1. Enable CEF 
2. Define MPLS label protocol as LDP
3. Define Router-ID for LDP (Note that this IP should be reachable from the peer's perspective)
4. Enable MPLS on core network facing interfaces

Let's go through the corresponding commands - 

```
PE1(config)#ip cef distributed 
PE1(config)#mpls label protocol ldp 
PE1(config)#mpls ldp router-id loopback 1
PE1(config)#int range gigabitEthernet 1-2
PE1(config-if-range)#mpls ip
```

That should enable MPLS with LDP to exchange labels for prefixes. Let's verify with the help of a show command - 

```
PE1#show mpls interfaces 
Interface              IP            Tunnel   BGP Static Operational
GigabitEthernet1       Yes (ldp)     No       No  No     Yes        
GigabitEthernet2       Yes (ldp)     No       No  No     Yes        
```

We can apply similar configs on the other XE routers P3 and P4. Note that instead of going to each interface and enabling MPLS manually, we can ask our IGP to enable MPLS (only on those interfaces running the IGP). Let's enable MPLS on P3.

```
P3(config)#ip cef distributed 
P3(config)#mpls label protocol ldp 
P3(config)#mpls ldp router-id loopback 1

P3(config)#router isis test
P3(config-router)#mpls ldp autoconfig 

P3#show mpls interfaces 
Interface              IP            Tunnel   BGP Static Operational
GigabitEthernet1       Yes (ldp)     No       No  No     Yes        
GigabitEthernet2       Yes (ldp)     No       No  No     Yes        
```
