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

  G2-SMETRICS:
    title : "XXX"
    author:
      -
        ins: J. Ros-Giralt
        name: Jordi Ros-Giralt
        org: Reservoir Labs

  G2-SCOMM:
    title : "XXX"
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
    title : Computing Bottleneck Structures at Scale for High-Precision Network Performance Analysis
    name : Noah Amsel, Jordi Ros-Giralt, Sruthi Yellamraju, Brendan von Hofe, Richard Lethin
    org : Reservoir Labs
    seriesinfo : "IEEE International Workshop on Innovating the Network for Data Intensive Science (INDIS), Supercomputing"

  LORENZ:
    title : "Does the flap of a butterfly’s wings in Brazil set off a tornado in Texas?"
    name : "Edward N Lorenz"
    date : 1972
    seriesinfo : "American Association for the Advancement of Science, 139th Meeting"


--- abstract

TODO Abstract


--- middle

# Introduction

This document proposes an extension to the base Application-Layer Traffic Optimization (ALTO) protocol
to support bottleneck structures as an efficient represention of the state of a network. Bottleneck
structures have been recently introduced in [G2-SCOMM] and [G2-SMETRICS] as computational graphs that
embed information about the topology, routing and flow information of a network. These computational
graphs allow operators and applications to efficiently compute network derivatives that can then be
used to make optimized traffic engineering decisions. For instance, using the bottleneck structure of a
network, an application can infer the current available bandwidth on a specific network path traversing multiple
network domains. Bottleneck structures can be used by the application to address a wide variety of
communication optimization problems, including routing, flow control, flow scheduling, network
slicing or capacity planning, among others. This extension introduces a new abstraction called
Gradient Graph (GG) to represent the bottleneck structure
of the network and extensions to the existing ALTO services (Network Map, Cost Map, Entity Property Map
and Endpoint Cost Map) to expose the properties of the bottleneck structure to help optimize application
optimize.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Brief Introduction to Bottleneck Structures

[G2-SMETRICS] and [G2-SCOMM] introduces a new mathematical framework to optimize network performace called the
Quantitative Theory of Bottleneck Structures (QTBS). QTBS builds a computational graph called "bottleneck
structure", which allows to qualify and quantify the forces of interactions that flows and
bottleneck links exert on each other. QTBS builds the bottleneck structure by assuming that flows in a network aim at maximizing
throughput while ensuring fairness. This general principle holds for all congestion control algorithms implemented
in the Internet (TCP Cubic, BBR, Quic, etc.)

## Example of Bottleneck Structure

