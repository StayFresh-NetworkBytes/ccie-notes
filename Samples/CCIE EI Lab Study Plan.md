# CCIE EI Lab Study Plan

Oct 22, 2026 · @Matt

## Goal and approach

Pass the CCIE Enterprise Infrastructure v1.1 lab (8 hours, dual-stack IPv4/IPv6) on a first attempt in 2027; the exact date is open until weekly study hours are confirmed.

- ENCOR 350-401 passed on Oct 20, 2026, which qualifies the lab attempt.
- The blueprint is a scorecard, not a reading list: rate each sub-topic 0 (never touched) to 3 (can configure, verify and break/fix from memory without docs).
- Time follows the exam weights: Network Infrastructure 30%, Software-Defined Infrastructure 25%, Transport 15%, Security and Services 15%, Automation 15%.
- Learn by building: build on the CML rack, break it on purpose, and diagnose from show output only.
- Quizzes test recall; timed labs test speed. Every miss goes in the mistakes log below.

## Schedule

&#91;embedded content: plan from Oct 20, 2026 to Jan 31, 2027 · 7 blocks\]

Topic blocks follow the exam weights through Dec 20, design and mixed labs cover the holidays, and full 8-hour mocks start Jan 4; multicast is rebuilt inside the Network Infrastructure block.

Weekly rhythm:

- Two weekday evenings of 1.5 to 2 hours: focused drills on one topic.
- One weekend block of 4 to 5 hours: a timed multi-technology scenario.
- After every session, add misses to the mistakes log.

## Section 1.0 scorecard

Multicast (1.6) is the weakest area at 3/10, OSPF (1.4) is next at 6/12, EIGRP (1.3) and routing concepts (1.2) followed at 7/10 each, BGP (1.5) scored 14/17, switching (1.1) scored 12/15, and the first 16-question set across section 1.0 scored 11/16.

| Subsection | Results so far | Status | Gaps to drill |
| --- | --- | --- | --- |
| 1.1 Switched campus | 3/3 on the first set, then 12/15 on the full set | Mostly fine | UDLD scope (1.1.b ii), extended VLANs and VTP (1.1.c v), STP path cost values (1.1.e ii); nothing timed |
| 1.2 Routing concepts | 1/2 on the first set, then 7/10 on the full set | Gap | Static multicast routes (1.2.b), local PBR (1.2.c), EIGRP HMAC-SHA-256 (1.2.i); EIGRP seed metric (1.2.h) from the first set |
| 1.3 EIGRP | 0/2 on the first set, then 7/10 on the full set; RD/FD and metric formula missed earlier | Improving | Adjacency requirements, variance applied after feasibility, K values under topology base |
| 1.4 OSPF | 1/2 on the first set, then 6/12 on the full set | Gap | Misses across adjacency (1.4.a), network and area types (1.4.c), path preference (1.4.d: E2 tie-break, Type 4 LSAs) and optimization (1.4.f); max-metric router-lsa missed earlier; graceful shutdown (1.4.e ii) still unchecked |
| 1.5 BGP | 2/3 on the first set, then 14/17 on the full set | Mostly fine | as-override / allowas-in (1.5.d); iBGP does not change next hop; hold-time negotiation (1.5.a iii), MED comparison default (1.5.b ii), aggregate as-set (1.5.e ii) |
| 1.6 Multicast | 4/4 easy set, then 3/10 harder set | Weak | Election rules, Auto-RP vs BSR, filters, SSM, anycast RP |

First pass on section 1.0 is complete: multicast (3/10), OSPF (6/12), EIGRP (7/10), routing concepts (7/10), BGP (14/17) and switching (12/15). Next: remediation, starting with the multicast stage 1 build.

## Multicast rebuild plan

Multicast is treated as generally weak and rebuilt in five stages that follow the data path; each stage is a concept check, a CML build with verification commands, then a break/fix drill.

| Stage | Topic | Blueprint | Status |
| --- | --- | --- | --- |
| 1 | Foundations: RPF, PIM-SM, static RP, reading show ip mroute | 1.6.b, 1.6.c(i) | Not started |
| 2 | RP discovery: Auto-RP, BSR, group-to-RP mapping, RP filters | 1.6.c(ii, iii, v) | Not started |
| 3 | SSM, IGMPv3, Layer 2: snooping, querier, filters, MLD | 1.6.a, 1.6.c(iv) | Not started |
| 4 | Anycast RP with MSDP, PIMv6 anycast RP, multipath, boundaries | 1.6.c(vi-viii) | Not started |
| 5 | Fault injection across all stages | all of 1.6 | Not started |

