---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc2629: https://github.com/cabo/kramdown-rfc2629
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "ALTO Extension: Gradient Graph Service"
category: info

docname: draft-giraltyellamraju-alto-gradient-graph-latest
ipr: trust200902
area: Application
workgroup: ALTO
keyword: ALTO
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Jordi Ros-Giralt
    organization: Qualcomm
    email: jros@qti.qualcomm.com
 -
    name: Sruthi Yellamraju
    organization: Qualcomm
    email: yellamra@qti.qualcomm.com

normative:

informative:

  G2-SIGMETRICS:
    title : "XXX"
    seriesinfo : "ACM SIGMETRICS"
    author:
      -
        ins: J. Ros-Giralt
        name: Jordi Ros-Giralt
        org: Reservoir Labs

  G2-SIGCOMM:
    title : "XXX"
    seriesinfo : "ACM SIGCOMM"
    author:
      -
        ins: J. Ros-Giralt
        name: Jordi Ros-Giralt
        org: Reservoir Labs

  G2-TREP:
    title : "XXX"
    author:
      -
        ins: J. Ros-Giralt
        name: Jordi Ros-Giralt
        org: Reservoir Labs

  G2-SC:
    title : "Computing Bottleneck Structures at Scale for High-Precision Network Performance Analysis"
    seriesinfo : "IEEE International Workshop on Innovating the Network for Data Intensive Science (INDIS), Supercomputing"
    author:
      -
        ins: J. Ros-Giralt
        name: Jordi Ros-Giralt
        org: Reservoir Labs

  LORENZ:
    title : "Does the flap of a butterfly’s wings in Brazil set off a tornado in Texas?"
    seriesinfo : "American Association for the Advancement of Science, 139th Meeting"
    author :
      -
        ins: Edward N Lorenz
    date : 1972

  GALLAGER:
    title : "Data Networks"
    author :
      -
        ins: "Robert Gallager and Dimitri Bertsekas"

  B4-SIGCOMM :
    title : "B4: Experience with a Globally-Deployed Software Defined WAN"
    author :
      -
        ins:
    date : 2013
    seriesinfo : "ACM SIGCOMM"

  BE-SIGCOMM :
    title : "BwE: Flexible, Hierarchical Bandwidth Allocation for WAN Distributed Computing"
    author :
      -
        ins:
    date : 2015
    seriesinfo : "ACM SIGCOMM"



--- abstract

TODO Abstract


--- middle

# Introduction

This document proposes an extension to the base Application-Layer Traffic Optimization (ALTO) protocol
to support bottleneck structures as an efficient representation of the state of a network. Bottleneck
structures have been recently introduced in [G2-SIGCOMM] and [G2-SIGMETRICS] as computational graphs that
embed information about the topology, routing and flow information of a network. These computational
graphs allow network operators and application service providers to efficiently compute network derivatives that can be
used to make optimized traffic engineering decisions. For instance, using the bottleneck structure of a
network, an application can infer the current available bandwidth on a specific network path traversing multiple
network domains. Bottleneck structures can be used by the application to address a wide variety of
communication optimization problems, including routing, flow control, flow scheduling, network
slicing or capacity planning, among others. This extension introduces a new abstraction called
Gradient Graph (G2) to represent the bottleneck structure
of the network and extensions to the existing ALTO services (Network Map, Cost Map, Entity Property Map
and Endpoint Cost Map) to expose the properties of the bottleneck structure to help optimize application
optimize.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Brief Introduction to Bottleneck Structures

[G2-SIGMETRICS] and [G2-SIGCOMM] introduce a new mathematical framework to optimize network performace called the
Quantitative Theory of Bottleneck Structures (QTBS). The core building block of QTBS is a computational graph called "bottleneck
structure", which allows to qualify and quantify the forces of interactions that flows and
bottleneck links exert on each other. QTBS builds the bottleneck structure by assuming that flows in a network aim at maximizing
throughput while ensuring fairness. This general principle holds for all congestion control algorithms implemented
in the Internet (TCP Cubic, BBR, Quic, etc.)

## Example of Bottleneck Structure

