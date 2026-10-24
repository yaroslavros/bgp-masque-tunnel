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
HTTPS resource records {{!SVCB=RFC9460}}. It carries IP address hints for
validation of the advertised proxy endpoint without a DNS lookup, together
with ALPN identifiers and other connection parameters.

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

# MASQUE Tunnel Encapsulation {#masque-tunnel-encapsulation}

A Tunnel Encapsulation TLV using one of the tunnel types defined in this
document identifies a MASQUE proxy and the corresponding HTTP request target
using the URI Template Sub-TLV defined in {{uri-template-sub-tlv}}. The URI
Template determines the MASQUE proxy endpoint and is used to construct the HTTP
request for the corresponding CONNECT mechanism.

The SVCB Parameters Sub-TLV, defined in {{svcb-parameters-sub-tlv}}, supplies
IP address hints for BGP validation of the proxy endpoint and can also carry
connection parameters such as supported application-layer protocols and an
alternative port.

The tunnel types defined in this section identify the MASQUE mechanism used to
carry the traffic associated with the BGP route. The NLRI to which the Tunnel
Encapsulation Attribute is attached determines the traffic, service, or
reachability information to which the MASQUE tunnel applies.

A Tunnel Encapsulation TLV whose tunnel type is one of the MASQUE tunnel types
defined in this document is referred to as a MASQUE Tunnel Encapsulation TLV.
A MASQUE Tunnel Encapsulation TLV MUST contain exactly one URI Template Sub-TLV
and MUST NOT contain more than one SVCB Parameters Sub-TLV. Unless the host
component of the URI Template is an IPv4 or IPv6 address literal, the TLV MUST
contain exactly one SVCB Parameters Sub-TLV containing at least one address
in `ipv4hint`, `ipv6hint`, or both. If the host is such an address literal,
the SVCB Parameters Sub-TLV MAY be omitted.

A MASQUE Tunnel Encapsulation TLV that does not meet these requirements MUST be handled as specified in {{masque-error-handling}}.
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
Encapsulation TLV MUST be handled as specified in {{masque-error-handling}}.

For the MASQUE tunnel types defined in this document, the endpoint address
requirements in {{masque-tunnel-encapsulation}} replace the need for a Tunnel
Egress Endpoint Sub-TLV in {{BGP-TUNNEL-ENCAP-ATTR}}. The address is supplied
by the URI Template when its host is an IPv4 or IPv6 address literal, or by
the IP address hints in the SVCB Parameters Sub-TLV otherwise.

# SVCB Parameters Sub-TLV {#svcb-parameters-sub-tlv}

The SVCB Parameters Sub-TLV carries connection parameters for the MASQUE proxy
identified by the URI Template in the same MASQUE Tunnel Encapsulation TLV.
The Sub-TLV Value contains the SvcParams sequence encoded as specified in
{{Section 2.2 of SVCB}}, using the "Service Parameter Keys (SvcParamKeys)"
registry defined by {{SVCB}}.

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
: The length, in octets, of the SvcParams field.

SvcParams:
: A sequence of one or more SvcParams. Each consists of a two-octet
  SvcParamKey, a two-octet length of the SvcParamValue, and that number of
  value octets. The key and length are unsigned integers in network byte
  order. Values use the wire encoding specified for their respective keys.

The encoding and self-consistency rules of {{SVCB}} apply to the SvcParams
field. An empty SvcParams field is invalid.

Validation of BGP parameters is independent of the receiving
speaker's ability to establish a MASQUE tunnel.  Lack of support for
a connection parameter or an advertised application-layer protocol
does not, by itself, invalidate the BGP advertisement.
## Endpoint Validation {#endpoint-validation}

The URI Template's address literal and any addresses in `ipv4hint` or
`ipv6hint` provide endpoint address information that can be validated without
a DNS lookup. The address validity rules in
{{Section 3.1 of BGP-TUNNEL-ENCAP-ATTR}} apply to these addresses, including its
configuration exceptions and treatment of valid but unreachable addresses.

A receiver MAY apply the procedure in
{{Section 3.1.1 of BGP-TUNNEL-ENCAP-ATTR}} to check whether these addresses
belong to the AS that originated the route.
When used, its configuration and revalidation rules
apply. If present, the URI Template's address literal
MUST be the address used for validation, and any
addresses in ipv4hint or ipv6hint SHOULD be ignored. The selection of which IP address hints should be
validated, if any, is left to the implementation. The
validation of more than one address is out of scope for this document.
An address found invalid is handled as specified in
{{masque-error-handling}}.

## Use of Connection Parameters

