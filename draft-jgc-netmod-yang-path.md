---
title: "YANG path format"
category: info

docname: draft-jgc-netmod-yang-path-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Network Modeling"
keyword:

- YANG
- path
- YANG path
- JSON instance path

venue:
  group: "Network Modeling"
  type: "Working Group"
  mail: "netmod@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/netmod/"
  github: "jgcumming/draft-jgc-netmod-yang-path"
  latest: "https://draft-jgc-netmod-yang-path.jgc.dev/draft-jgc-netmod-yang-path.html"

author:
 -
    fullname: "James Cumming"
    organization: Nokia
    email: "james.cumming@nokia.com"

normative:

informative:

  RFC9535:
  XPATH:
    target: https://www.w3.org/TR/1999/REC-xpath-19991116/
    title: XML Path Language (XPath) Version 1.0
    author:
      - name: W3C
    date: false

...

--- abstract

There is a common path format described when interacting with YANG
modules and the tools surrounding it, named xpath.  The xpath format
has some shortcomings for YANG module and tool designers and users.

This draft provides a common path format, the YANG path, to
provide a self describing method of detailing YANG modelled paths
(both in schema and in instance data).

--- middle

# Introduction

A number of path formats currently exist, such as JSON path and Xpath.
These path formats serve well for their initial use cases, however,
when considering a broader YANG generic path format that describes both
the path and the YANG models that the paths are described in are not covered
completely.

In addition, there is a trend towards using JSON for described data for YANG
instance data and the YANG path format in this document is a familiar format.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
