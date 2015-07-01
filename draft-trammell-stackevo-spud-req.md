---
title: Requirements for the design of a Substrate Protocol for User Datagrams (SPUD)
abbrev: SPUD requirements
docname: draft-trammell-stackevo-spud-req-latest
date: 2015-5-20
category: info
ipr: trust200902


author:
  -
    ins: B. Trammell
    name: Brian Trammell
    role: editor
    org: ETH Zurich
    email: ietf@trammell.ch
    street: Gloriastrasse 35
    city: 8092 Zurich
    country: Switzerland

informative:
  RFC6347:
  I-D.hardie-spud-use-cases:
  I-D.hildebrand-spud-prototype:
  I-D.huitema-tls-dtls-as-subtransport:
  I-D.trammell-stackevo-newtea:
  I-D.trammell-semi-report:
  I-D.ietf-rtcweb-data-channel:

--- abstract

The Substrate Protocol for User Datagrams (SPUD) BoF session at the IETF 92 meeting in Dallas in March 2015 identified the potential need for a UDP-based encapsulation protocol to allow explicit cooperation with middleboxes while using new, encrypted transport protocols. This document defines a set of requirements for this protocol.

--- middle

## Motivation

A number of efforts to create new transport protocols or experiment with new
network behaviors have been built on top of UDP, as it traverses firewalls and
other middleboxes more commonly than new protocols do.  Each such effort must,
however, either manage its flows within common middlebox assumptions for UDP
or train the  middleboxes on the new protocol (thus losing the benefit of
using UDP). A common Substrate Protocol for User Datagrams (SPUD) would allow each effort
to re-use a set of shared methods for notifying middleboxes of the flows'
semantics, thus avoiding both the limitations of current flow semantics and
the need to re-invent the mechanism for notifying the middlebox of the new
semantics.

As a concrete example, it is common for some middleboxes to tear down required
state (such as NAT bindings) very rapidly for UDP flows. By notifying the
path that a particular transport using UDP maintains session state and explicitly
signals session start and stop using the substrate, the using protocol may reduce or avoid
the need for heartbeat traffic.

This document defines a specific set of requirements for a SPUD protocol,
based on analysis on a target set of applications to be developed on SPUD
developing experience with a prototype described in
{{I-D.hildebrand-spud-prototype}}. It is intended as the basis for eventually
chartering a working group for the further development of a SPUD protocol.


# History

An outcome of the IAB workshop on Stack Evolution in a Middlebox
Internet (SEMI) {{I-D.trammell-semi-report}}, held in Zurich in January 2015,
was a discussion on the creation of a substrate protocol to support the
deployment of new transport protocols in the Internet. Assuming that a way
forward for transport evolution in user space would involve encapsulation in
UDP datagrams, the workshop noted that it may be useful to have a
facility built atop UDP to provide minimal signaling of the semantics of a
flow that would otherwise be available in TCP. At the very least, indications
of first and last packets in a flow may assist firewalls and NATs in policy
decision and state maintenance. Further transport semantics would be used by
the protocol running atop this facility, but would only be visible to the
endpoints, as the transport protocol headers themselves would be encrypted,
along with the payload, to prevent inspection or modification.  This encryption
might be accomplished by using DTLS {{RFC6347}} as a subtransport {{I-D.huitema-tls-dtls-as-subtransport}}
or by other suitable methods. This facility could also provide minimal
application-to-path and path-to-application signaling, though there was
less agreement about what should or could be signaled here.

The Substrate Protocol for User Datagrams (SPUD) BoF was held at IETF 92 in Dallas in
March 2015 to develop this concept further. It is clear from discussion before and
during the SPUD BoF that any selective exposure of traffic metadata outside a
relatively restricted trust domain must be advisory, non-negotiated, and declarative
rather than imperative. This conclusion matches experience with previous
endpoint-to-middle and middle-to-endpoint signaling approaches. As with other
metadata systems, exposure of specific elements must be carefully assessed for
privacy risks and the total of exposed elements must be so assessed.  Each exposed parameter
should also be independently verifiable, so that each entity can assign its own trust
to other entities. Basic transport over the substrate must continue working even if
signaling is ignored or stripped, to support incremental deployment. These restrictions
on vocabulary are discussed further in {{I-D.trammell-stackevo-newtea}}. This discussion includes
privacy and trust concerns as well as the need for strong incentives for middlebox cooperation based
on the information that are exposed.

# Terminology

This document uses the following terms

Overtransport: : A transport protocol that uses SPUD for middlebox signaling and traversal.

Endpoint: : A node that sends or receives a packet. In this document, this term may refer to either the SPUD implementation on this node, the overtransport implementation on this node, or the applications running over that overtransport.