Service parameter semantics and compatibility are defined in Sections 7 and 8
of {{SVCB}}, with the HTTPS mapping in {{Section 9 of SVCB}}. The URI Template
identifies the service being accessed, and its host supplies the effective
TargetName. For an IPv4 or IPv6 address literal, that address is used directly.

The address information required for BGP validation does not constrain the
addresses used for connection establishment. {{Section 7.3 of SVCB}} describes
the use of address hints and the preference for addresses obtained from A and
AAAA records. Procedures for establishing the MASQUE tunnel, including address
resolution and protocol negotiation, are outside the scope of this document
and follow the applicable HTTP and MASQUE specifications.

## Error Handling {#masque-error-handling}

A MASQUE Tunnel Encapsulation TLV is invalid if it does not contain exactly one
URI Template Sub-TLV with a valid value, contains more than one SVCB Parameters
Sub-TLV, or contains an SVCB Parameters Sub-TLV whose value is malformed or
not self-consistent. It is also invalid if it lacks the address information
required by {{masque-tunnel-encapsulation}} or any advertised endpoint address
is found invalid under {{endpoint-validation}}. The optional AS-ownership
procedure can cause rejection only when it is applied.
Unknown SvcParamKeys do not by themselves make a Sub-TLV malformed; their
connection-establishment semantics are handled according to {{SVCB}}.

An invalid MASQUE Tunnel Encapsulation TLV MUST be treated as a TLV containing
a malformed Tunnel Egress Endpoint Sub-TLV under
{{Section 13 of BGP-TUNNEL-ENCAP-ATTR}}: the entire TLV MUST be ignored and
removed from the attribute before propagation. The receiver MUST NOT retain
such a TLV by discarding only the SVCB Parameters Sub-TLV or an invalid hint
address. The remaining validation and error-handling rules of that section
apply, including treat-as-withdraw if no valid TLVs remain and the handling of
framing errors in the enclosing attribute.

## Example

A CONNECT-UDP tunnel can advertise the URI Template
`https://proxy.example.org/masque/udp/{target_host}/{target_port}/` together with
these SvcParams, shown in presentation format for readability:

~~~
alpn=h3,h2 no-default-alpn port=8443
ipv4hint=192.0.2.1 ipv6hint=2001:db8::1
~~~

The SvcParams field is the following 48 octets in hexadecimal:

~~~
00 01 00 06 02 68 33 02 68 32
00 02 00 00
00 03 00 02 20 fb
00 04 00 04 c0 00 02 01
00 06 00 10 20 01 0d b8 00 00 00 00 00 00 00 00 00 00 00 01
~~~

Both hint addresses are available for validation as described in
{{endpoint-validation}}. These documentation addresses illustrate the encoding;
a deployment uses actual proxy addresses.

The connection parameters advertise HTTP/3 over QUIC to UDP port 8443 and
HTTP/2 over TLS to TCP port 8443 of `proxy.example.org`. Address selection
follows the applicable connection-establishment procedures. The HTTP authority
and authenticated proxy identity remain `proxy.example.org`.

If the URI Template instead uses an IPv4 or IPv6 address literal as its host,
the literal supplies the address for BGP validation and the SVCB Parameters
Sub-TLV can be omitted.

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
Encapsulation TLVs, each containing a single URI Template Sub-TLV and the
address information required by {{masque-tunnel-encapsulation}}. This can be used to
advertise alternative MASQUE proxies. Selection among available tunnel
candidates is left to local policy and outside the scope of this specification.

A BGP Tunnel Encapsulation Attribute MAY contain both MASQUE Tunnel Encapsulation
TLVs and Tunnel Encapsulation TLVs of other tunnel types defined by {{BGP-TUNNEL-ENCAP-ATTR}} or
subsequent specifications. This document does not define any preference between
MASQUE and non-MASQUE tunnel types. Selection among available tunnel types is
determined by local policy and the procedures applicable to the associated
AFI/SAFI.

Operators should consider the stability of URI Template and SVCB parameter
values when attaching MASQUE Tunnel Encapsulation TLVs to routes. Frequent
changes to these values can increase route churn.
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
limit, the receiver SHOULD ignore the enclosing MASQUE Tunnel Encapsulation
TLV. A truncated parameter set MUST NOT be used, and the receiver MUST NOT
bypass the endpoint validation requirements by discarding the Sub-TLV.

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
{{CONNECT-IP}}, {{CONNECT-TCP}}, {{CONNECT-ETHERNET}}, {{URI-TEMPLATE}},
{{ALPN}}, and {{SVCB}} apply.

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

Endpoint address validation, when performed, checks the addresses carried
in BGP. It does not validate a different address subsequently obtained through
DNS or authenticate the MASQUE proxy. Connection policy and
proxy authentication remain the responsibility of the MASQUE endpoints.

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