Consider as an example the following network configuration:


	     f3 f6 f1
	      │ │ │
	      │ │ │
	    ┌─┼─┼─┼─┐
	    │ │ │ └─┼───        l1
	    │ │ │   │           c1=25
	    │ │ │   │
	    └─┼─┼───┘
	      │ │     f2
	      │ │ ┌─────
	      │ │ │
	    ┌─┼─┼─┼─┐
	────┼─┘ │ │ │           l2
	    │   │ │ │           c2=50
	────┼─┐ │ │ │
	f4  └─┼─┼─┼─┘
	      │ │ │
	      │ │ └─────
	      │ │
	    ┌─┼─┼───┐
	────┼─┘ └───┼────       l3
	    │       │           c3=100
	    │   ┌───┼────
	    └───┼───┘  f5
		│
		│
	    ┌───┼───┐
	    │   │   │           l4
	    │   └───┼────       c4=75
	    │       │
	    └───────┘
{: #net title="Network configuration example." }


Each link l_i is represented by a squared box while each flow f_i is represented by a line. In addition,
each link has a capacity c_i in units of bps. The bottleneck structure of this network corresponds to
the following digraph (see [G2-TREP] for details on how a bottleneck structure is computed):


	┌────────┐   ┌────────┐                   ┌────────┐
	│        │   │        │                   │        │
	│        ◄───┤        ├───────────────────►        │
	│   f1   │   │   l1   │                   │   f6   │
	│        │   │        │    ┌──────────────┤        │
	└────────┘   └────┬───┘    │              └───┬────┘
			  │        │                  │
			  │        │                  │
			  │        │                  │
		     ┌────▼───┐    │                  │
		     │        │    │                  │
		     │        │    │                  │
		     │   f3   │    │                  │
		     │        │    │                  │
		     └────┬───┘    │                  │
			  │        │                  │
			  │        │                  │
			  │        │                  │
		     ┌────▼───┐    │ ┌────────┐   ┌───▼────┐   ┌────────┐
		     │        ◄────┘ │        │   │        │   │        │
		     │        │      │        │   │        │   │        │
		     │   l2   ├─────►│   f4   ├───►   l3   │   │   l4   │
		     │        │      │        │   │        │   │        │
		     └────┬───┘      └────────┘   └───┬────┘   └────┬───┘
			  │                           │             │
			  │                           │             │
			  │                           │             │
		     ┌────▼───┐                       │ ┌────────┐  │
		     │        │                       │ │        │  │
		     │        │                       │ │        │  │
		     │   f2   │                       └─►   f5   ◄──┘
		     │        │                         │        │
		     └────────┘                         └────────┘
{: #fgg title="Bottleneck structure of the network in Figure 1." }


The bottleneck structure is interpreted as follows:

- Links and flows are represented by vertices in the graph.

- There is a directed edge from a link l to a flow f if and only if flow f is bottlenecked at link l.

- There is a directed edge from a flow f to a link l if and only if flow f traverses link l.

For instance, we have that flow f3 is bottlenecked at link l1 (since there is a directed edge from
l1 to f3) and it traverses links l1 and l2 (since there is a directed edge from f3 to l1 and from f3
to l2).

## Some Mathematical Principles of Bottleneck Structures

QTBS demonstrates that, under the assumption of max-min fairness, the following two properties hold:

- Property 1. Flow perturbation. An infinitesimal change in the transmission rate of a flow f will have an effect on the transmission rate
of a flow f'  if and only if there exists a path from flow f to flow f'.

- Property 2. Link perturbation. An infinitesimal change in the capacity of a link l will have an effect on the transmission rate
of a flow f' if and only if there exists a path from link l to flow f.

The above two properties qualitatively relate to the classic question in chaos theory: Can the flap of a butterfly’s wings
in Brazil set off a tornado in Texas? (see [LORENZ]) Obviously a butterfly alone cannot create a tornado, but every element
is interconnected in a distributed system, and even the flap of a butterfly's wings in Brazil will have an effect in Texas.
That's the type of effect that bottleneck structures can characterize and quantify.
In particular, a bottleneck structure reveals how a small perturbation in a network propagates through it,
describing which flows will be affected by such a perturbation. These forces of interactions are in general non-intuitive,
even for a small simple network configuration like the one in {{net}}. For instance, from Property 2, the bottleneck structure reveals
that a small variation in the capacity of link l2 (e.g., in a wireless network, a variation in the capacity of a link
could be due to a change in the signal to noise ratio of the communication channel) will propagate through the network
and have an impact on the transmission rate of flows f2, f4 and f5 (since from Property 2, in the bottleneck structure
there is a directed path from link l2 to each of these flows). However, such a perturbation, will have no effect
on the transmission rate of flows f1, f3 and f6 (since there is no path from l2 to any of these other flows). Similarly,
from property 1, a small perturbation on the rate of flow f4 (e.g., this could be due to the effect of a traffic
shaper altering the transmission rate of flow f4), will have an impact on the rate of flows f2 and f5, but it
will have no effect on the rate of flows f1, f3 and f6. As another example, given the network in {{net}}, it is also not most intuitive
that while a perturbation on flow f1 will have an impact on flow f5 (even though they do not traverse any common link),
a perturbation on flow f5 will not have any effect on flow f1. (See [G2-SMETRICS] for empirical validation of the presence
of bottleneck structures in IP networks and of Properties 1 and 2.)

Bottleneck structures not only allow network opertors to reason about the qualitative nature of the forces that flows
and links exert on each other, but they also allow them to quantify such forces. This leads to the Quantitative
Theory of Bottleneck Structures (QTBS), introduced in [G2-TREP]. QTBS introduces the link and flow equations, which
mathematically characterize how a perturbation in a network propagates through each link and flow
vertex in the bottleneck structure (see [G2-TREP] for exact mathematical details). This makes bottleneck structures
an efficient computational graph to compute flow and link derivatives with precision, at scale, and fast,
about two or three orders of magnitude faster than using general purpose techniques such as LP (see [G2-SC]).

## Applications of Bottleneck Structures

XXX

# Use Cases

TODO

# Requirements

TODO

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
