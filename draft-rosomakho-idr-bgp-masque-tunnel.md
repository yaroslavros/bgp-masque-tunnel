---
title: "BGP Signaling of MASQUE Tunnel Encapsulation"
abbrev: "BGP MASQUE Tunnel Encapsulation"
category: std

docname: draft-rosomakho-idr-bgp-masque-tunnel-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "IDR Working Group"
keyword:
 - bgp
 - masque
 - tunnel
venue:
  group: "IDR"
  type: "Working Group"
  mail: "idr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/idr/"
  github: "yaroslavros/bgp-masque-tunnel"
  latest: "https://yaroslavros.github.io/bgp-masque-tunnel/draft-rosomakho-idr-bgp-masque-tunnel.html"

author:
 -
    fullname: Yaroslav Rosomakho
    organization: Zscaler
    email: yrosomakho@zscaler.com

normative:

informative:
  H1:
    =: RFC9112
    display: HTTP/1.1
  H2:
    =: RFC9113
    display: HTTP/2
  H3:
    =: RFC9114
    display: HTTP/3

--- abstract

This document defines BGP Tunnel Encapsulation Attribute tunnel types for
MASQUE CONNECT-TCP, CONNECT-UDP, CONNECT-IP, and CONNECT-ETHERNET. It also
defines URI Template and ALPN Sub-TLVs for advertising the MASQUE proxy endpoint,
the HTTP request target template, and the application-layer protocol constraints
used to establish the corresponding MASQUE tunnel.

--- middle

# Introduction

The BGP Tunnel Encapsulation Attribute
{{!BGP-TUNNEL-ENCAP-ATTR=RFC9012}} allows BGP speakers to advertise the
tunnel encapsulation information associated with reachability information
carried in BGP UPDATE messages. It is used to indicate that traffic associated
with a route can be carried using a particular tunnel encapsulation and to
provide the parameters needed to establish or use that tunnel.

MASQUE defines mechanisms for proxying traffic using HTTP. These mechanisms
include proxying UDP payloads using CONNECT-UDP {{!CONNECT-UDP=RFC9298}},
proxying IP packets using CONNECT-IP {{!CONNECT-IP=RFC9484}}, proxying TCP
connections using CONNECT-TCP {{!CONNECT-TCP=I-D.ietf-httpbis-connect-tcp}},
and proxying Ethernet frames using CONNECT-ETHERNET
{{!CONNECT-ETHERNET=I-D.ietf-masque-connect-ethernet}}. These mechanisms allow
traffic to be carried over HTTP/1.1 {{H1}}, HTTP/2 {{H2}}, or HTTP/3 {{H3}} connections to a MASQUE
proxy and are applicable to environments such as SD-WAN, data center
interconnect, VPN services, and other overlay connectivity deployments.

In some deployments, BGP is already used as the control plane for advertising
reachability and associated tunnel encapsulation information. Allowing BGP to
advertise MASQUE tunnel encapsulation parameters enables a BGP speaker to signal
that traffic associated with a route is reachable through a MASQUE proxy using
one of the CONNECT-* mechanisms. This document defines BGP Tunnel Encapsulation
Attribute tunnel types for CONNECT-TCP, CONNECT-UDP, CONNECT-IP, and
CONNECT-ETHERNET.

This document also defines a URI Template Sub-TLV for the BGP Tunnel
Encapsulation Attribute. The URI Template identifies the MASQUE proxy endpoint
and provides the template used to construct the HTTP request target for the
corresponding CONNECT-* request. In addition, because MASQUE tunnels can be
established over different HTTP versions, this document defines an ALPN Sub-TLV
that can be used to indicate the application-layer protocols supported or
preferred for use with the advertised tunnel.

This document does not define new BGP NLRI, does not define a new MASQUE
protocol mechanism, and does not define a new proxy authentication or
authorization mechanism. The NLRI to which the Tunnel Encapsulation Attribute is
attached identifies the traffic, service, or reachability information to which
the MASQUE tunnel applies. Authentication and authorization of the MASQUE proxy
remain the responsibility of the endpoints and the applicable HTTP and TLS mechanisms.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses terminology from {{BGP-TUNNEL-ENCAP-ATTR}},
{{CONNECT-UDP}}, {{CONNECT-IP}}, {{CONNECT-TCP}}, and
{{CONNECT-ETHERNET}}.

MASQUE proxy:
: An HTTP proxy that supports one or more of the MASQUE CONNECT mechanisms
  identified by the tunnel types defined in this document.

MASQUE tunnel:
: A tunnel established using one of the CONNECT mechanisms identified by the
  tunnel types defined in this document.

