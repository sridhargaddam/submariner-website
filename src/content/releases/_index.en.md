+++
date = 2016-04-09T16:50:16+02:00
title = "Releases"
pre = "<b>4. </b>"
weight = 15
+++

## v0.2.0 Overlapping CIDR support

> This release is focused on overlapping CIDR support between clusters

* Support for Overlapping CIDRs between clusters (globalnet)
* Enhanced e2e scripts, which will be shared between repositories in the shipyard project (ongoing work)
* Improved e2e deployment by using a local registry.
* Refactoring to support pluggable drivers (in preparation for [WireGuard](https://www.wireguard.com/) support)


## v0.1.1 Submariner with more light

> This release has focused on stability for the Lighthouse support

* Cleaner logging for the submariner-engine
* Cleaner logging for the submariner-route-agent
* Fixed issue with wrong token stored in subm file #244
* Added flag to disable the OpenShift CVO #235
* Fixed several service-discovery related bugs #194 , #167
* Fixed several panics on nil network discovery
* Added checks to ensure the CIDRs for joining cluster don't overlap with an existing ones.
* Fix context handling related to service-discovery / kubefed #180
* Use the right CoreDNS image for OpenShift.

## v0.1.0 Submariner with some light

> This release has focused on stability, bugfixes and making https://github.com/submariner.io/lighthouse available as developer preview via subctl deployments.

* Several bugfixes and enhancements around HA failover (#346, #348, #332)
* Migrated to Daemonsets for submariner gateway deployment
* Added support for hostNetwork to remote pod/service connectivity (#288)
* Auto detection and configuration of MTU for vx-submariner, jumbo frames support (#301)
* Support for updated strongswan (#288)
* Better iptables detection for some hosts (#227)

> subctl and the submariner operator have the following improvements

* support to verify-connectivity between two connected clusters
* deployment of submariner gateways based in daemonsets instead of deployments
* renaming submariner pods to "submariner-gateway" pods for clarity
* print version details on crash (subctl)
* stop storing IPSEC key on broker during deploy-broker, now it's only contained into the .subm file
* version command for subctl
* nicer spinners during deployment (thanks to kind)


## v0.0.3 -- KubeCon NA 2019

>Submariner has been greatly enhanced to allow operators to deploy into Kubernetes clusters without the necessity for layer-2 adjacency for nodes. Submariner now allows for VXLAN interconnectivity between nodes (facilitated by the route agent). Subctl
was created to make deployment of submariner easier.

## v0.0.1 Second Submariner release
## v0.0.1 First Submariner release
