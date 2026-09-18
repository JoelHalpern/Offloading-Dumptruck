---
coding: utf-8
coding: us-ascii

title: The IGP is not a Dump Truck
abbrev: rtgwg-not-a-dump-truck
docname: draft-halpern-rtgwg-not-a-dump-truck-latest
category: std
submissiontype: IETF
stand_alone: yes
pi: [toc, tocompact, tocindent, sortrefs, symrefs, compact]

ipr: trust200902
area: Routing
wg: RTGWG Working Group
kw: IGP

author:

  -
    ins: J. Halpern
    name: Joel Halpern
    org: HPE
    country: USA
    email: joel.halpern@hpe.com
  -
    ins: N. Buraglio
    name: Nick Buraglio
    org: Energy Sciences Network
    email: buraglio@forwardingplane.net
 -
    ins: A. Alston
    name: Andrew Alston
    org: Equity Technology Group
    email: Andrew.alston@equitybank.co.ke 
     
normative:
  RFC2104:
  RFC2119:

--- abstract

This document explores addressing the problem of using an IGP to carry 
arbitrary
information, often referred to using the phrase 
"the IGP is not a dump truck". 
It describes the kinds of information carried in an IGP
and proposes an approach to changing the system.

--- middle

# Introduction {#intro}

As the usage of protocols for IP routing within domains has evolved, 
a need has emerged to carry many more different kinds of information across 
the domain in support of the IP handling system.  This has resulted in a 
syndrome informally referred to as treating the IGP (the routing system 
within many domains) as a dump truck.

This phenomenon results in multiple kinds of problems.  The most obvious
effect is that the sheer volume of information grows faster than the 
operational systems to support it.  We have seen many cases where efforts
to add further information to the set are met with "that is too much". 
This leaves the requestor with a need that is hard to meet.  Further, as the 
IGP operations require (slow) refresh of information, we pay additional 
information propagation costs that are not well-matched to the problem.  
The net effect is a set of needs that are not well-met and which can 
interfere with effective operation of the IGP for its primary purpose.

This draft outlines an approach to changing the situation.  It starts by
describing the information carried in the IGP in terms of the dynamics
relevant to separating the dump truck aspects.  That section then 
describes what appears to be a tractable and useful separation.

An additional section then starts to describe the interface properties
that a solution proposal can look at.

This is followed by a brief description of the infrastructure of a solution,
while explicitly not picking a mechanism.  There are a few properties that
are anticipated to be needed by any infrastructure addressing this space.

Finally, and in many ways most importantly, there is a description of the
deployability issues and properties that need to be recognized to make this
undertaking effective.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Information categorization

To discuss what information can be helpfully unloaded, this draft proposes 
three categories of information.

1. Highly dynamic information - This is information that changes faster than 
typical routing information.  An example would be the current congestion state 
of a queue.  This information is not the subject of this draft.  How it is 
handled is left to other documents and practices.

2. Topological state and similar information - This is the core of the job of 
routing protocols.  This draft does not propose to change this.  Information 
in this category is critical for driving forwarding decisions, and changes 
at a moderate rate.

3. Long term stable information - This is information that while subject
to change as device software or configuration changes, is not subject to 
rapid environmental effects or similar dynamics.  
Examples include node capability
information, the set of VPNs a node is supporting, and similar.  
Link MTU is another example of information that can be handled by
this system.  Service SIDs
are an example of information which while currently supported in other ways
may benefit from this mechanism if it gains traction.
Whether the supported set of information
should include information such as link latency for non-radio links is for
future discussion.  The first important property is that this information 
does not need to be refreshed.  So designs built around reliable delivery
can be considered.  It is also frequently information which is only of 
interest to a subset of nodes.  So a distribution mechanism that does not
burden uninterested nodes with effort or storage is worth evaluating.

# Information distribution properties

From the point of view of standardization, the important part of this 
proposal is a mechanism to put information into the system, and get 
information out of the system

## Discovery

To participate in this system in either or both of the roles of
information provider or information recipient, a node needs to discover
an entry point to the system.  It is assumed for purposes of discussion
that this appears as a list of IP Address / Protocol / Port / Higher layer
behavior tuples.  The exact encoding is yet to be determined, but should 
leverage existing work in this area.  The choice among explicit list, 
some form of
URN/URI, or a different representation are considerations to be evaluated
later in the process.