**Stage 1 build:** four routers in a chain (source LAN, R1, R2 as RP, R3, R4, receiver LAN). Enable ip multicast-routing everywhere, ip pim sparse-mode on every link and the RP loopback, a static RP, and ip igmp join-group 239.1.1.1 on the receiver interface.

**Stage 1 verify:** show ip pim neighbor, show ip pim rp mapping, show ip rpf \<source>, and show ip mroute 239.1.1.1 (expect (\*,G) with SC flags first, then (S,G) with T once traffic flows; the first-hop router shows F).

**Stage 1 breaks:**

1. Remove ip pim sparse-mode from one transit interface.
2. Point one router at the wrong RP address.
3. Create an RPF failure by making the unicast path to the source differ from the multicast path.

## Mistakes log

Every miss so far, newest first, with the idea to remember and the symptom that points to it; add a row after each quiz or lab.

| Date | Topic | Correct idea | Symptom or tell |
| --- | --- | --- | --- |
| Oct 24 | EIGRP feasibility (1.3.b i), 6th miss | A path with RD equal to the successor FD is not feasible: RD must be strictly below it. Variance x best FD applies only after feasibility | Equal RD looks like a backup path but the wrong path gets installed |
| Oct 24 | OSPF hello mask check (1.4.a), repeat miss | On broadcast and NBMA, the Hello carries the subnet mask and a mismatch blocks the adjacency; point-to-point does not check it | Neighbors on the same segment never appear |
| Oct 24 | OSPF Type 4 LSAs (1.4.d), repeat miss | Type 5 LSAs flood domain-wide but name the ASBR by router ID; the ABR generates a Type 4 so other areas can reach that ASBR | External routes present but the ASBR is unreachable from other areas |
| Oct 23 | 2.2.b v SD-WAN TLOC extension | A TLOC extension lets a router reach a transport through another local router's WAN interface when it has no direct connection to it | At a dual-edge site, one edge has only one color and no tunnels on the other transport |
| Oct 23 | 2.1.b iii SD-Access SGACL enforcement | The ingress edge classifies and tags the packet with the SGT; the SGACL is enforced at the egress edge, close to the destination | Looking for the policy at the source edge finds nothing; check the destination edge |
| Oct 23 | 4.5.b ii PTP boundary vs transparent clock | A boundary clock terminates PTP on its slave port and acts as master on its other ports; a transparent clock only updates the correction field as packets pass | Design choice: boundary clocks reset delay error at each hop, transparent clocks pass the original timing through |
| Oct 23 | 4.7.b Conditional debug | debug platform condition limits platform debug and packet trace output to matching traffic such as an interface or IP address | Unfiltered debugging floods the console and CPU on a busy router |
| Oct 23 | 4.2.b IPv6 ACL implicit permits | IPv6 ACLs end with implicit permits for neighbor solicitation and neighbor advertisement, then an implicit deny | An explicit deny ipv6 any any overrides the implicit permits and breaks neighbor discovery on that interface |
| Oct 23 | 4.5.d NAT terms | Inside local is the inside host's private address; inside global is the address that host shows to the outside | Mixing up the columns when reading show ip nat translations |
| Oct 23 | 4.5.c Stateless DHCPv6 | RA with the O flag set and M cleared: address from SLAAC, other options like DNS from DHCPv6. M means stateful and gives the address | Hosts get an address but no DNS if the O flag is missing; with M set they wait for a DHCPv6 address |
| Oct 23 | 4.4.e Shaping vs policing | Shaping buffers excess traffic and sends it later; policing drops or re-marks it | Policing shows up as drops and retransmits; shaping shows up as added delay |
| Oct 23 | 3.2.b ii L3VPN label stack | A customer packet carries two labels across the core: a transport (LDP) label and a VPN label. The route target is not a label | With PHP the penultimate hop pops the transport label, so the egress PE sees only the VPN label |
| Oct 23 | 3.2.b i PE-CE BGP with same ASN | as-override on the PE replaces the customer AS with the provider AS; allowas-in on the CE is the other fix | Remote CE drops routes because it sees its own AS in the path (loop prevention) |
| Oct 23 | 3.2.a ii LDP session transport address | The LDP TCP session is built to the transport address, normally the router ID loopback, so it must be reachable in the IGP | Discovery works on the link but the session never comes up |
| Oct 23 | 3.2.a ii LDP discovery and session | UDP 646 hellos to 224.0.0.2 for discovery, then a TCP 646 session | Neighbors are discovered but no session if TCP 646 is blocked |
| Oct 23 | 5.1.d Jinja | {% %} holds statements and a for loop closes with {% endfor %}; {{ }} prints a value; loop.index is 1-based, loop.index0 is 0-based | A template with no endfor errors out; numbering is off by one if you assume loop.index starts at 0 |
| Oct 23 | 5.1.a Python json loads vs dumps | loads parses a JSON string into Python objects; dumps turns Python objects into a JSON string; load and dump (no s) work on files | Using dumps on an API response gives a string you cannot index; a TypeError on string indices |
| Oct 23 | 5.3.c gRPC telemetry | In an IOS XE configured subscription the router dials out to the receiver | No data arrives if the receiver is unreachable or not listening on the configured port |
| Oct 23 | 5.3.a vManage API authentication | POST credentials to /j\_security\_check and keep the JSESSIONID cookie; POST and PUT also need X-XSRF-TOKEN from /dataservice/client/token | Login works but write calls fail without the XSRF token |
| Oct 23 | 5.2.b Guest Shell CLI Python module | cli() returns the output as a string, clip() prints it | A script using clip() has nothing to parse; one using cli() prints nothing until you print the string |
| Oct 23 | OSPF path preference (1.4.d), repeat miss | Intra-area beats inter-area beats E1 beats E2 before metric is considered; a sham link makes the MPLS path intra-area | Backdoor link always wins |
| Oct 23 | Multicast filters, repeat miss | igmp access-group filters joins; multicast boundary blocks both ways; pim neighbor-filter limits PIM neighbors | Wrong scope blocked; pick the command by job |
| Oct 23 | EIGRP feasibility (1.3.b i), 5th miss | A path with RD equal to the successor FD is not feasible: RD must be strictly below it. Variance x best FD applies only after feasibility | Equal RD looks like a backup path but the wrong path gets installed |
| Oct 23 | Static multicast routes (1.2.b), 3rd miss | ip mroute changes the RPF lookup only; it does not forward traffic or install a unicast route | RPF failures despite good unicast routing |
| Oct 23 | EIGRP adjacency requirements (1.3.a), repeat miss | AS number, K values, primary subnet and authentication must match; hello and hold timers need not match | A timer mismatch causes flapping, not failure to form |
| Oct 22 | Multicast SSM (1.6.a) | SSM uses 232.0.0.0/8 with no RP and no shared tree, and needs IGMPv3; ip pim ssm default enables the range | (S,G) join fails |
| Oct 22 | Static multicast routes (1.2.b), repeat miss | ip mroute changes the RPF lookup only; it does not forward traffic or install a unicast route | RPF failures despite good unicast routing |
| Oct 22 | Auto-RP vs BSR, repeat miss | Auto-RP: C-RPs multicast to 224.0.1.39, agent announces on 224.0.1.40, so sparse-only needs ip pim autorp listener. BSR: C-RPs unicast to the BSR, which floods the RP-set to 224.0.0.13 | RP mappings never learned |
| Oct 22 | EIGRP variance after feasibility (1.3.b i), 4th miss | Check feasibility first: RD must be strictly below the successor FD (equal fails). Variance x best FD applies only to the survivors | Equal RD, or a low RD with a high FD, is the trap; wrong path installed |
| Oct 22 | OSPF Type 4 LSAs (1.4.d) | Type 5 LSAs flood domain-wide but name the ASBR by router ID; the ABR generates a Type 4 so other areas can reach that ASBR | External routes present but the ASBR is unreachable from other areas |
| Oct 22 | OSPF E2 tie-break (1.4.d) | E2 metrics ignore internal cost, so equal E2 metrics are broken by the lower cost to reach the ASBR (or forwarding address) | Two ASBRs, same E2 metric, which is preferred |
| Oct 22 | OSPF hello mask check (1.4.a) | On broadcast and NBMA, the Hello carries the subnet mask and a mismatch blocks the adjacency; point-to-point does not check it | Neighbors on the same segment never appear |
| Oct 22 | STP path cost values (1.1.e ii) | Short method: 10M = 100, 100M = 19, 1G = 4, 10G = 2. Long method: 1G = 20000, 10G = 2000 | Root path costs disagree when switches use different methods |
| Oct 22 | Extended VLANs (1.1.c v) | VLANs 1006-4094 need VTP transparent mode in VTPv1/v2 (VTPv3 allows server mode); they live in the running config | VLAN creation refused in server or client mode |
| Oct 22 | UDLD scope (1.1.b ii) | Global udld enable and udld aggressive apply to fiber ports only; copper ports need udld port on the interface | Copper link unprotected despite the global command |
| Oct 22 | BGP aggregate as-set (1.5.e ii) | as-set carries the component AS numbers into the aggregate's path so loop prevention still works; summary-only is what suppresses the specifics | An aggregate returns to a component's AS and is accepted |
| Oct 22 | BGP MED comparison (1.5.b ii) | By default MED is compared only between paths from the same neighboring AS; bgp always-compare-med changes that | A lower MED across different ASes has no effect on best path |
| Oct 22 | BGP hold-time negotiation (1.5.a iii) | Peers use the lower of the two proposed hold times; a mismatch does not prevent the session | Session is up with the smaller value |
| Oct 22 | EIGRP authentication (1.2.i) | Classic mode: MD5 with a key chain. Named mode adds HMAC-SHA-256, set under af-interface | Which authentication option exists only in named mode |
| Oct 22 | Local PBR (1.2.c) | Interface ip policy route-map only affects traffic arriving on that interface; router-originated traffic needs ip local policy route-map | Pings from the router ignore the policy |
| Oct 22 | Static multicast routes (1.2.b) | ip mroute overrides the RPF lookup only; it does not forward traffic or install a unicast route | Multicast dropped on RPF failure despite good unicast routing |
| Oct 22 | EIGRP named mode K values (1.3.d) | Metric weights are set under topology base with metric weights; af-interface is only for per-interface settings | Which sub-mode holds a setting depends on its scope |
| Oct 22 | EIGRP variance after feasibility (1.3.b i) | Feasibility first: RD must be strictly below the successor FD (equal fails). Then variance x best FD applies only to the survivors | Third miss on this topic; a path with equal RD or a low RD but a high FD is the trap |
| Oct 22 | EIGRP adjacency requirements (1.3.a) | Must match: AS number, K values, primary subnet, authentication. Hello and hold timers need not match | A mismatch that still lets the neighbor form |
| Oct 22 | Multicast elections | IGMP querier: lowest IP. PIM DR: highest priority, then highest IP | Two routers on one LAN; which role each plays |
| Oct 22 | Auto-RP vs BSR | Auto-RP: C-RPs multicast to 224.0.1.39, agent announces on 224.0.1.40. BSR: C-RPs unicast to the BSR, which floods the RP-set | Sparse-only network needs ip pim autorp listener for Auto-RP |
| Oct 22 | Multicast filters | igmp access-group filters joins; multicast boundary blocks both ways; pim neighbor-filter limits PIM neighbors | Which command fits which job |
| Oct 22 | AS path (1.5.d) | Same AS on both sites: CE rejects the route as a loop; fix with as-override on the PE or allowas-in on the CE | CE2 never learns CE1 routes |
| Oct 22 | OSPF stub router (1.4.f) | max-metric router-lsa makes transit links cost 0xFFFF; its own prefixes stay reachable | Draining a router without losing its prefixes |
| Oct 22 | EIGRP named mode (1.3.d) | af-interface holds timers, passive-interface, summary-address; topology base holds redistribution, variance, metrics | Where a setting lives in named mode |
| Oct 22 | EIGRP stub (1.3.e) | Default stub advertises connected and summary routes; neighbors do not query it | Query scope bounded at the spoke |
| Oct 22 | Redistribution into EIGRP (1.2.h) | EIGRP needs a seed metric or the routes are not advertised | Redistributed OSPF routes missing |
| Oct 21 | EIGRP successor selection | Successor = lowest FD (first number); feasible successor needs RD below that FD; variance applies only after feasibility | Read the columns before applying the gates |
| Oct 21 | EIGRP classic metric | 256 x (10^7 / min bandwidth in kbps + cumulative delay in tens of microseconds) | Delay units and the x256 |
| Oct 20 | iBGP next hop | iBGP does not change the next hop; an unreachable next hop shows as inaccessible; fix with next-hop-self or an IGP route | Route valid but no best path |
| Oct 20 | OSPF path preference | Intra-area beats inter-area beats E1 beats E2 before metric is considered; a sham link makes the MPLS path intra-area | Backdoor link always wins |
| Oct 20 | MPLS PHP | Egress advertises implicit-null; the penultimate hop pops the label | Read both halves of a two-part question |
| Oct 20 | DMVPN Phase 3 | ip nhrp redirect on the hub, ip nhrp shortcut on the spokes | Zero Traffic Indication counters on the hub |

## Next steps and open questions

Next up is the 10-question EIGRP set, then the multicast stage 1 build; two inputs are still missing: weekly study hours and a target exam date.

- [x] 10-question EIGRP set (1.3): scored 7/10
- [ ] Multicast stage 1 build and break/fix on CML
- [ ] Check OSPF graceful shutdown (1.4.e ii) on the rack; the OSPF set scored 6/12
- [ ] Timed 4-hour diagnostic lab on the CML rack (speed baseline)
- [ ] Rebuild the scorecard after all six subsection sets
- [ ] Confirm weekly study hours (the March estimate was 5-10 hours a week)
- [ ] Pick a target lab date and check seat availability in the Cisco scheduler