URI Template:
: A URI Template as defined by {{!URI-TEMPLATE=RFC6570}}. In this document, a URI
  Template is carried in the URI Template Sub-TLV and is used to construct the
  HTTP request target for a MASQUE tunnel.

ALPN:
: Application-Layer Protocol Negotiation, as defined by {{!ALPN=RFC7301}}.

# MASQUE Tunnel Encapsulation

A Tunnel Encapsulation TLV using one of the tunnel types defined in this
document identifies a MASQUE proxy and the corresponding HTTP request target
using the URI Template Sub-TLV defined in {{uri-template-sub-tlv}}. The URI
Template determines the MASQUE proxy endpoint and is used to construct the HTTP
request for the corresponding CONNECT mechanism.

An ALPN Sub-TLV, defined in {{alpn-sub-tlv}}, MAY be included to indicate the
application-layer protocols supported or preferred for use when connecting to
the MASQUE proxy.

The tunnel types defined in this section identify the MASQUE mechanism used to
carry the traffic associated with the BGP route. The NLRI to which the Tunnel
Encapsulation Attribute is attached determines the traffic, service, or
reachability information to which the MASQUE tunnel applies.

A Tunnel Encapsulation TLV whose tunnel type is one of the MASQUE tunnel types
defined in this document is referred to as a MASQUE Tunnel Encapsulation TLV.
A MASQUE Tunnel Encapsulation TLV MUST contain exactly one URI Template Sub-TLV
and MAY contain at most one ALPN Sub-TLV.

The following Sub-TLVs are not applicable to MASQUE Tunnel Encapsulation TLVs
and MUST NOT be included:

| Sub-TLV | Code |
| --- | --- |
| Encapsulation | 1 |
| Protocol Type | 2 |
| IPsec Tunnel Authenticator | 3 |
| Tunnel Egress Endpoint | 6 |
| UDP Destination Port | 8 |
| Embedded Label Handling | 9 |
| MPLS Label Stack | 10 |
| Prefix-SID | 11 |
| Preference | 12 |
| Binding SID | 13 |
| ENLP | 14 |
| Priority | 15 |
| SPI/SI Representation | 16 |
| SRv6 Binding SID | 20 |
{: #masque-prohibited-sub-tlvs title="Sub-TLVs not defined for use with MASQUE Tunnel Encapsulation TLVs"}

If a MASQUE Tunnel Encapsulation TLV contains a prohibited Sub-TLV, contains no
URI Template Sub-TLV, or contains more than one URI Template Sub-TLV, the MASQUE
Tunnel Encapsulation TLV MUST be ignored.

## CONNECT-TCP Tunnel Type

The CONNECT-TCP Tunnel Type indicates that traffic associated with the route is
to be carried using CONNECT-TCP {{CONNECT-TCP}}. This tunnel type is applicable
to routes or service-specific information that identify TCP connectivity or TCP
flow steering.

## CONNECT-UDP Tunnel Type

The CONNECT-UDP Tunnel Type indicates that traffic associated with the route is
to be carried using CONNECT-UDP {{CONNECT-UDP}}. This tunnel type is applicable
to routes or service-specific information that identify UDP connectivity or UDP
flow steering.

## CONNECT-IP Tunnel Type

The CONNECT-IP Tunnel Type indicates that traffic associated with the route is
to be carried using CONNECT-IP {{CONNECT-IP}}. This tunnel type is applicable to
routes that identify IP reachability, such as IP prefixes, VPN-IP routes, or
other service-specific IP reachability information.

## CONNECT-ETHERNET Tunnel Type

The CONNECT-ETHERNET Tunnel Type indicates that traffic associated with the
route is to be carried using CONNECT-ETHERNET {{CONNECT-ETHERNET}}. This tunnel
type is applicable to routes that identify Ethernet or Layer 2 service
reachability, such as EVPN or other service-specific Layer 2 reachability
information.

# URI Template Sub-TLV {#uri-template-sub-tlv}

The URI Template Sub-TLV identifies the MASQUE proxy endpoint and provides the
template used to construct the HTTP request target for the MASQUE tunnel. The
URI Template Sub-TLV is carried in a MASQUE Tunnel Encapsulation TLV.

The URI Template Sub-TLV has the following format:

~~~ ascii-art
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |   Type=TBD    |           Length              |               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               |
 ~                     URI Template Value                        ~
 |                                                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-uri-template-sub-tlv title="URI Template Sub-TLV"}

Type:
: TBD, to be assigned by IANA from the range 128-255.