Consider as an example the following network configuration:


                             f3 f6 f1
                              | | |
                              | | |
                           +--+-+-+---+
                           |  | | |   |
                           |  | | +---+---   l1
                           |  | |     |      c1=25
                           |  | |     |
                           +--+-+-----+
                              | |
                              | | +----- f2
                              | | |
                           +--+-+-+---+
                           |  | | |   |
                       ----+--+ | |   |      l2
                           |    | |   |      c2=50
                    f4 ----+--+ | |   |
                           |    | |   |
                           +--+-+-+---+
                              | | |
                              | | +-----
                              | |
                           +--+-+-----+
                           |  | |     |
                       ----+--+ +-----+----  l3
                           |          |      c3=100
                       ----+----+     |
                           |    |     |
                           +----+-----+
                                |
                                |
                           +----+-----+
                           |    |     |      l4
                     f5 ---+----+     |      c4=75
                           |          |
                           |          |
                           |          |
                           +----------+
{: #net title="Network configuration example." }


Each link l_i is represented by a squared box while each flow f_i is represented by a line. In addition,
each link has a capacity c_i in units of bps. The bottleneck structure of this network corresponds to
the following digraph (see [G2-TREP] for details on how a bottleneck structure is computed):

       +--------+   +--------+                   +--------+
       |        |   |        |                   |        |
       |        <--->        <------------------->        |
       |   f1   |   |   l1   |                   |   f6   |
       |        |   |        |   +---------------+        |
       +--------+   +----^---+   |               +---+----+
                         |       |                   |
                         |       |                   |
                         |       |                   |
                    +----v---+   |                   |
                    |        |   |                   |
                    |        |   |                   |
                    |   f3   |   |                   |
                    |        |   |                   |
                    +----+---+   |                   |
                         |       |                   |
                         |       |                   |
                         |       |                   |
                    +----v---+   |  +--------+   +---v----+  +--------+
                    |        <---+  |        |   |        |  |        |
                    |        |      |        |   |        |  |        |
                    |   l2   <----->|   f4   +--->   l3   |  |   l4   |
                    |        |      |        |   |        |  |        |
                    +----^---+      +--------+   +--^-----+  +-----^--+
                         |                          |              |
                         |                          |              |
                         |                          |              |
                    +----v---+                      |  +--------+  |
                    |        |                      |  |        |  |
                    |        |                      |  |        |  |
                    |   f2   |                      +-->   f5   <--+
                    |        |                         |        |
                    +--------+                         +--------+
{: #fgg title="Bottleneck structure of the network in Figure 1." }


The bottleneck structure is interpreted as follows:

- Links and flows are represented by vertices in the graph.

- There is a directed edge from a link l to a flow f if and only if flow f is bottlenecked at link l.

- There is a directed edge from a flow f to a link l if and only if flow f traverses link l.

For instance, in {{fgg}} we have that flow f3 is bottlenecked at link l1 (since there is a directed edge from
l1 to f3) and it traverses links l1 and l2 (since there is a directed edge from f3 to l1 and from f3
to l2).


## Propagation lemmas

Under the assumption of max-min fairness [GALLAGER], QTBS demonstrates the following two properties:

- Property 1. Flow perturbation. An infinitesimal change in the transmission rate of a flow f will have an effect on the transmission rate
of a flow f' if and only if the bottleneck structure has a directed path from flow f to flow f'.

- Property 2. Link perturbation. An infinitesimal change in the capacity of a link l will have an effect on the transmission rate
of a flow f' if and only if the bottleneck structure has a directed path from link l to flow f.

The above two properties qualitatively relate to the classic question in chaos theory: Can the flap of a butterfly’s wings
in Brazil set off a tornado in Texas? (see [LORENZ]) Obviously a butterfly alone cannot create a tornado, but every element
is interconnected in a distributed system, and even the flap of a butterfly's wings in Brazil will have an effect in Texas.
Bottleneck structures are graphs that characterize and quantify such type of effects in a communication network.
In particular, a bottleneck structure reveals how a perturbation propagates through the network,
describing which flows will be affected and by what magnitude.


## Forces of interaction among flows links

Bottleneck structures are powerful computational graphs because they are able to capture the forces of interaction
that flows and bottleneck links exert on each other. These forces of interactions are in general non-intuitive,
even for a small simple network configuration like the one in {{net}}. For instance, from Property 2, the bottleneck structure reveals
that a small variation in the capacity of link l2 (e.g., in a wireless network, a variation in the capacity of a link
could be due to a change in the signal to noise ratio of the communication channel) will propagate through the network
and have an impact on the transmission rate of flows f2, f4 and f5 (since from Property 2, the bottleneck structure
has a directed path from link l2 to each of these flows). However, such a perturbation will have no effect
on the transmission rate of flows f1, f3 and f6 (since there is no path from l2 to any of these other flows). Similarly,
from property 1, a small perturbation on the rate of flow f4 (e.g., this could be due to the effect of a traffic
shaper altering the transmission rate of flow f4), will have an impact on the rate of flows f2 and f5, but it
will have no effect on the rate of flows f1, f3 and f6.


## Ripple effects in a communication network

As another example, given the network in Figure 1, it is also not
most intuitive to foresee that flows f1 and f5 are related to each
other by the forces of interaction inherent to the communication
network, even though they do not traverse any common link.
Specifically, flow f1 traverses link l1, while flow f5 traverses links
l3 and l4. In between both flows, there is an additional hop
(link l2) further separating them.  Despite not being directly
connected, the bottleneck structure reveals that a small perturbation
on the performance of flow f1 (i.e., a change in its transmission
rate), will create a ripple effect that will reach flow f5, affecting
its transmission rate.  In particular, the perturbation on flow f1 will
propagate through the bottleneck structure and reach flow f5
via the following two paths:

    f1 -> l1 -> f3 -> l2 -> f4 -> l3 -> f5

    f1 -> l1 -> f6 -> l3 -> f5

It is also not intuitive to see that the reverse is not true.  That
is, a small perturbation on flow f5, will have no effect on flow f1,
since the bottleneck structure has no direct path from vertex f5 to
vertex f1.  In [G2-SIGMETRICS], empirical validation of these results
is presented for a variety congestion-controlled IP networks.


## Not all bottleneck links are born equal

Bottleneck structures also reveal that not all links have the same
relevancy.  In Figure 2, links at the top of the graph have a higher
impact to the overall performance of the network than links at the
bottom.  For instance, consider link l1 first.  A variation on its
capacity will create a ripple effect that will impact the performance
of all the flows in the network, since all flow vertices are
reachable from vertex l1 according to the bottleneck structure.  In
contrast, link l3 has a much smaller impact on the overall
performance of the network, since a variation of its capacity will
affect flow f5, but have no impact on any of the other flows.  This
information is valuable to network operators and application service
providers as it can be used to make traffic engineering decisions.
For instance, in edge computing, an operator could choose to place a
containerized service (e.g., for XR), on compute nodes that would
yield communication paths traversing bottleneck links with lower
impact on the overall performance of the network.

Overall, bottleneck structures provide a mechanism to rank bottleneck
links according to their impact on the overall performance of the network.
This information can be used in a variety of optimization problems such
as traffic engineering, routing, capacity planning, or resilience analysis,
among others.


## Quantifying the ripple effects

Bottleneck structures not only allow network operators to reason
about the qualitative nature of the forces that flows and links exert
on each other, but also provide a mathematical framework to quantify
such forces. In particular, the Quantitative Theory of Bottleneck
Structures (QTBS) introduced in [G2-TREP] provides a mathematical
framework that uses bottleneck structures as efficient computational
graphs to quantify the impact that perturbations in a network have
on all of its flows.

One of the core building blocks of the QTBS framework
is based on the *link and flow equations*,
which mathematically characterize how a perturbation in a network
propagates through each of the link and flow vertices in the
bottleneck structure (see [G2-TREP] for an exact mathematical formulation).
Because quantifying the effect of a perturbation on a system is nothing more
than computing a derivative of the system's performance with respect to
the parameter that's been perturbed, bottleneck structures can be used as
efficient and scalable computational graphs to compute
flow and link derivatives in a communication network.

Consider for instance the computation of the following derivative:

    d F() / d r_i

where F() represents the total throughput of the network (the sum of
all flows' throughput) and r_i is the transmission rate of flow f_i.
Computing this derivative using a traditional calculus approach is
both very complex and costly, since it requires modeling the
congestion control algorithm in the function F(), for which there is
no closed form solution. Using bottleneck structures, however, the
computation of this derivative is both simple and inexpensive.  It is
simple because it can be done by applying an infinetissimal
change in the rate of flow f_i and then using the link and flow
equations to measure how this perturbation propagates through the
network [G2-TREP], [G2-SIGCOMM].
It is also very efficient because the computation is
performed by applying delta calculations on the bottleneck structure,
without involving links and flows that are not affected by the
perturbation.  For instance, in Figure 1, the computation of d F() /
d f4 only requires recomputing the transmission rates of flows f2
and f5, without the need to recompute the rates of f1, f3 and f6,
since these other flows are not affected by the perturbation.  In
practice, QTBS provides a methodology to
compute network derivatives two or three orders of magnitude faster than
general purpose methods such as liner programming [G2-SC].


## Types of pertubations

Bottleneck structures can be used to compute a variety of derivatives
(perturbations) on a network, which enables a mathematical framework
to optimize a large variety of communication network problems. The
following is a list of some of the derivatives that can be efficiently
computed using QTBS:


- **Flow routing**. Both the operation of routing a new flow or
rerouting an existing flow on a network can be modeled as a perturbation,
whose impact can be efficiently measured using bottleneck structures.
For instance, QTBS can be used to resolve the joint congestion
control and routing problem for individual flows (see Section 3.1 in [G2-TREP]).

- **Traffic shaping**. Traffic shaping a flow correspond to the action
of taking a derivative with respect to the rate of the flow. Bottleneck
structures can be used by network operators and application service
providers to compute such type of perturbations. For instance, to accelerate
a large scale data transfer, an operator can use bottleneck structures
to identify optimal traffic shaping configurations (see Section 3.3 in
[G2-TREP]).

- **Bandwidth enforcement**. In high-performance networks that target
close to 100% link utilization such as Google's B4 network [B4-SIGCOMM],
a centralized SDN controller is used collect the state of the network
and compute an optimized multipath bandwidth allocation vector. The
solution is then deployed at the edge of the network using a technique
known as bandwidth enforcement [BE-SIGCOMM]. By using bottleneck structures
to efficiently compute changes in the bandwidth allocated to each flow path,
operators can efficiently derive improved bandwidth allocations vectors.

- **Link capacity upgrades**.

- **Path shortcuts**.

- **Flow scheduling**.

- **Flow completion**.

- **Job mapping**.

- **Multi-job scheduling**.


Because of the generality of the QTBS framework, bottleneck
structures can be used in a wide variety of network optimization
problems, including traffic engineering, routing, flow scheduling,
network design, capacity planning, resiliency analysis, network
slicing, or service level agreement (SLA) management, among others.

In each of these problems, the goal is to configure the network
in a way that a certain objective function is optimized. Optimization
requires the capability to compute derivates,

In the next section, we describe some of the use cases.


# ALTO G2 Service Use Cases

Bottleneck structures have numerous applications in the area of
communication networks, including traffic engineering, routing,
flow scheduling, network design, capacity planning, resiliency analysis,
network slicing, or service level agreement (SLA) management, among others.

In this section we describe a few use cases that relate to the objectives
of the IETF ALTO Standard (See [G2-TREP], [G2-SIGCOMM] and [G2-SIGMETRICS] for
a broader set of applications):


## Application Rate Limiting for CDN and Edge Cloud Applications

Bottleneck structures can be used to estimate the available bandwidth on a given path.
This information can be used by real-time applications such as CDN, XR, IoT or V2X to adjust the encoder's transmission
rate to match the path's available bandwidth [G2-SIGMETRICS].

## Time-bound Constrained Flow Acceleration for Large Data Set Transfers

XXX

## AI Hybrid Modeling for Bandwidth Prediction

XXX

## Optimal joint congestion control and routing.

XXX

## Service Placement for Edge Computing

Determining the proper location to deploy an application service in the edge cloud is critical
in order to ensure good performance, measured in terms of latency and available bandwidth. Bottleneck structure can
be used to help the application provider know where (in which location in the edge cloud) to deploy the server.

## 5G Network Slicing

Bottleneck structures can also be used by network operators and application service providers to
define optimized network slices. See [G2-SIGCOMM] for examples on how bottleneck structures can be used to design
network slices in data centers.

# Requirements

This section provides an illustrative example of how an application can benefit from the bottleneck structure
service defined in this document. From this example, we then provide a discussion on the necessary requirements
to integrate BSS into the ALTO standard.

Figure

        +------+  +------+  +------+  +------+   +------+    +------+
        |      |  |      |  |      |  |      |   |      |    |      |
        | DC1  +--+ DC3  +--+ DC4  +--+ DC7  +---+ DC11 +----+ DC12 |
        |      |  |      |  |      |  |      |   |      |    |      |
        +---+--+  +--+---+  +--+-+-+  +----+-+   +-+--+-+    +----+-+
            |        |         | |         |       |  |           |
            |        +-------+ | +------+  |       |  +--------+  |
            |                | |        |  |       |           |  |
            |                | |        |  |       |           |  |
            |        +-------+-+ +------+--+       |   +-------+--+
            |        |       |   |      |          |   |       |
        +---+--+  +--+---+  ++---+-+  +-+----+   +-+---++    +-+----+
        |      |  |      |  |      |  |      |   |      |    |      |
        | DC2  +--+ DC5  +--+ DC6  +--+ DC8  +---+ DC10 +----+ DC9  |
        |      |  |      |  |      |  |      |   |      |    |      |
        +------+  +------+  +------+  +------+   +------+    +------+

        |      |  |                          |   |                  |
        +------+  +--------------------------+   +------------------+
          Asia                 America                 Europe
{: #b4 title="A subset of Google's B4 network introduced in [B4-SGOM]." }

       +--------+         +--------+
       |        |         |        |
       |        |         |        |
       |  l15   |         |   l7   |
       |        |         |        |
       +-^----^-+         +--^--^--+
         |    |              |  |
         |    |              |  +------------------------------+
         |    |              |                                  |
         |    +--------------|------------------+               |
         |                   |                  |               |
       +-v------+         +--v-----+       +----v---+       +---v----+
       |        |         |        |       |        |       |        |
       |        |         |        |       |        |       |        |
       |   f1   |         |   f2   |       |   f3   | (...) |   f10  |
       |        |         |        |       |        |       |        |
       +-+-+-+--+         +-+----+-+       +---+--+-+       +--------+
         | | |              |    |             |  |
         | | +--------------|----|-------------|--|----------------+
         | |                |    |             |  |                |
         | | +--------------+    +-----------+ |  |                |
         | | |                               | |  |                |
         | +-|--------------+    +-----------|-+  +-----------+    |
         |   |              |    |           |                |    |
         |   |              |    |           |                |    |
       +-v---v--+         +-v----v-+       +-v------+       +-v----v-+
       |        |         |        |       |        |       |        |
       |        |         |        |       |        |       |        |
       |   l8   |         |  l10   |       |  l5    |       |   l3   |
       |        |         |        |       |        |       |        |
       +-^---^--+         +-^---^--+       +--------+       +--------+
         |   |              |   |
         |   |              |   +--------------------------------+
         |   |              |                                    |
         |   +--------------|------------------+                 |
         |                  |                  |                 |
       +-v------+         +-v------+       +---v----+        +---v----+
       |        |         |        |       |        |        |        |
       |        |         |        |       |        |        |        |
       |   f6   |         |   f9   |       |  f23   | (...)  |  f18   |
       |        |         |        |       |        |        |        |
       +--------+         +--------+       +--------+        +--------+
{: #b4fgg title="Bootleneck structure of Google's B4 network example." }



# Gradient Graph Extension: Overview

TODO

# Specification: Basic Data Types

TODO

# Specification: Service Extensions

TODO

# Compatibility with Other ALTO Services and Extensions

TODO

# General Discussions

TODO

# Examples

TODO

# Security Considerations

TODO

# IANA Considerations

TODO

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