Path: : The set of Internet Protocol nodes a given packet traverses from endpoint to endpoint.

Middlebox: : A device on the path that makes decisions about forwarding behavior based on other than IP or sub-IP addressing information, and/or that modifies the packet before forwarding.

# Use Cases

[EDITOR'S NOTE: specific applications we think need this go here? reference draft-kuehlewind-spud-use-cases

# Functional Requirements

The following requirements detail the services that SPUD must provide to overtransports, endpoints, and middleboxes using SPUD.

## Grouping of Packets

Transport semantics and many properties of communication that endpoints may want to expose to middleboxes are bound to flows or groups of flows. SPUD must therefore provide a basic facility for associating packets together (into what we call a "tube" ,for lack of a better term) and associate information to these groups of packets. The simplest mechanisms for association would involve the addition of an identifier to each packet in a tube; current thoughts on the tradeoffs on requirements and constraints on this identifier space are given in {{tradeoffs-in-tube-identifiers}}.

## Endpoint to Path Signaling

SPUD must be able to provide information from the end-point(s) to all SPUD-aware nodes on the path. To be able to potentially communicate with all SPUD-aware middleboxes on the path SPUD must either be designed as an in-band signaling protocol, or there must be a pre-existing relationship between the endpoint and the SPUD-aware middleboxes along the path. Since it is implausible that an endpoint has these relationships to all SPUD-aware middleboxes on a certain path in the context of the Internet, SPUD must provide in-band signaling. SPUD may in addition also offer mechanisms for out-of-band signaling when it is appropriate to use.

## Extensibility

SPUD must enable multiple new transport semantics without requiring updates to SPUD implementations in middleboxes.

## Authentication

The basic SPUD protocol must not require any authentication or a priori trust relationship between endpoints and middleboxes to function.  However, SPUD should support the presentation/exchange of authentication information in environments where a trust relationship already exists, or can be easily established, either in-band our out-of-band.

## Integrity

SPUD must provide integrity protection of SPUD-encapsulated packets, though the details of this integrity protection are still open; see {{tradeoffs-in-integrity-protection}}. At the very least, endpoints should be able to:

1. detect packet modifications by non-SPUD-aware middleboxes along the path

2. detect the injection of packets into a SPUD flow (defined by 5-tuple) or tube by nodes other than the remote endpoint.

SPUD must provide integrity to detect modifications of information that are not supposed to be changed deliberately or non-deliberately by (SPUD-aware or not-SPUD-aware) middleboxes.

## Privacy

SPUD must allow endpoints to control the amount of information exposed to middleboxes, with the default being the minimum necessary for correct functioning.  [QUESTION: can/should endpoints be allowed to make different choices?  Minimal amount always trumps?  Need to be careful here about embedding policy in mechanism...]

# Non-Functional Requirements

The following requirements detail the constraints on how the SPUD facility must meet its functional requirements.

## Middlebox Traversal

SPUD must be able to traverse middleboxes that are not SPUD-aware. Therefore SPUD must be encapsulated in a transport protocol that is known to be accepted on a large fraction of paths in the Internet, or implement some form of probing to determine in advance which transport protocols will be accepted on a certain path.

## Low Overhead in Network Processing

SPUD must be low-overhead, specifically requiring very little effort to recognize that a packet is a SPUD packet and to determine the tube it is associated with.

## Implementability in User-Space

To enable fast deployment SPUD and transports above SPUD must be implementable without requiring kernel replacements or modules on the endpoints, and without having special privilege (root or "jailbreak") on the endpoints. Usually all operating systems will allow a user to open a UDP socket. Therefore SPUD has to be naturally encapsulated in UDP or at least a possibility to encapsulate SPUD in UDP must be specified in addition.

Incremental Deployability in an Untrusted, Unreliable Environment: :  SPUD must operate in the present Internet. In order to maximize deployment, it should also be useful as an encapsulation between endpoints even before the deployment of middleboxes that understand it. The information exposed over SPUD must provide incentives for adoption by both endpoints and middleboxes, and must maximize privacy (by minimizing information exposed). Further, SPUD must be robust to packet loss, duplication and reordering by the underlying network service.  SPUD must work in multipath, multicast, and multi-homing environments.

Minimum restriction on the overlying transport: : SPUD must impose minimal restrictions on the transport protocols it encapsulates. However, to serve as a substrate, it is necesary to factor out the information that middleboxes commonly rely on. This information should be included in SPUD and might add additional restrictions to the overlying transport. One example is that SPUD is likely to impose bidirectional communication for all transports on top of it. However, even though UDP is n unidirectional protocol, many communications on top of UDP are bidirectional anyway. For those services where only unidirectional communication is needed SPUD should potentially not be applied.

Minimum Header Overhead: : To avoid reducing network performance, the information and coding used in SPUD should be designed to use a minimum amount of additional bits.

No additional start-up latency: : SPUD should not introduce additional start-up latency for the overlying (non-connection-oriented) transport protocols.

# Open Questions

## Tradeoffs in tube identifiers

If SPUD is used, all packets of a 5-tuple flow must carry the SPUD header. Only packets with the same tube ID and 5-tuple are considered to belong to the same tube. (29.6. call: Not quite. There are three requirements here. (1) Need to be able to tie per-tube properties to all packets in the tube. How this happens is open. (2) Not allow additional trackability across 5-tuples. (3) Mitigate blind MOTS attacks. Discuss tradeoffs here. Maybe we need a Tube ID section? Talk about mapping streams in multistream protocols to tubes?)

We must address the tradeoff between extensibility and privacy preservation.
  While scoping the Tube ID to a five-tuple will reduce or eliminate its
  usefulness for tracking (as well as fixing problems with ID collision in NAT
  environments), one concern raised at the BoF is that a completely extensible
  mechanism for binding information to tubes, if taken together with an ability for middleboxes to arbitrarily add information to a tube, could make
  it easier for devices other than the endpoints to bind additional identifiers in-band for tracking purposes. [Addressed.]

## Tradeoffs in integrity protection

Split out endpoint authentication vs middlebox authentication. (1) detecting modification my non-spud path, (2) by spud-path, (3) packet injection. (Verify these are captured.)

# Poorly organized notes from the 13 May call

[EDITOR'S NOTE: blame Brian for the fact that this is a grab-bag of thoughts.]

- In general, we should piggyback SPUD as much as possible on the semantics of
  the overlying transport.

- One of the main goals in transport evolution is to support a spectrum in
  terms of reliability, as opposed to just TCP's "reliable and in order" and
  UDP's "thanks for the packet, good luck!". In this case, we cannot rely on the underlying transport to provide reliable retransmission, so if we are piggybacking an in-band signal over the overlying transport, that signaling
  must either be best-effort-tolerant, the endpoints will need (and the middleboxes may need) to know something about the overlying transport's reliability mechanism, or the substrate will need to provide its own reliability for signaling. Key exchange, for example, requires reliable transport.

- If reliable in-band signaling is not available, then signaling requiring
  reliability (mainly key exchange) can happen out-of-band.

- Related: Will it be necessary to build a feedback channel
  into SPUD if the overlying transport doesn't provide it? Further related: can we declare transports without a feedback channel to be out of scope (as they are by definition not congestion controlled without a feedback channel, and therefore potentially dangerous to use in the Internet)?

- There are different security considerations for different security contexts.
  The end-to-end context is one; anything that only needs to be seen by the
  path shouldn't be exposed in SPUD, but rather by the overlying transport.
  There are multiple different types of end-to-middle context based on levels of trust between end and middle -- is the middlebox on the same network as the endpoint, under control of the same owner? Is there some contract between the application user and the middlebox operator? We need to define the different kinds of relationships here more concretely.

- The final approach may mix in-band signaling with out-of-band signaling.
  It's possible the middle-to-end signaling in band will be primarily about
  middlebox discovery: based on some property of traffic passing through a
  middlebox, it may expose some information which can be used to initiate a
  conversation over some other protocol.

# Discussion & Open Questions (post 29.6.)

- Which packets do need to have a spud header? Does all packet of a 5-tuple flow have to have a SPUD header? Currently we say yes (see first requirement on grouping).

- Is spud information always per tube or can a spud packet also have per packet information? We don't know enough yet, discuss the tradeoffs...

- How does the discovery work? Three issues here -- first, we need to deal with protocols that sometimes run over spud and not. We need to deal with handling cases where spud doesn't work. We need to deal with cases where the working-ness of spud is dynamic. Second, we need to find spud-talking things on the path. Third, we need to discover which bits of the vocabulary are supported. Fourth, we need for endpoints to be able to figure out which overtransports are in use. Non-requirement: allowing the path to discover overtransports.

- consider adding draft-hardie use cases text here...

- ecn?

- Issues #5 and #6: split out endpoint authentication vs middlebox authentication. (1) detecting modification my non-spud path, (2) by spud-path, (3) packet injection. (Verify these are captured.)

# Security Considerations

[EDITOR'S NOTE: point to security- (and privacy-) relevant requirements here.]

# IANA Considerations

This document has no actions for IANA.

# Contributors

This document is the work of the SPUD Design Team within the IAB IP Stack Evolution program: Ken Calvert, Ted Hardie, Joe Hildebrand, Jana Iyengar, and Mirja Kuehlewind.

[EDITOR'S NOTE: make this a real contributor's section once we figure out how to make kramdown do that...]