Length:
: The length, in octets, of the URI Template Value field.

URI Template Value:
: A UTF-8 encoded URI Template {{URI-TEMPLATE}}.

The URI Template Value MUST be a syntactically valid URI Template. The URI
Template MUST be in absolute form and MUST include non-empty scheme, authority,
and path components. The URI Template MUST satisfy the URI Template requirements
of the MASQUE mechanism identified by the enclosing MASQUE Tunnel Encapsulation
TLV.

The authority component of the URI Template identifies the MASQUE proxy endpoint
to which the receiver establishes the HTTP connection. The path and query
components of the expanded URI identify the request target used for the
corresponding CONNECT mechanism.

If the URI Template Value is not a syntactically valid URI Template, if it is
not in absolute form, if it does not include non-empty scheme, authority, and
path components, or if it is otherwise not usable with the MASQUE mechanism
identified by the enclosing MASQUE Tunnel Encapsulation TLV, the MASQUE Tunnel
Encapsulation TLV MUST be ignored.

# ALPN Sub-TLV {#alpn-sub-tlv}

The ALPN Sub-TLV indicates the application-layer protocol or protocols that are
supported or preferred for use with the tunnel described by the enclosing Tunnel
Encapsulation TLV. When used with a MASQUE Tunnel Encapsulation TLV, the ALPN
Sub-TLV identifies the HTTP version or versions that can be used to establish
the corresponding MASQUE tunnel.

The ALPN Sub-TLV has the following format:

~~~ ascii-art
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |   Type=TBD    |           Length              |               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               |
 ~                       ALPN ProtocolNameList                   ~
 |                                                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-alpn-sub-tlv title="ALPN Sub-TLV"}

Type:
: TBD, to be assigned by IANA from the range 128-255.

Length:
: The length, in octets, of the ALPN ProtocolNameList field.

ALPN ProtocolNameList:
: A sequence of one or more ALPN protocol identifiers, encoded as a TLS ALPN
  `ProtocolNameList` as defined in {{Section 3.1 of ALPN}}.

The ALPN ProtocolNameList field contains one or more non-empty ALPN protocol
identifiers. The order of ALPN protocol identifiers indicates preference, with
the most preferred protocol listed first.

When the ALPN Sub-TLV is present in a MASQUE Tunnel Encapsulation TLV, the
receiver MUST use one of the advertised ALPN protocol identifiers when
establishing the HTTP connection to the MASQUE proxy. If none of the advertised
protocol identifiers is supported by the receiver, the MASQUE Tunnel
Encapsulation TLV MUST be ignored.

A MASQUE Tunnel Encapsulation TLV MUST contain at most one ALPN Sub-TLV. If more
than one ALPN Sub-TLV is present, the MASQUE Tunnel Encapsulation TLV MUST be
ignored.

If the ALPN Sub-TLV is malformed, including if it contains an empty
ProtocolNameList or an empty protocol identifier, the MASQUE Tunnel
Encapsulation TLV MUST be ignored.

# Use with BGP NLRI

The tunnel types and Sub-TLVs defined in this document are used with the BGP
Tunnel Encapsulation Attribute. This document does not define new BGP NLRI or
new procedures for associating tunnel encapsulation information with routes.

The NLRI to which the Tunnel Encapsulation Attribute is attached identifies the
traffic, service, or reachability information to which the MASQUE tunnel applies.
The tunnel type in the Tunnel Encapsulation TLV identifies the MASQUE mechanism
used to carry that traffic.

The following examples illustrate possible uses:

* A route that identifies IP reachability, such as an IP prefix or VPN-IP route,
  can use the CONNECT-IP Tunnel Type to indicate that matching IP packets are
  carried using CONNECT-IP.

* A route that identifies Ethernet or Layer 2 service reachability, such as an
  EVPN route, can use the CONNECT-ETHERNET Tunnel Type to indicate that matching
  Ethernet frames are carried using CONNECT-ETHERNET.

* A route or service-specific advertisement that identifies TCP connectivity or
  TCP flow steering can use the CONNECT-TCP Tunnel Type to indicate that matching
  TCP traffic is carried using CONNECT-TCP.

* A route or service-specific advertisement that identifies UDP connectivity or
  UDP flow steering can use the CONNECT-UDP Tunnel Type to indicate that matching
  UDP traffic is carried using CONNECT-UDP.

Specifications or deployment profiles that use the tunnel types defined in this
document MAY define additional rules for their use with particular AFI/SAFI
combinations or service models.

# Operational Considerations

