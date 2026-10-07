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
 -
    fullname: Alvaro Retana
    organization: Futurewei Technologies, Inc.
    email: aretana@futurewei.com


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
defines URI Template and SVCB Parameters Sub-TLVs for advertising the MASQUE
proxy endpoint, the HTTP request target template, and extensible connection
parameters used to establish the corresponding MASQUE tunnel.

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
one of the template-driven CONNECT mechanisms. This document defines BGP Tunnel Encapsulation
Attribute tunnel types for CONNECT-TCP, CONNECT-UDP, CONNECT-IP, and
CONNECT-ETHERNET.

This document also defines a URI Template Sub-TLV for the BGP Tunnel
Encapsulation Attribute. The URI Template identifies the MASQUE proxy endpoint
and provides the template used to construct the HTTP request target for the
corresponding CONNECT request. This document also defines an SVCB Parameters
Sub-TLV that reuses the service parameter encoding and registry of SVCB and
HTTPS resource records {{!SVCB=RFC9460}}. It carries ALPN identifiers and other
connection parameters as a single parameter set. These parameters are
advertised through BGP and need not correspond to a DNS resource record
published for the MASQUE proxy.

This document does not define new BGP NLRI, does not define a new MASQUE
protocol mechanism, and does not define a new proxy authentication or
authorization mechanism. The NLRI to which the Tunnel Encapsulation Attribute is
attached identifies the traffic, service, or reachability information to which
the MASQUE tunnel applies. Authentication and authorization of the MASQUE proxy
remain the responsibility of the endpoints and the applicable HTTP and TLS mechanisms.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses terminology from {{BGP-TUNNEL-ENCAP-ATTR}},
{{CONNECT-UDP}}, {{CONNECT-IP}}, {{CONNECT-TCP}},
{{CONNECT-ETHERNET}}, and {{SVCB}}.

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

SvcParam:
: A service parameter consisting of a SvcParamKey and a SvcParamValue, as
  defined by {{SVCB}}.

# MASQUE Tunnel Encapsulation

A Tunnel Encapsulation TLV using one of the tunnel types defined in this
document identifies a MASQUE proxy and the corresponding HTTP request target
using the URI Template Sub-TLV defined in {{uri-template-sub-tlv}}. The URI
Template determines the MASQUE proxy endpoint and is used to construct the HTTP
request for the corresponding CONNECT mechanism.

An SVCB Parameters Sub-TLV, defined in {{svcb-parameters-sub-tlv}}, MAY be
included to supply connection parameters for that proxy, including supported
application-layer protocols, an alternative port, and IP address hints.

The tunnel types defined in this section identify the MASQUE mechanism used to
carry the traffic associated with the BGP route. The NLRI to which the Tunnel
Encapsulation Attribute is attached determines the traffic, service, or
reachability information to which the MASQUE tunnel applies.

A Tunnel Encapsulation TLV whose tunnel type is one of the MASQUE tunnel types
defined in this document is referred to as a MASQUE Tunnel Encapsulation TLV.
A MASQUE Tunnel Encapsulation TLV MUST contain exactly one URI Template Sub-TLV
and MAY contain at most one SVCB Parameters Sub-TLV. Support for the SVCB
Parameters Sub-TLV is OPTIONAL.

Only the following Sub-TLVs are applicable to MASQUE Tunnel Encapsulation TLVs:

| Sub-TLV | Code |
| --- | --- |
| Color | 4 |
| DS Field | 7 |
| URI Template | TBD5 |
| SVCB Parameters | TBD6 |
{: #masque-allowed-sub-tlvs title="Sub-TLVs allowed for use with MASQUE Tunnel Encapsulation TLVs"}

All other Sub-TLVs not explicitly listed above are not defined for use with
MASQUE Tunnel Encapsulation TLVs. Receivers MUST ignore these Sub-TLVs when
validating MASQUE-specific semantics. Future specifications MAY define
additional Sub-TLVs for use with MASQUE Tunnel Encapsulation TLVs.

If a MASQUE Tunnel Encapsulation TLV contains no
URI Template Sub-TLV, or contains more than one URI Template Sub-TLV,
it MUST be ignored.

## CONNECT-TCP Tunnel Type

The CONNECT-TCP Tunnel Type indicates that traffic associated with the route is
to be carried using CONNECT-TCP {{CONNECT-TCP}}. This tunnel type is applicable
to routes or service-specific information that identify TCP connectivity or TCP
flow steering.

> Name: MASQUE CONNECT-TCP Tunnel
> Type: TBD1
> Length: 0

## CONNECT-UDP Tunnel Type

The CONNECT-UDP Tunnel Type indicates that traffic associated with the route is
to be carried using CONNECT-UDP {{CONNECT-UDP}}. This tunnel type is applicable
to routes or service-specific information that identify UDP connectivity or UDP
flow steering.

> Name: MASQUE CONNECT-UDP Tunnel
> Type: TBD2
> Length: 0

## CONNECT-IP Tunnel Type

The CONNECT-IP Tunnel Type indicates that traffic associated with the route is
to be carried using CONNECT-IP {{CONNECT-IP}}. This tunnel type is applicable to
routes that identify IP reachability, such as IP prefixes, VPN-IP routes, or
other service-specific IP reachability information.

> Name: MASQUE CONNECT-IP Tunnel
> Type: TBD3
> Length: 0

## CONNECT-ETHERNET Tunnel Type

The CONNECT-ETHERNET Tunnel Type indicates that traffic associated with the
route is to be carried using CONNECT-ETHERNET {{CONNECT-ETHERNET}}. This tunnel
type is applicable to routes that identify Ethernet or Layer 2 service
reachability, such as EVPN or other service-specific Layer 2 reachability
information.

> Name: MASQUE CONNECT-ETHERNET Tunnel
> Type: TBD4
> Length: 0

# URI Template Sub-TLV {#uri-template-sub-tlv}

The URI Template Sub-TLV identifies the MASQUE proxy endpoint and provides the
template used to construct the HTTP request target for the MASQUE tunnel. The
URI Template Sub-TLV is carried in a MASQUE Tunnel Encapsulation TLV.

The URI Template Sub-TLV has the following format:

~~~ ascii-art
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |   Type=TBD5   |           Length              |               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               |
 ~                     URI Template Value                        ~
 |                                                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-uri-template-sub-tlv title="URI Template Sub-TLV"}

Type:
: TBD5

Length:
: The length, in octets, of the URI Template Value field.

URI Template Value:
: A UTF-8 encoded URI Template {{URI-TEMPLATE}}.

The URI Template Value MUST be a syntactically valid URI Template. The URI
Template MUST be in absolute form and MUST include non-empty scheme, authority,
and path components. The URI Template MUST satisfy the URI Template requirements
of the MASQUE mechanism identified by the enclosing MASQUE Tunnel Encapsulation
TLV.

The authority component of the URI Template identifies the MASQUE proxy endpoint.
An SVCB Parameters Sub-TLV can supply connection parameters for that endpoint
without changing the authority used in the HTTP request or the identity used
to authenticate the proxy. The path and query components of the expanded URI
identify the request target used for the corresponding CONNECT mechanism.

If the URI Template Value is not a syntactically valid URI Template, if it is
not in absolute form, if it does not include non-empty scheme, authority, and
path components, or if it is otherwise not usable with the MASQUE mechanism
identified by the enclosing MASQUE Tunnel Encapsulation TLV, the MASQUE Tunnel
Encapsulation TLV MUST be ignored.

The Tunnel Egress Endpoint Sub-TLV defined by {{BGP-TUNNEL-ENCAP-ATTR}} is not used with MASQUE
Tunnel Encapsulation TLVs because the authority component of the URI Template
identifies the MASQUE proxy endpoint. Connection parameters can be supplied by
the SVCB Parameters Sub-TLV.

# SVCB Parameters Sub-TLV {#svcb-parameters-sub-tlv}

The SVCB Parameters Sub-TLV carries connection parameters for the MASQUE proxy
identified by the URI Template in the same MASQUE Tunnel Encapsulation TLV.
Its value uses the SvcParams wire encoding defined in {{Section 2.2 of SVCB}}
and the "Service Parameter Keys (SvcParamKeys)" registry defined by {{SVCB}}.

Only the SvcParams portion is carried. The value does not include SvcPriority,
TargetName, a DNS resource record header, or a DNS message. The URI Template
supplies the proxy identity, and selection among alternative tunnels remains
subject to local policy. SVCB AliasMode and DNS alias discovery are not part of
this Sub-TLV.

The SVCB Parameters Sub-TLV has the following format:

~~~ ascii-art
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |   Type=TBD6   |           Length              |               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               |
 ~                          SvcParams                            ~
 |                                                               |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #fig-svcb-parameters-sub-tlv title="SVCB Parameters Sub-TLV"}

Type:
: TBD6

Length:
: The length, in octets, of the SvcParams field, encoded as a two-octet
  unsigned integer in network byte order.

SvcParams:
: A sequence of one or more SvcParams. Each consists of a two-octet
  SvcParamKey, a two-octet length of the SvcParamValue, and that number of
  value octets. The key and length are unsigned integers in network byte
  order. Values use the wire encoding specified for their respective keys.

The encoding and validation rules in {{Section 2.2 of SVCB}} apply to the
SvcParams field. An empty SvcParams field is invalid; an advertiser with no
parameters to signal omits the Sub-TLV.

## Parameter Processing

The SvcParams form a single parameter set associated with the enclosing
MASQUE Tunnel Encapsulation TLV. They need not match any SVCB or HTTPS DNS
record, and no such DNS record needs to exist. Use of the advertised parameters
is optional. A receiver can instead use normal connection establishment
procedures for the URI Template. Selection between BGP-advertised parameters
and information obtained through other mechanisms, including DNS HTTPS records,
is a matter of local policy. Each parameter set is evaluated independently.

Receivers that use the parameter set MUST apply the service parameter semantics
and compatibility rules in Sections 7 and 8 of {{SVCB}}, with the HTTPS mapping
in {{Section 9 of SVCB}}. This includes the applicable keys, default ALPN set,
and automatically mandatory keys defined by that mapping. Additional parameters
applicable to HTTPS can be used according to their defining specifications.

For this purpose, the URI Template identifies the service being accessed, and
its host supplies the effective TargetName. If the host is an IP literal,
that address is used directly and address hints are not applicable. The URI
Template continues to determine the HTTP authority and the identity against
which the receiver authenticates the MASQUE proxy, including when a port
override or address hint is used.

The negotiated HTTP protocol and capabilities must support the selected MASQUE
mechanism, as required by its specification.

## Error Handling

If more than one SVCB Parameters Sub-TLV is present, or if its value is
malformed, not self-consistent, or incompatible as defined by {{SVCB}}, the
receiver MUST ignore the SVCB Parameters Sub-TLVs in that MASQUE Tunnel
Encapsulation TLV. This does not by itself make the enclosing TLV unusable;
the receiver can use normal connection establishment procedures for the URI
Template. Framing errors in the enclosing BGP attribute remain subject to
{{Section 13 of BGP-TUNNEL-ENCAP-ATTR}}.

## Example

A CONNECT-UDP tunnel can advertise the URI Template
`https://proxy.example.org/masque/udp/{target_host}/{target_port}/` together with
these SvcParams, shown in presentation format for readability:

~~~
alpn=h3,h2 no-default-alpn port=8443
~~~

The SvcParams field is the following 20 octets in hexadecimal:

~~~
00 01 00 06 02 68 33 02 68 32
00 02 00 00
00 03 00 02 20 fb
~~~

A receiver using this parameter set can attempt HTTP/3 over QUIC to UDP port
8443 or HTTP/2 over TLS to TCP port 8443 of `proxy.example.org`, following the
ALPN procedures in
{{Section 7.1.2 of SVCB}}. The HTTP authority remains `proxy.example.org`, and
the proxy is authenticated as `proxy.example.org`. The parameter set can be
used even if DNS publishes different HTTPS parameters or publishes no HTTPS
record for that name.

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
  can use the CONNECT-IP, CONNECT-TCP, or CONNECT-UDP Tunnel Types to indicate that matching IP, TCP, or UDP traffic packets are
  carried using the correspondind CONNECT mechanism.

* A route that identifies Ethernet or Layer 2 service reachability, such as an
  EVPN route, can use the CONNECT-ETHERNET Tunnel Type to indicate that matching
  Ethernet frames are carried using CONNECT-ETHERNET.

Specifications or deployment profiles that use the tunnel types defined in this
document may define additional rules for their use with particular AFI/SAFI
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
candidates is left to local policy and outside the scope of this specification.

A BGP Tunnel Encapsulation Attribute MAY contain both MASQUE Tunnel Encapsulation
TLVs and Tunnel Encapsulation TLVs of other tunnel types defined by {{BGP-TUNNEL-ENCAP-ATTR}} or
subsequent specifications. This document does not define any preference between
MASQUE and non-MASQUE tunnel types. Selection among available tunnel types is
determined by local policy and the procedures applicable to the associated
AFI/SAFI.

