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
Area 2 (R5, R11, R12) is configured as stub area which will filter LSA 4 and LSA 5. 
R(config-router)area 2 stub. This takes out LSA 4 and 5 advertisements out of routing table and creates a default LSA 3 summary route via ABR R5.
In Area 1 because it connects to adjoining external domain, configuring stub will stop LSA 5 advertisements necessary for propagation of EIGRP 200 external domain routes. Therefore, NSSA (Not-so-stubby area) is deployed to filter LSA 4 and 5 but allow LSA 7 for ASBR R7. For routers R2, R3, R6, R7 (config-router)area 1 nssa. Now on R7 it learns EIGRP routes from R8 as LSA 7 routes. However, after this implementation EIGRP 100 on R9 cannot be reached. Unlike stub area that generated a default route automatically, in the NSSA implementation a LSA 7 default route needs to injected into ABR R2 and R3 to enable R6 and R7 to learn routes in EIGRP 100. R2(config-router)# area 1 nssa default-information-originate or area 1 nssa translate type 7 always. whereas ASBR R4 generates LSA 5 it is forbidden in Area 1 so ABR R2 and R3 generate LSA 7 to translate LSA5 to enable R6 and R7 to learn of EIGRP 100. 
**Virtual Link**
R12 interfaces area 2 and area 5. Whiles area 5 has 80.1.0.0 and 90.1.0.0 prefixes in database it has no routes in routing table to them because those prefixes are reachable via R2 and R4 respectively which are in area 0. Also, if the show border-routers command is issued it will return empty result. To allow reachability a virtual link is configured on R5 and R12. Now R12 will have virtually interface backbone area and display LSA 1 advertisements to 10.1.0.0 prefixes as if it is in backbone area. It will also learn those prefixes from R5 as LSA 5 summary advertisements. external domain prefixes 80.1.0.0 and 90.1.0.0 are also reachable after virtual link implementation.
**Internet routing:**
Now to enable reachability to internet from all domains through R1 in backbone area a default route is injected from R1 into backbone area R1(config-router)default-information originate always
On R1, NAT is implemented to allow traffic to ISP
