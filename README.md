# MPLS-L3-VPN
Using Cisco Modeling Labs to create a MPLS L3 VPN 

# Projective Objective
Build and configure an MPLS Layer 3 VPN (L3VPN) network using VRFs, MPLS label switching, and MP-BGP.

The goal is to provide end-to-end connectivity between customer sites while maintaining traffic separation through VRFs.

(Troubleshooting notes and Configs in the Troubleshooting section of the repository. ) 

# Technology used

-VRF's (route distinguisher and route-targets) 

-MP-BGP (VPNv4 route exchange between PE routers) 

-MPLS label switching

- LDP — MPLS label distribution between provider routers

-EIGRP on the Core network

- BGP (PE to CE)

# Troubleshooting Issues

# Issue 1: CE-to-CE Connectivity Failure

After configuring BGP, CE-1 was unable to ping networks behind CE-2. I ran BGP show commands and determined that both CE routers were using ASN 65000. When the routes were advertised to the remote CE, BGP detected its own ASN in the AS_PATH and rejected the routes as part of BGP loop prevention.

Fix

Configured AS-Override on the PE BGP neighbors facing the CE routers within the Porsche VRF. This allows the PE routers to replace the customer ASN in the AS_PATH before advertising the routes to the remote CE.

Ran show bgp ipv4 unicast vrf Porsche to verify that the CE-2 loopback routes were present in the BGP table.

Validation

* Successfully pinged the CE-2 loopback address from CE-1 using the CE-1 loopback as the source.
* Ran a traceroute from CE-1 to verify the end-to-end path.
* Confirmed CE-1 to CE-2 connectivity was restored.

# Issue 2: Route Target Mismatch

Issue:
Only certain customer routes were appearing in the BGP table across the provider network.

Fix:

* Checked the VRF configuration and noticed the Route Targets (RT) did not match.
* Corrected the Route Targets so the exported VPN routes matched the RT being imported by the remote VRF.
* Verified that MP-BGP was properly importing the VPN routes into the Porsche VRF.

Validation:

* Checked the BGP tables and verified the expected routes were appearing for CE-1 and CE-2.
* Tested end-to-end connectivity by pinging the CE-2 loopback address from CE-1.
* Performed a traceroute between the CE loopback networks to verify the path.
* All connectivity tests were successful.
