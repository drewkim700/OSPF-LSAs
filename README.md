**Problem Statement**: 
How to implement an OSPF network to interact with external routing domain (EIGRP) and redistribute routes to ensure all area and domains are reachable to and from each other. 
This case study consists of 12 routers; Area 0 (Backbone area) connected to external domain (EIGRP 100), Area 1 and  Area 2 via ASBR and ABR respectively. 
Area 1 also connects to external domain (EIGRP 200) and Area 2 connects to Area 5. Because interconnecting router between Area 2 and 5 is not an ABR, a **virtual ink** is provisioned to enable communication between areas.
After initial connectivity another consideration is to reduce size of routing table by implementing Stub and NSSA areas in Area2 and Area 1 respectively
Router 1 also connects to ISP to route traffic to the internet.

**Solution and Key implementation**:
After configuring all router interfaces in their respective OSPF areas and EIGRP domains, redistribution of routes across domains is implemented.
For instance on R4 which is an ASBR redistribution of OSPF to EIGRP is done by entering EIGRP 100 context and executing below:
R4(config-router)redistribute ospf 1 metric 1 1 1 1 1
Now on same R4 which EIGRP is redistributed into OSPF by entering OSPF 1 context and executing below:
R4(config-router)redistribute eigrp 100 metric 100 subnets metric-type 1 route-map EIGRP
R4(config)route-map EIGRP 10
R4(config)match ip address 10
R4(config)access-list 10 permit 90.1.0.0 0.0.255.255
The implementation of route-map EIGRP is to eliminate the WAN links in EIGRP domain from being redistributed to allow only 'real' networks to be sent.
R7 is also an ASBR interfacing EIGRP 200 and OSPF Area 1. 
From the EIGRP context
R7(config-router)redistribute ospf 1 metric 1 1 1 1 1
Now on same R7 EIGRP is redistributed into OSPF by entering OSPF 1 context and executing below:
R4(config-router)redistribute eigrp 100 metric 100 subnets metric-type 1 route-map EIGRP
R4(config)route-map EIGRP 10
R4(config)match ip address 10
R4(config)access-list 10 permit 80.1.0.0 0.0.255.255
The implementation of **route-map EIGRP** is to eliminate the WAN links in EIGRP domain from being redistributed to allow only 'real' networks to be sent.
**ABR R2 and R3 generate LSA 4 to advertise ASBR R5** to enable R6 in Area 1 to be able to learn routes in EIGRP 100

**Reducing OSPF database size in production network**
Area 2 (R5, R11, R12) is now configured as stub area which will filter LSA 4 and LSA 5