Operators should consider the stability of URI Template and SVCB parameter
values when attaching MASQUE Tunnel Encapsulation TLVs to routes. Frequent
changes to these values can increase route churn. SVCB parameters advertised
in BGP have no DNS TTL; their availability for new tunnel establishment follows
the corresponding BGP advertisement and its replacement or withdrawal.
Deployments that advertise the same MASQUE proxy parameters for many routes
should consider existing BGP mechanisms and service-specific profiles that avoid
unnecessary repetition.

## Parameter Length

{{URI-TEMPLATE}} does not define a general maximum length for a URI Template.
When a URI Template is carried in the URI Template Sub-TLV, the length of the
URI Template Value is constrained by the Sub-TLV encoding and by the size of the
BGP UPDATE message in which the Sub-TLV is carried.

The URI Template Sub-TLV uses a two-octet Length field. This allows the URI
Template Value to be longer than 255 octets. However, long URI Template Values
can significantly increase the size of the enclosing BGP UPDATE message and
can affect propagation across BGP sessions where BGP Extended Messages
{{?BGP-EXTENDED-MESSAGES=RFC8654}} are not used.

URI Template Values SHOULD NOT exceed 1024 octets and should be kept as short as practical.

Deployments that carry URI Template Sub-TLVs SHOULD use BGP Extended Messages
{{BGP-EXTENDED-MESSAGES}} on the BGP sessions over which the
corresponding routes are propagated. If BGP Extended Messages are not available, URI Template
Values MUST be kept small enough that the complete BGP UPDATE message,
including all path attributes, NLRI, and protocol overhead, does not exceed
4096 octets. These size considerations also apply to SVCB Parameters Sub-TLVs.
Advertisers MUST account for their combined size with the URI Template and all
other contents of the UPDATE, even when each Sub-TLV fits its own Length field.
If an SVCB Parameters Sub-TLV exceeds a receiver's supported or configured
limit, the receiver SHOULD ignore that Sub-TLV. A truncated parameter set
MUST NOT be used.

If the URI Template Value exceeds the receiver's supported or configured
limit, the receiver SHOULD ignore the enclosing MASQUE Tunnel Encapsulation TLV.

If the length of the URI Template results in the BGP UPDATE exceeding the
maximum message size, the result may include a lack of reachability, as detailed
in {{BGP-EXTENDED-MESSAGES}}. If BGP Extended Messages are partially supported,
the BGP Tunnel Encapsulation Attribute may be discarded {{BGP-EXTENDED-MESSAGES}},
as it is eligible under the "attribute discard" approach of
{{?BGP-ERROR-HANDLING=RFC7606}}, which would result in the information
not reaching the intended recipients.
# Security Considerations

The security considerations of {{BGP-TUNNEL-ENCAP-ATTR}}, {{CONNECT-UDP}},
{{CONNECT-IP}}, {{CONNECT-TCP}}, {{CONNECT-ETHERNET}}, {{URI-TEMPLATE}}, and
{{ALPN}} apply. The security considerations for service parameters in
{{SVCB}} also apply; carrying them in BGP does not provide DNSSEC validation.

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
For more details, see Section 11 of {{BGP-TUNNEL-ENCAP-ATTR}}.

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

The SVCB Parameters Sub-TLV can change the transport protocol, destination
port, and candidate addresses used to establish a MASQUE tunnel. Receivers
MUST apply local connection policy to the resulting endpoint and MUST validate
the proxy identity determined by the URI Template. BGP-supplied parameters
MUST NOT be treated as proof of that identity or as a reason to bypass TLS
certificate validation. Both URI Templates and SVCB parameters can expose
internal service configuration and topology.

Because use of the SVCB Parameters Sub-TLV is optional, advertisers cannot
rely on it to enforce connection policy. Such policy must be enforced by the
MASQUE endpoints and their local configuration.

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

IANA is requested to allocate the following values from the 192-252 range in the "BGP Tunnel
Encapsulation Attribute Sub-TLVs" registry:

| Value | Description | Reference |
|---:|---|---|
| TBD5 | URI Template | this document |
| TBD6 | SVCB Parameters | this document |
{: #masque-tunnel-subtlvs title="MASQUE BGP Tunnel Encapsulation Attribute Sub-TLVs"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