As is discussed in the infrastructure section, it is assumed that the 
system appears to be a distributed service.  As such, a node may receive
multiple identifier tuples.

It is for further study whether the provision of these identifiers to nodes
is done via an advertisement in a routing protocol or is configured on each
participating node. 

## Information Structure

The information to be handled by the system must have certain properties
in order for this proposal to be effective.

Each piece of information needs to be uniquely identified so that
the system can tell the difference among adding new information, updating
information, or deleting information.  It is likely that this will be
achieved via a combination of domain-wide unique identifiers of system
participants and local generation of identifiers for information.

Information in this offload system needs to be useable in relation
to the underlying routing system.  As such, advertisements in this system
will need to use identifiers that correlate with node and link identification
as understood by the IGP.   To a first approximation, one can assume a single
style of IGP, and use the identifiers exactly as it represents them.  It is
worth exploring whether this should be generalized.

To enable distribution of information only to interested parties, there
also needs to be a standardized way to represent the semantic scope (e.g.
provider edge devices, devices supporting specific VPNs, devices serving 
as head ends for a particular tunnel technology.)

Information has to have enough structure and content identification
that it can be understood.  This implies the use of standardized schema of
some sort, standardized encoding, and well-defined extensibility so that
new information can be introduced incrementally.  Whether YANG is the right
schema, or some other already defined candidate is to be used remains to
be determined.  It is not the goal of this proposal to introduce a new
schema language.

## Providing Information

Once a node has one or more identifiers for this offload service, it 
contacts the service.  It is presumed that there is some sort of 
registration and authentication process, and that all communication
with the service will have the capability at least for integrity
protection.  Whether confidentiality is needed is for further study
and may well depend upon the specific operational environment.

Once properly registered, a node provides to the system its collection 
of suitably static information such as nodal capabilities.  The properties
of this information are described above.  Encoding details such
as whether lengths are used, etc. are for future study.  A stable definition
of this information structure is needed for interoperability.  Equally,
extensibility is critical as we cannot and should not restrict what 
information is handled by this system.

Providers of information will need to be able to withdraw information they
have provided and update the content of specific items.

## Collecting information

A node desiring to retrieve information from the system registers and
authenticates as above.  Following that, it provides a series of requests
for information.  The common case will be a request for some scoped set 
of information from all contributors, with updates when content changes
such as new information becoming available, information being withdrawn,
or an update of content.  In particular, this frequently occurs whan
a new node joins the operating environment.  There is no rush
about deleting out-of-date information, although explicit deletion does need
to be propagated.

# Infrastructure properties

The obvious starting point for such a system is a set of servers offering the
information handling service.  This must be distributed and replicated so as
to be resilient across failure of individual servers.  It is assumed that
the system can be delivered by a combination of services co-resident with 
routers and separate software servers, as the operator prefers.

The infrastructure will need a mechanism so that when new servers join the
system, such new participants can reliably fetch the data set.  It is not 
expected that the system instances will have persistent storage, as, under a 
full restart, the supporting nodes will re-register and provide their 
information.

The exact mechanism for doing this may be a distributed database, a pub-sub
service, or any number of techniques developed in the server world.  
Assuming we have a stable interface, it is not clear whether there is even 
a need to standardize one or more infrastructure handling systems.

# Deployment considerations

This represents a significant change in the information handling paradigm
of an operational environment.  This leads to the question of whether it
is practical to deploy this.

There are clearly a number of existing use cases for which this system
would improve the overall behavior.  However, it is hard to see how vendors
and operators would justify the complications of changing already deployed,
if awkward distribution mechanisms.

As such, the expected first step would be to define this, and define its use 
for new use cases which are currently emerging and seem to place an 
unfortunate strain on IGPs.  Only once there is traction for this approach
in operational environments would we then propose carrying some of the 
existing stable information cases in this and, if successful, gradually
make more use of this approach.

# Security Considerations

Any protocol effort intending to address this problem will need to include
authentication and confidentiality mechanisms, along with analysis of how
these mechanisms address the risks of inappropriate information disclosure.  


# IANA Considerations

This document does not assign or modify any IANA code points.

# Acknowledgements

This document does not yet have any acknowledgements.