The tunnel types and Sub-TLVs defined in this document allow BGP to advertise
MASQUE tunnel encapsulation parameters associated with BGP routes. Operators
MUST ensure that these attributes are propagated only within routing domains
where the advertised MASQUE proxy information is intended to be used.

A URI Template carried in BGP can reveal information about proxy names, service
structure, or internal topology. Operators MUST apply BGP import and export
policy to avoid leaking MASQUE tunnel encapsulation information outside the
intended administrative scope.

Multiple MASQUE tunnel candidates can be advertised by including multiple Tunnel
Encapsulation TLVs, each containing a single URI Template Sub-TLV. This can be
used to advertise alternative MASQUE proxies. Selection among available tunnel
candidates follows the procedures and local policy used for the BGP Tunnel
Encapsulation Attribute.

Operators SHOULD consider the size and stability of URI Template values when
attaching MASQUE Tunnel Encapsulation TLVs to routes. Large URI Templates, or
frequent changes to URI Templates, can increase BGP UPDATE size and route churn.
Deployments that advertise the same MASQUE proxy parameters for many routes
SHOULD consider existing BGP mechanisms and service-specific profiles that avoid
unnecessary repetition.

# Security Considerations

This document defines BGP signaling for MASQUE tunnel encapsulation parameters.
It does not define a new authentication or authorization mechanism for MASQUE
proxies, and it does not change the security properties of the MASQUE
mechanisms identified by the tunnel types defined in this document.

Authorization to use the MASQUE proxy, including authorization to reach the
requested target or to carry the traffic associated with the route, is enforced
by the MASQUE proxy and the applicable mechanisms. Implementations MUST NOT
treat possession of a BGP route or Tunnel Encapsulation Attribute as sufficient
authorization to use a MASQUE proxy.

Misconfiguration, route leaks, or malicious injection of BGP routes carrying
MASQUE tunnel encapsulation information can cause traffic to be directed to an
unintended MASQUE proxy. This can result in traffic interception, denial of
service, policy bypass, or loss of connectivity. Operators should apply the same
route filtering, origin validation, session protection, and import/export policy
controls used for other BGP routes carrying tunnel encapsulation information.

The URI Template Sub-TLV can reveal information about proxy hostnames, service
structure, tenant identifiers, or internal topology. Such information can be
sensitive. Operators should ensure that routes carrying MASQUE Tunnel
Encapsulation TLVs are distributed only within the administrative scope where
the corresponding MASQUE proxy information is intended to be visible.

The URI Template carried in BGP is used to construct an HTTP request target.
Implementations MUST validate the URI Template before use and MUST apply the
processing rules in this document and in the applicable MASQUE specification.
Implementations MUST NOT expand or use a URI Template in a way that creates a
request target outside the scope intended by the received route and local
policy.

The ALPN Sub-TLV can constrain the application-layer protocols used to establish
a MASQUE tunnel. If an ALPN Sub-TLV is present, receivers MUST use only one of
the advertised ALPN protocol identifiers for the corresponding tunnel. Ignoring
an ALPN constraint could cause a receiver to use an HTTP version or transport
that the advertising speaker did not intend to support for that tunnel.

The security considerations of {{BGP-TUNNEL-ENCAP-ATTR}}, {{CONNECT-UDP}},
{{CONNECT-IP}}, {{CONNECT-TCP}}, {{CONNECT-ETHERNET}}, {{URI-TEMPLATE}}, and
{{ALPN}} apply.

# IANA Considerations

IANA is requested to update the "BGP Tunnel Encapsulation" registry group as
specified in the following sections.

## BGP Tunnel Encapsulation Attribute Tunnel Types

IANA is requested to allocate the following values from the "BGP Tunnel
Encapsulation Attribute Tunnel Types" registry:

| Value | Description | Reference |
|---:|---|---|
| TBD1 | CONNECT-TCP | this document |
| TBD2 | CONNECT-UDP | this document |
| TBD3 | CONNECT-IP | this document |
| TBD4 | CONNECT-ETHERNET | this document |
{: #masque-tunnel-types title="MASQUE BGP Tunnel Encapsulation Attribute Tunnel Types"}

## BGP Tunnel Encapsulation Attribute Sub-TLVs

IANA is requested to allocate the following values from the "BGP Tunnel
Encapsulation Attribute Sub-TLVs" registry:

| Value | Description | Reference |
|---:|---|---|
| TBD5, from the 128-255 range | URI Template | this document |
| TBD6, from the 128-255 range | ALPN | this document |
{: #masque-tunnel-subtlvs title="MASQUE BGP Tunnel Encapsulation Attribute Sub-TLVs"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
