---
layout: post
tags: ios, netconf, restconf
title: NETCONF, RESTCONF on IOS XE
---

There is a lot of buzz around network APIs such as NETCONF and RESTCONF.  Here we'll take a quick a look at these APIs on Cisco IOS XE.  On the surface, it seems Cisco IOS XE is the first network device platform that supports NETCONF and RESTCONF both driven from YANG models.

> Technically, RESTCONF isn't officially supported or even seen in the CLI to enable it, but more on that later.

## YANG

When APIs are model driven, the model is the source of truth.  If done right, all API documentation and configuration validation could occur using tooling built directly from the models.  YANG is the leading data modeling language and as such, all API requests using RESTCONF/NETCONF are directly modeled from the YANG models IOS XE supports.  For this post, we'll just say the models can easily be represented as JSON k/v pairs or XML documents.  We'll cover YANG in more detail in a future post.

## NETCONF

You can directly access the NETCONF server on IOS XE using the following SSH command (or equivalent from a SSH client).  

> The NETCONF server is a SSH sub-system.

```
$ ssh -p 830 ntc@csr1kv -s netconf 
```

The full response from the IOS XE NETCONF server can be seen below.

When you get the response from the device, you need to respond with client capabilities, and then can you can enter NETCONF request objects into that terminal session directly communicating to the device using NETCONF --- all without writing any code.  This is a good way to ensure your XML objects are built properly before testing them out in any type of script.

So first, we can paste this object into the terminal:

```
<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <capabilities>
        <capability>urn:ietf:params:netconf:base:1.0</capability>
    </capabilities>
</hello>]]>]]>
```

Now the client and the server have exchanged capabilities and the client is now able to send NETCONF request objects.

We are going to query the device for the IP configuration on the GigabitEthernet2 interface.  We'll do this by sending the following object to the device (still in the same terminal session from above).

> This is not a typical interactive session, so don't be alarmed if you aren't getting feedback from the device before pasting in this object.


```xml
<?xml version="1.0"?>
<nc:rpc message-id="101" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
    <nc:get>
        <nc:filter type="subtree">
            <native xmlns="http://cisco.com/ns/yang/ned/ios">
             <interface>
              <GigabitEthernet>
               <name>2</name>
                 <ip></ip>
              </GigabitEthernet>
             </interface>
            </native>
        </nc:filter>
    </nc:get>
</nc:rpc>
]]>]]>
```

This is the response we get back:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="101" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0"><data><native xmlns="http://cisco.com/ns/yang/ned/ios"><interface><GigabitEthernet><name>2</name><ip><address><primary><address>10.1.1.1</address><mask>255.255.255.0</mask></primary></address></ip></GigabitEthernet></interface></native></data></rpc-reply>]]>]]>
```

And if we clean up the response:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="101">
   <data>
      <native xmlns="http://cisco.com/ns/yang/ned/ios">
         <interface>
            <GigabitEthernet>
               <name>2</name>
               <ip>
                  <address>
                     <primary>
                        <address>10.1.1.1</address>
                        <mask>255.255.255.0</mask>
                     </primary>
                  </address>
               </ip>
            </GigabitEthernet>
         </interface>
      </native>
   </data>
</rpc-reply>]]>]]>
```

We can see the structured XML response that makes it extremely easy to programmatically get data out of devices (as well as configure them).  Say good bye to manual parsing forever.


## RESTCONF

The RESTCONF API on IOS XE is built from the same models NETCONF is using.  You also have your choice if you want to use XML or JSON data encoding when using RESTCONF.

Here we'll use JSON.

The following URL using an HTTP GET accomplishes the same thing as shown in the previous NETCONF GET operation.

```
HTTP GET
http://csr1kv/restconf/api/config/native/interface/GigabitEthernet/2/ip/address
```

The JSON response returned back to us is this:

```
{
  "ned:address": {
    "primary": {
      "address": "10.1.1.1",
      "mask": "255.255.255.0"
    }
  }
}
```

This maps nicely back into a Python dictionary that we can easily parse and work with.


## Closing


* RESTCONF and NETCONF are both model driven APIs on IOS XE
* RESTCONF is **NOT** the same REST API that has been on the CSR1KV or IOS XE - it's a brand new API
* You'll need 16.3.1 to test this - the testing for this post used the CSR1KV
* The RESTCONF/NETCONF APIs support 100s of YANG models - all testing here was using the native Cisco IOS model.  This is a personal favorite of mine as the full running configuration is modeled in JSON/XML.

### RESTCONF, as good as it seems, is not yet officially supported by TAC and it's actually **hidden** in the CLI.  Why?  Who knows?  But if you're interested in it, make sure Cisco is aware.

As you can see, there is no RESTCONF command:

```
csr1(config)#rest?
% Unrecognized command
csr1(config)#rest
```

But, watch this:

```
csr1(config)#
csr1(config)#restconf
csr1(config)#
csr1(config)#do show run | inc restconf
restconf
csr1(config)#
```

And it then works like a charm.

## Want to test NETCONF/RESTCONF?

Check the Network to Code [Labs](https://labs.networktocode.com).


## NETCONF Server Capabilities 

RESPONSE AS DESCRIBED ABOVE


```
<?xml version="1.0" encoding="UTF-8"?>
<hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
<capabilities>
<capability>urn:ietf:params:netconf:base:1.0</capability>
<capability>urn:ietf:params:netconf:base:1.1</capability>
<capability>urn:ietf:params:netconf:capability:writable-running:1.0</capability>
<capability>urn:ietf:params:netconf:capability:xpath:1.0</capability>
<capability>urn:ietf:params:netconf:capability:validate:1.0</capability>
<capability>urn:ietf:params:netconf:capability:validate:1.1</capability>
<capability>urn:ietf:params:netconf:capability:rollback-on-error:1.0</capability>
<capability>urn:ietf:params:netconf:capability:notification:1.0</capability>
<capability>urn:ietf:params:netconf:capability:interleave:1.0</capability>
<capability>http://tail-f.com/ns/netconf/actions/1.0</capability>
<capability>http://tail-f.com/ns/netconf/extensions</capability>
<capability>urn:ietf:params:netconf:capability:with-defaults:1.0?basic-mode=report-all</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-netconf-with-defaults?revision=2011-06-01&amp;module=ietf-netconf-with-defaults</capability>
<capability>http://cisco.com/ns/example/enable?module=enable</capability>
<capability>http://cisco.com/ns/yang/ned/ios?module=ned&amp;revision=2016-07-01</capability>
<capability>http://cisco.com/ns/yang/ned/ios/asr1k?module=ned-asr1k&amp;revision=2016-04-07</capability>
<capability>http://cisco.com/yang/cisco-ia?module=cisco-ia&amp;revision=2016-05-20</capability>
<capability>http://cisco.com/yang/cisco-odm?module=cisco-odm&amp;revision=2016-05-16</capability>
<capability>http://cisco.com/yang/cisco-self-mgmt?module=cisco-self-mgmt&amp;revision=2016-05-14</capability>
<capability>http://tail-f.com/ns/aaa/1.1?module=tailf-aaa&amp;revision=2015-06-16</capability>
<capability>http://tail-f.com/ns/mibs/IPV6-TC/199812010000Z?module=IPV6-TC&amp;revision=1998-12-01</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-COMMUNITY-MIB/200308060000Z?module=SNMP-COMMUNITY-MIB&amp;revision=2003-08-06</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-FRAMEWORK-MIB/200210140000Z?module=SNMP-FRAMEWORK-MIB&amp;revision=2002-10-14</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-MPD-MIB/200210140000Z?module=SNMP-MPD-MIB&amp;revision=2002-10-14</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-NOTIFICATION-MIB/200210140000Z?module=SNMP-NOTIFICATION-MIB&amp;revision=2002-10-14</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-TARGET-MIB/200210140000Z?module=SNMP-TARGET-MIB&amp;revision=2002-10-14</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-USER-BASED-SM-MIB/200210160000Z?module=SNMP-USER-BASED-SM-MIB&amp;revision=2002-10-16</capability>
<capability>http://tail-f.com/ns/mibs/SNMP-VIEW-BASED-ACM-MIB/200210160000Z?module=SNMP-VIEW-BASED-ACM-MIB&amp;revision=2002-10-16</capability>
<capability>http://tail-f.com/ns/mibs/SNMPv2-MIB/200210160000Z?module=SNMPv2-MIB&amp;revision=2002-10-16</capability>
<capability>http://tail-f.com/ns/mibs/SNMPv2-SMI/1.0?module=SNMPv2-SMI</capability>
<capability>http://tail-f.com/ns/mibs/SNMPv2-TC/1.0?module=SNMPv2-TC</capability>
<capability>http://tail-f.com/ns/mibs/TRANSPORT-ADDRESS-MIB/200211010000Z?module=TRANSPORT-ADDRESS-MIB&amp;revision=2002-11-01</capability>
<capability>http://tail-f.com/ns/webui?module=tailf-webui&amp;revision=2013-03-07</capability>
<capability>http://tail-f.com/yang/acm?module=tailf-acm&amp;revision=2013-03-07</capability>
<capability>http://tail-f.com/yang/common-monitoring?module=tailf-common-monitoring&amp;revision=2013-06-14</capability>
<capability>http://tail-f.com/yang/confd-monitoring?module=tailf-confd-monitoring&amp;revision=2013-06-14</capability>
<capability>http://tail-f.com/yang/netconf-monitoring?module=tailf-netconf-monitoring&amp;revision=2014-11-13</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-acl-oper?module=cisco-acl-oper&amp;revision=2016-03-30</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-bfd-state?module=cisco-bfd-state&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-bgp-state?module=cisco-bgp-state&amp;revision=2015-10-16</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-bridge-common?module=cisco-bridge-common&amp;revision=2014-09-25&amp;features=configurable-bd-mac-limit-notif,configurable-bd-mac-limit-max,configurable-bd-mac-limit-actions,configurable-bd-mac-aging-types,configurable-bd-flooding-control</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-bridge-domain?module=cisco-bridge-domain&amp;revision=2014-12-01&amp;features=parameterized-bridge-domains,configurable-bd-storm-control,configurable-bd-static-mac,configurable-bd-snooping-profiles,configurable-bd-sh-group-number,configurable-bd-mtu,configurable-bd-member-features,configurable-bd-mac-secure,configurable-bd-mac-features,configurable-bd-mac-event-action,configurable-bd-ipsg,configurable-bd-groups,configurable-bd-flooding-mode,configurable-bd-flooding,configurable-bd-dai,clear-bridge-domain</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-cfm-stats?module=cisco-cfm-stats&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-cfm-stats-dev?module=cisco-cfm-stats-dev&amp;revision=2015-05-27</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-checkpoint-archive?module=cisco-checkpoint-archive&amp;revision=2015-05-20</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-efp-stats?module=cisco-efp-stats&amp;revision=2015-07-07</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-environment?module=cisco-environment&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-ethernet?module=cisco-ethernet&amp;revision=2016-05-10</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-flow-monitor?module=cisco-flow-monitor&amp;revision=2015-10-26</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-ip-sla-stats?module=cisco-ip-sla-stats&amp;revision=2015-05-29</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-ip-sla-stats-dev?module=cisco-ip-sla-stats-dev&amp;revision=2015-06-30</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-lldp-state?module=cisco-lldp-state&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-memory-stats?module=cisco-memory-stats&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-mpls-fwd?module=cisco-mpls-fwd&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-platform-software?module=cisco-platform-software&amp;revision=2015-07-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-process-cpu?module=cisco-process-cpu&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-process-memory?module=cisco-process-memory&amp;revision=2015-04-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-qos-action-marking-cfg?module=cisco-qos-action-marking-cfg&amp;revision=2015-05-09&amp;features=set-wlan-user-priority-support,set-vlan-inner-support,set-srp-priority-support,set-qos-grp-support,set-prec-tunnel-support,set-prec-support,set-mpls-exp-top-support,set-mpls-exp-imp-support,set-fr-fecn-becn-support,set-fr-de-support,set-dscp-tunnel-support,set-discard-class-support,set-dei-support,set-dei-imp-support,set-cos-support,set-cos-inner-suppport,set-atm-clp-support</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-qos-action-oper?module=cisco-qos-action-oper&amp;revision=2015-05-09&amp;features=queue-peak-size-stats-support,priority-bandwidth-exceed-drops-support,marking-stats-support,drop-pkts-no-buffer-stats-support,drop-pkts-flow-stats-support,aggregate-priority-stats-support</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-qos-action-qlimit-cfg?module=cisco-qos-action-qlimit-cfg&amp;revision=2015-05-09&amp;features=qos-grp-based-queuing-support,mpls-exp-based-queuing-support,disc-class-based-queuing-support</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-qos-common?module=cisco-qos-common&amp;revision=2015-05-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-routing-ext?module=cisco-routing-ext&amp;revision=2016-07-09</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-storm-control?module=cisco-storm-control&amp;revision=2014-09-25&amp;features=configurable-storm-control-actions</capability>
<capability>urn:cisco:params:xml:ns:yang:cisco-virtual-service?module=cisco-virtual-service&amp;revision=2016-04-19</capability>
<capability>urn:cisco:params:xml:ns:yang:govern?module=govern&amp;revision=2014-07-16</capability>
<capability>urn:cisco:params:xml:ns:yang:pim?module=pim&amp;revision=2014-06-27&amp;features=bsr,auto-rp</capability>
<capability>urn:cisco:params:xml:ns:yang:pw?module=cisco-pw&amp;revision=2014-12-01&amp;features=static-label-direct-config,pw-vccv,pw-tag-impose-vlan-id,pw-status-config,pw-static-oam-config,pw-short-config,pw-sequencing,pw-preferred-path,pw-port-profiles,pw-oam-refresh-config,pw-mac-withdraw-config,pw-load-balancing,pw-ipv6-source,pw-interface,pw-grouping-config,pw-class-tag-rewrite,pw-class-switchover-delay,pw-class-status,pw-class-source-ip,pw-class-flow-setting,preferred-path-peer,predictive-redundancy-config,flow-label-tlv-code17,flow-label-static-config</capability>
<capability>urn:cisco:params:xml:ns:yang:table-map?module=cisco-table-map&amp;revision=2015-05-19&amp;features=table-map-template-support</capability>
<capability>urn:ietf:params:xml:ns:yang:c3pl-types?module=policy-types&amp;revision=2013-10-07&amp;features=protocol-name-support,match-wlan-user-priority-support,match-vpls-support,match-vlan-support,match-vlan-inner-support,match-src-mac-support,match-security-group-support,match-qos-group-support,match-prec-support,match-packet-length-support,match-mpls-exp-top-support,match-mpls-exp-imp-support,match-metadata-support,match-ipv6-acl-support,match-ipv6-acl-name-support,match-ipv4-acl-support,match-ipv4-acl-name-support,match-ip-rtp-support,match-input-interface-support,match-fr-dlci-support,match-fr-de-support,match-flow-record-support,match-flow-ip-support,match-dst-mac-support,match-discard-class-support,match-dei-support,match-dei-inner-support,match-cos-support,match-cos-inner-support,match-class-map-support,match-atm-vci-support,match-atm-clp-support,match-application-support</capability>
<capability>urn:ietf:params:xml:ns:yang:cisco-ospf?module=cisco-ospf&amp;revision=2016-03-30&amp;features=graceful-shutdown,flood-reduction,database-filter</capability>
<capability>urn:ietf:params:xml:ns:yang:cisco-policy?module=cisco-policy&amp;revision=2016-03-30</capability>
<capability>urn:ietf:params:xml:ns:yang:cisco-policy-filters?module=cisco-policy-filters&amp;revision=2016-03-30</capability>
<capability>urn:ietf:params:xml:ns:yang:cisco-policy-target?module=cisco-policy-target&amp;revision=2016-03-30</capability>
<capability>urn:ietf:params:xml:ns:yang:common-mpls-static?module=common-mpls-static&amp;revision=2015-07-22&amp;deviations=common-mpls-static-devs</capability>
<capability>urn:ietf:params:xml:ns:yang:common-mpls-types?module=common-mpls-types&amp;revision=2015-05-28</capability>
<capability>urn:ietf:params:xml:ns:yang:iana-crypt-hash?module=iana-crypt-hash&amp;revision=2014-04-04&amp;features=crypt-hash-sha-512,crypt-hash-sha-256,crypt-hash-md5</capability>
<capability>urn:ietf:params:xml:ns:yang:iana-if-type?module=iana-if-type&amp;revision=2014-05-08</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-diffserv-action?module=ietf-diffserv-action&amp;revision=2015-04-07&amp;features=priority-rate-burst-support,hierarchial-policy-support,aqm-red-support</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-diffserv-classifier?module=ietf-diffserv-classifier&amp;revision=2015-04-07&amp;features=policy-inline-classifier-config</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-diffserv-policy?module=ietf-diffserv-policy&amp;revision=2015-04-07&amp;features=policy-template-support,hierarchial-policy-support</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-diffserv-target?module=ietf-diffserv-target&amp;revision=2015-04-07&amp;features=target-inline-policy-config</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-inet-types?module=ietf-inet-types&amp;revision=2013-07-15</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-interfaces?module=ietf-interfaces&amp;revision=2014-05-08&amp;features=pre-provisioning,if-mib,arbitrary-names&amp;deviations=ietf-ip-devs</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-interfaces-ext?module=ietf-interfaces-ext</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-ip?module=ietf-ip&amp;revision=2014-01-08&amp;features=ipv6-privacy-autoconf,ipv4-non-contiguous-netmasks</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-ipv4-unicast-routing?module=ietf-ipv4-unicast-routing&amp;revision=2015-05-25</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-ipv6-unicast-routing?module=ietf-ipv6-unicast-routing&amp;revision=2015-05-25</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-key-chain?module=ietf-key-chain&amp;revision=2015-02-24&amp;features=independent-send-accept-lifetime,hex-key-string,accept-tolerance</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-netconf-acm?module=ietf-netconf-acm&amp;revision=2012-02-22</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-netconf-monitoring?module=ietf-netconf-monitoring&amp;revision=2010-10-04</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-netconf-notifications?module=ietf-netconf-notifications&amp;revision=2012-02-06</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-ospf?module=ietf-ospf&amp;revision=2015-03-09&amp;features=ttl-security,te-rid,router-id,remote-lfa,prefix-suppression,ospfv3-authentication-ipsec,nsr,node-flag,multi-topology,multi-area-adj,mtu-ignore,max-lsa,max-ecmp,lls,lfa,ldp-igp-sync,ldp-igp-autoconfig,interface-inheritance,instance-inheritance,graceful-restart,fast-reroute,demand-circuit,bfd,auto-cost,area-inheritance,admin-control&amp;deviations=ietf-ospf-devs</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-routing?module=ietf-routing&amp;revision=2015-05-25&amp;features=router-id,multiple-ribs&amp;deviations=ietf-ipv4-unicast-routing-devs,ietf-ipv6-unicast-routing-devs,ietf-ospf-devs,ietf-routing-devs</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-yang-smiv2?module=ietf-yang-smiv2&amp;revision=2012-06-22</capability>
<capability>urn:ietf:params:xml:ns:yang:ietf-yang-types?module=ietf-yang-types&amp;revision=2013-07-15</capability>
<capability>urn:ietf:params:xml:ns:yang:nvo?module=nvo&amp;revision=2015-06-02&amp;deviations=nvo-devs</capability>
<capability>urn:ietf:params:xml:ns:yang:policy-attr?module=policy-attr&amp;revision=2015-04-27</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ATM-FORUM-TC-MIB?module=ATM-FORUM-TC-MIB</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ATM-MIB?module=ATM-MIB&amp;revision=1998-10-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ATM-TC-MIB?module=ATM-TC-MIB&amp;revision=1998-10-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:BGP4-MIB?module=BGP4-MIB&amp;revision=1994-05-05</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:BRIDGE-MIB?module=BRIDGE-MIB&amp;revision=2005-09-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-AAA-SERVER-MIB?module=CISCO-AAA-SERVER-MIB&amp;revision=2003-11-17</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-AAA-SESSION-MIB?module=CISCO-AAA-SESSION-MIB&amp;revision=2006-03-21</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-AAL5-MIB?module=CISCO-AAL5-MIB&amp;revision=2003-09-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ATM-EXT-MIB?module=CISCO-ATM-EXT-MIB&amp;revision=2003-01-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ATM-PVCTRAP-EXTN-MIB?module=CISCO-ATM-PVCTRAP-EXTN-MIB&amp;revision=2003-01-20</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ATM-QOS-MIB?module=CISCO-ATM-QOS-MIB&amp;revision=2002-06-10</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-BGP-POLICY-ACCOUNTING-MIB?module=CISCO-BGP-POLICY-ACCOUNTING-MIB&amp;revision=2002-07-26</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-BGP4-MIB?module=CISCO-BGP4-MIB&amp;revision=2010-09-30</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-BULK-FILE-MIB?module=CISCO-BULK-FILE-MIB&amp;revision=2002-06-10</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CBP-TARGET-MIB?module=CISCO-CBP-TARGET-MIB&amp;revision=2006-05-24</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CBP-TARGET-TC-MIB?module=CISCO-CBP-TARGET-TC-MIB&amp;revision=2006-03-24</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CBP-TC-MIB?module=CISCO-CBP-TC-MIB&amp;revision=2008-06-24</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CDP-MIB?module=CISCO-CDP-MIB&amp;revision=2005-03-21</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CEF-TC?module=CISCO-CEF-TC&amp;revision=2005-09-30</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CONFIG-COPY-MIB?module=CISCO-CONFIG-COPY-MIB&amp;revision=2005-04-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CONFIG-MAN-MIB?module=CISCO-CONFIG-MAN-MIB&amp;revision=2007-04-27</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-CONTEXT-MAPPING-MIB?module=CISCO-CONTEXT-MAPPING-MIB&amp;revision=2008-11-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-DATA-COLLECTION-MIB?module=CISCO-DATA-COLLECTION-MIB&amp;revision=2002-10-30</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-DIAL-CONTROL-MIB?module=CISCO-DIAL-CONTROL-MIB&amp;revision=2005-05-26</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-DOT3-OAM-MIB?module=CISCO-DOT3-OAM-MIB&amp;revision=2006-05-31</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-DYNAMIC-TEMPLATE-MIB?module=CISCO-DYNAMIC-TEMPLATE-MIB&amp;revision=2007-09-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-DYNAMIC-TEMPLATE-TC-MIB?module=CISCO-DYNAMIC-TEMPLATE-TC-MIB&amp;revision=2012-01-27</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-EIGRP-MIB?module=CISCO-EIGRP-MIB&amp;revision=2004-11-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-EMBEDDED-EVENT-MGR-MIB?module=CISCO-EMBEDDED-EVENT-MGR-MIB&amp;revision=2006-11-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ENTITY-ALARM-MIB?module=CISCO-ENTITY-ALARM-MIB&amp;revision=1999-07-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ENTITY-FRU-CONTROL-MIB?module=CISCO-ENTITY-FRU-CONTROL-MIB&amp;revision=2013-08-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ENTITY-SENSOR-MIB?module=CISCO-ENTITY-SENSOR-MIB&amp;revision=2015-01-15</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ENTITY-VENDORTYPE-OID-MIB?module=CISCO-ENTITY-VENDORTYPE-OID-MIB&amp;revision=2014-12-09</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ETHERLIKE-EXT-MIB?module=CISCO-ETHERLIKE-EXT-MIB&amp;revision=2010-06-04</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-FIREWALL-TC?module=CISCO-FIREWALL-TC&amp;revision=2006-03-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-FTP-CLIENT-MIB?module=CISCO-FTP-CLIENT-MIB&amp;revision=2006-03-31</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-HSRP-EXT-MIB?module=CISCO-HSRP-EXT-MIB&amp;revision=2010-09-02</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-HSRP-MIB?module=CISCO-HSRP-MIB&amp;revision=2010-09-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-ATM2-PVCTRAP-MIB?module=CISCO-IETF-ATM2-PVCTRAP-MIB&amp;revision=1998-02-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-ATM2-PVCTRAP-MIB-EXTN?module=CISCO-IETF-ATM2-PVCTRAP-MIB-EXTN&amp;revision=2000-07-11</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-BFD-MIB?module=CISCO-IETF-BFD-MIB&amp;revision=2011-04-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-FRR-MIB?module=CISCO-IETF-FRR-MIB&amp;revision=2008-04-29</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-ISIS-MIB?module=CISCO-IETF-ISIS-MIB&amp;revision=2005-08-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-MPLS-ID-STD-03-MIB?module=CISCO-IETF-MPLS-ID-STD-03-MIB&amp;revision=2012-06-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-MPLS-TE-EXT-STD-03-MIB?module=CISCO-IETF-MPLS-TE-EXT-STD-03-MIB&amp;revision=2012-06-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-PW-ATM-MIB?module=CISCO-IETF-PW-ATM-MIB&amp;revision=2005-04-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-PW-ENET-MIB?module=CISCO-IETF-PW-ENET-MIB&amp;revision=2002-09-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-PW-MIB?module=CISCO-IETF-PW-MIB&amp;revision=2004-03-17</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-PW-MPLS-MIB?module=CISCO-IETF-PW-MPLS-MIB&amp;revision=2003-02-26</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-PW-TC-MIB?module=CISCO-IETF-PW-TC-MIB&amp;revision=2006-07-21</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IETF-PW-TDM-MIB?module=CISCO-IETF-PW-TDM-MIB&amp;revision=2006-07-21</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IF-EXTENSION-MIB?module=CISCO-IF-EXTENSION-MIB&amp;revision=2013-03-13</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IMAGE-LICENSE-MGMT-MIB?module=CISCO-IMAGE-LICENSE-MGMT-MIB&amp;revision=2007-10-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IMAGE-MIB?module=CISCO-IMAGE-MIB&amp;revision=1995-08-15</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IP-LOCAL-POOL-MIB?module=CISCO-IP-LOCAL-POOL-MIB&amp;revision=2007-11-12</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IP-TAP-MIB?module=CISCO-IP-TAP-MIB&amp;revision=2004-03-11</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IP-URPF-MIB?module=CISCO-IP-URPF-MIB&amp;revision=2011-12-29</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPMROUTE-MIB?module=CISCO-IPMROUTE-MIB&amp;revision=2005-03-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPSEC-FLOW-MONITOR-MIB?module=CISCO-IPSEC-FLOW-MONITOR-MIB&amp;revision=2007-10-24</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPSEC-POLICY-MAP-MIB?module=CISCO-IPSEC-POLICY-MAP-MIB&amp;revision=2000-08-17</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPSLA-AUTOMEASURE-MIB?module=CISCO-IPSLA-AUTOMEASURE-MIB&amp;revision=2007-06-13</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPSLA-ECHO-MIB?module=CISCO-IPSLA-ECHO-MIB&amp;revision=2007-08-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPSLA-JITTER-MIB?module=CISCO-IPSLA-JITTER-MIB&amp;revision=2007-07-24</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-IPSLA-TC-MIB?module=CISCO-IPSLA-TC-MIB&amp;revision=2007-03-23</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-MEDIA-GATEWAY-MIB?module=CISCO-MEDIA-GATEWAY-MIB&amp;revision=2009-02-25</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-MPLS-LSR-EXT-STD-MIB?module=CISCO-MPLS-LSR-EXT-STD-MIB&amp;revision=2012-04-30</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-MPLS-TC-EXT-STD-MIB?module=CISCO-MPLS-TC-EXT-STD-MIB&amp;revision=2012-02-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-NBAR-PROTOCOL-DISCOVERY-MIB?module=CISCO-NBAR-PROTOCOL-DISCOVERY-MIB&amp;revision=2002-08-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-NTP-MIB?module=CISCO-NTP-MIB&amp;revision=2006-07-31</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-OSPF-MIB?module=CISCO-OSPF-MIB&amp;revision=2003-07-18</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-OSPF-TRAP-MIB?module=CISCO-OSPF-TRAP-MIB&amp;revision=2003-07-18</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-PIM-MIB?module=CISCO-PIM-MIB&amp;revision=2000-11-02</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-PING-MIB?module=CISCO-PING-MIB&amp;revision=2001-08-28</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-PRODUCTS-MIB?module=CISCO-PRODUCTS-MIB&amp;revision=2014-11-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-PTP-MIB?module=CISCO-PTP-MIB&amp;revision=2011-01-28</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-QOS-PIB-MIB?module=CISCO-QOS-PIB-MIB&amp;revision=2007-08-29</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-RADIUS-EXT-MIB?module=CISCO-RADIUS-EXT-MIB&amp;revision=2010-05-25</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-RF-MIB?module=CISCO-RF-MIB&amp;revision=2005-09-01</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-RTTMON-TC-MIB?module=CISCO-RTTMON-TC-MIB&amp;revision=2012-05-25</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SESS-BORDER-CTRLR-CALL-STATS-MIB?module=CISCO-SESS-BORDER-CTRLR-CALL-STATS-MIB&amp;revision=2010-09-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SESS-BORDER-CTRLR-STATS-MIB?module=CISCO-SESS-BORDER-CTRLR-STATS-MIB&amp;revision=2010-09-15</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SMI?module=CISCO-SMI&amp;revision=2012-08-29</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SONET-MIB?module=CISCO-SONET-MIB&amp;revision=2003-03-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-ST-TC?module=CISCO-ST-TC&amp;revision=2012-08-08</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-STP-EXTENSIONS-MIB?module=CISCO-STP-EXTENSIONS-MIB&amp;revision=2013-03-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SUBSCRIBER-IDENTITY-TC-MIB?module=CISCO-SUBSCRIBER-IDENTITY-TC-MIB&amp;revision=2011-12-23</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SUBSCRIBER-SESSION-MIB?module=CISCO-SUBSCRIBER-SESSION-MIB&amp;revision=2012-08-08</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SUBSCRIBER-SESSION-TC-MIB?module=CISCO-SUBSCRIBER-SESSION-TC-MIB&amp;revision=2012-01-27</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-SYSLOG-MIB?module=CISCO-SYSLOG-MIB&amp;revision=2005-12-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-TAP2-MIB?module=CISCO-TAP2-MIB&amp;revision=2009-11-06</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-TC?module=CISCO-TC&amp;revision=2011-11-11</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-UBE-MIB?module=CISCO-UBE-MIB&amp;revision=2010-11-29</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VLAN-IFTABLE-RELATIONSHIP-MIB?module=CISCO-VLAN-IFTABLE-RELATIONSHIP-MIB&amp;revision=2013-07-15</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VLAN-MEMBERSHIP-MIB?module=CISCO-VLAN-MEMBERSHIP-MIB&amp;revision=2007-12-14</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VOICE-COMMON-DIAL-CONTROL-MIB?module=CISCO-VOICE-COMMON-DIAL-CONTROL-MIB&amp;revision=2010-06-30</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VOICE-DIAL-CONTROL-MIB?module=CISCO-VOICE-DIAL-CONTROL-MIB&amp;revision=2012-05-15</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VOICE-DNIS-MIB?module=CISCO-VOICE-DNIS-MIB&amp;revision=2002-05-01</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VPDN-MGMT-MIB?module=CISCO-VPDN-MGMT-MIB&amp;revision=2009-06-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:CISCO-VTP-MIB?module=CISCO-VTP-MIB&amp;revision=2013-10-14</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DIAL-CONTROL-MIB?module=DIAL-CONTROL-MIB&amp;revision=1996-09-23</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DIFFSERV-DSCP-TC?module=DIFFSERV-DSCP-TC&amp;revision=2002-05-09</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DIFFSERV-MIB?module=DIFFSERV-MIB&amp;revision=2002-02-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DISMAN-EVENT-MIB?module=DISMAN-EVENT-MIB&amp;revision=2000-10-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DISMAN-EXPRESSION-MIB?module=DISMAN-EXPRESSION-MIB&amp;revision=2000-10-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DRAFT-MSDP-MIB?module=DRAFT-MSDP-MIB&amp;revision=1999-12-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DS1-MIB?module=DS1-MIB&amp;revision=1998-08-01</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:DS3-MIB?module=DS3-MIB&amp;revision=1998-08-01</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ENTITY-MIB?module=ENTITY-MIB&amp;revision=2005-08-10</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ENTITY-SENSOR-MIB?module=ENTITY-SENSOR-MIB&amp;revision=2002-12-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ENTITY-STATE-MIB?module=ENTITY-STATE-MIB&amp;revision=2005-11-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ENTITY-STATE-TC-MIB?module=ENTITY-STATE-TC-MIB&amp;revision=2005-11-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:ETHER-WIS?module=ETHER-WIS&amp;revision=2003-09-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:EXPRESSION-MIB?module=EXPRESSION-MIB&amp;revision=2005-11-24</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:EtherLike-MIB?module=EtherLike-MIB&amp;revision=2003-09-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:FRAME-RELAY-DTE-MIB?module=FRAME-RELAY-DTE-MIB&amp;revision=1997-05-01</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:HCNUM-TC?module=HCNUM-TC&amp;revision=2000-06-08</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IANA-ADDRESS-FAMILY-NUMBERS-MIB?module=IANA-ADDRESS-FAMILY-NUMBERS-MIB&amp;revision=2000-09-08</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IANA-RTPROTO-MIB?module=IANA-RTPROTO-MIB&amp;revision=2000-09-26</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IANAifType-MIB?module=IANAifType-MIB&amp;revision=2006-03-31</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IEEE8021-TC-MIB?module=IEEE8021-TC-MIB&amp;revision=2008-10-15</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IF-MIB?module=IF-MIB&amp;revision=2000-06-14</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IGMP-STD-MIB?module=IGMP-STD-MIB&amp;revision=2000-09-28</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:INET-ADDRESS-MIB?module=INET-ADDRESS-MIB&amp;revision=2005-02-04</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:INT-SERV-MIB?module=INT-SERV-MIB&amp;revision=1997-10-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:INTEGRATED-SERVICES-MIB?module=INTEGRATED-SERVICES-MIB&amp;revision=1995-11-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IP-FORWARD-MIB?module=IP-FORWARD-MIB&amp;revision=1996-09-19</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IP-MIB?module=IP-MIB&amp;revision=2006-02-02</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IPMROUTE-STD-MIB?module=IPMROUTE-STD-MIB&amp;revision=2000-09-22</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:IPV6-FLOW-LABEL-MIB?module=IPV6-FLOW-LABEL-MIB&amp;revision=2003-08-28</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-L3VPN-STD-MIB?module=MPLS-L3VPN-STD-MIB&amp;revision=2006-01-23</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-LDP-GENERIC-STD-MIB?module=MPLS-LDP-GENERIC-STD-MIB&amp;revision=2004-06-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-LDP-STD-MIB?module=MPLS-LDP-STD-MIB&amp;revision=2004-06-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-LSR-STD-MIB?module=MPLS-LSR-STD-MIB&amp;revision=2004-06-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-TC-MIB?module=MPLS-TC-MIB&amp;revision=2001-01-04</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-TC-STD-MIB?module=MPLS-TC-STD-MIB&amp;revision=2004-06-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:MPLS-TE-STD-MIB?module=MPLS-TE-STD-MIB&amp;revision=2004-06-03</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:NHRP-MIB?module=NHRP-MIB&amp;revision=1999-08-26</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:NOTIFICATION-LOG-MIB?module=NOTIFICATION-LOG-MIB&amp;revision=2000-11-27</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:OSPF-MIB?module=OSPF-MIB&amp;revision=2006-11-10</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:OSPF-TRAP-MIB?module=OSPF-TRAP-MIB&amp;revision=2006-11-10</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:P-BRIDGE-MIB?module=P-BRIDGE-MIB&amp;revision=2006-01-09</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:PIM-MIB?module=PIM-MIB&amp;revision=2000-09-28</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:PerfHist-TC-MIB?module=PerfHist-TC-MIB&amp;revision=1998-11-07</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:RFC-1212?module=RFC-1212</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:RFC-1215?module=RFC-1215</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:RFC1155-SMI?module=RFC1155-SMI</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:RFC1315-MIB?module=RFC1315-MIB</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:RMON-MIB?module=RMON-MIB&amp;revision=2000-05-11</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:RSVP-MIB?module=RSVP-MIB&amp;revision=1998-08-25</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:SNMP-FRAMEWORK-MIB?module=SNMP-FRAMEWORK-MIB&amp;revision=2002-10-14</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:SNMP-PROXY-MIB?module=SNMP-PROXY-MIB&amp;revision=2002-10-14</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:SNMP-TARGET-MIB?module=SNMP-TARGET-MIB&amp;revision=1998-08-04</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:SNMPv2-MIB?module=SNMPv2-MIB&amp;revision=2002-10-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:SNMPv2-TC?module=SNMPv2-TC</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:SONET-MIB?module=SONET-MIB&amp;revision=2003-08-11</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:TCP-MIB?module=TCP-MIB&amp;revision=2005-02-18</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:TOKEN-RING-RMON-MIB?module=TOKEN-RING-RMON-MIB</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:TOKENRING-MIB?module=TOKENRING-MIB&amp;revision=1994-10-23</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:TUNNEL-MIB?module=TUNNEL-MIB&amp;revision=2005-05-16</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:UDP-MIB?module=UDP-MIB&amp;revision=2005-05-20</capability>
<capability>urn:ietf:params:xml:ns:yang:smiv2:VPN-TC-STD-MIB?module=VPN-TC-STD-MIB&amp;revision=2005-11-15</capability>
</capabilities>
<session-id>729</session-id></hello>]]>]]>
```


