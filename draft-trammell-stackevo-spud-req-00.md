---
title: IAB Workshop on Stack Evolution in a Middlebox Internet (SEMI) Report
abbrev: SEMI Workshop
docname: draft-trammell-semi-report-00
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
  I-D.hardie-spud-use-cases:
  I-D.hildebrand-spud-prototype:
  I-D.huitema-tls-dtls-as-subtransport:
  I-D.trammell-stackevo-newtea:
  I-D.trammell-semi-report:
  I-D.ietf-rtcweb-data-channel:

--- abstract

The Substrate Protocol for User Datagrams (SPUD) BoF session at the IETF 92 meeting in Dallas in March 2015 identified the potential need for a UDP-based encapsulation protocol to allow explicit cooperation with middleboxes while using new, encrypted transport protocols. This document defines a set of requirements for this protocol.

--- middle

# Introduction

An outcome the outcomes of the IAB workshop on Stack Evolution in a Middlebox
Internet (SEMI) {{I-D.trammell-semi-report}}, held in Zurich in January 2015,
was a discussion on the creation of a substrate protocol to support the
deployment of new transport protocols in the Internet. Assuming that a way
forward for transport evolution in user space would involve encapsulation in
UDP datagrams, the workshop identified that it may be useful to have a
facility built atop UDP to provide minimal signaling of the semantics of a
flow that would otherwise be available in TCP: at the very least, indications
of first and last packets in a flow to assist firewalls and NATs in policy
decision and state maintenance. Further transport semantics would be used by
the protocol running atop this facility, but would only be visible to the
endpoints, as the transport protocol headers themselves would be encrypted
along with the payload to prevent inspection or modification, e.g. using DTLS
{{RFC6347}} as a subtransport {{I-D.huitema-tls-dtls-as-subtransport}}. This
facility could also provide minimal application-to-path and
path-to-application signaling, though there was less agreement exactly what
should or could be signaled here.

The Substrate Protocol for User Datagrams was held at IETF 92 in Dallas in
March 2015 to develop this concept further. Clear from discussion before and
during the SPUD BoF, and drawing on experience with previous
endpoint-to-middle and middle-to-endpoint signaling approaches, is that any
selective exposure of traffic metadata outside a relatively restricted trust
domain must be declarative as opposed to imperative, non-negotiated, and
advisory. Each exposed parameter should also be independently verifiable, so
that each entity can assign its own trust to other entities. Basic transport
over the substrate must continue working even if signaling is ignored or
stripped, to support incremental deployment. These restrictions on vocabulary
are discussed further in {{I-D.trammell-stackevo-newtea}}.

This document defines a specific set of requirements for a SPUD protocol,
based on analysis on a target set of applications to be developed on SPUD
developing experience with a prototype described in
{{I-D.hildebrand-spud-prototype}}. It is intended as the basis for eventually
chartering a working group for the further development of a SPUD protocol.

## Motivation

A number of efforts to create new transport protocols or experiment with new
network behaviors have been built on top of UDP, as it traverses firewalls and 
other middleboxes more commonly than new protocols do.  Each such effort must, 
however,either manage its flows within common middlebox assumptions for UDP 
or train the  middleboxes on the new protocol (thus losing the benefit of 
using UDP). A common substrate for UDP-based transports would allow each effort
to re-use a set of shared methods for notifying middleboxes of the flows' 
semantics, thus avoiding both the limitations of current flow semantics and
the need to re-invent the mechanism for notifying the middlebox of the new 
semantics.

As a concrete example, it is common for some middleboxes  to tear down required 
state (such as NAT bindings) very rapidly for UDP flows.   By notifying the
path that a particular transport using UDP maintains session state and explicitly
signals session start and stop using the substrate, the using protocol may avoid
the need for heartbeat traffic.

# Terminology

[EDITOR'S NOTE: todo]

# Use Cases

[EDITOR'S NOTE: specific applications we think need this go here]

# Initial requirements

These are taken from {{I.D-hildebrand-spud-prototype}}. I believe these all apply beyond the prototype as well. Each of these should eventually get its own subsection, with some discussion motivating each requirement.

- SPUD must enable multiple new transport semantics without requiring updates
  to SPUD implementations in middleboxes.

- Transport semantics and many properties of communication that endpoints
  may want to expose to middleboxes are bound to flows or groups of flows.
  SPUD must therefore provide a basic facility for associating packets
  together (into what we call a "tube" for lack of a better term).

- SPUD and transports above SPUD must be implementable without requiring
  kernel replacements or modules on the endpoints, and without having special
  privilege (root or "jailbreak") on the endpoints.

- SPUD must operate in the present Internet. In order to ensure deployment, it
  must also be useful as an encapsulation between endpoints even before the
  deployment of middleboxes that understand it.

- SPUD must be low-overhead, specifically requiring very little effort to
  recognize that a packet is a SPUD packet and to determine the tube it is
  associated with.

- SPUD must impose minimal restrictions on the transport protocols it
  encapsulates.  SPUD must work in multipath, multicast, and mobile
  environments.

- SPUD must provide incentives for development and deployment by multiple
  communities.  These communities and incentives will be defined through the
  prototyping process.

# Poorly organized notes from the 13 May call

[EDITOR'S NOTE: blame Brian for the fact that this is a grab-bag of thoughts.]

- We must address the tradeoff between extensibility and privacy preservation.
  While scoping the Tube ID to a five-tuple will reduce or eliminate its
  usefulness for tracking (as well as fixing problems with ID collision in NAT
  environments), one concern raised at the BoF is that a completely extensible
  mechanism for binding information to tubes, if taken together with an ability for middleboxes to arbitrarily add information to a tube, could make
  it easier for devices other than the endpoints to bind additional identifiers in-band for tracking purposes.

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

# Security Considerations

[EDITOR'S NOTE: point to security- (and privacy-) relevant requirements here.]

# IANA Considerations

This document has no actions for IANA.

# Contributors

This document is the work of the SPUD Design Team within the IAB IP Stack Evolution program: Ken Calvert, Ted Hardie, Joe Hildebrand, Jana Iyengar, and Mirja Kuehlewind.

[EDITOR'S NOTE: make this a real contributor's section once we figure out how to make kramdown do that...]
