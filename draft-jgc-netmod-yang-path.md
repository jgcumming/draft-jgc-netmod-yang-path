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
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "YANG path format"
category: info

docname: draft-jgc-netmod-yang-path-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: NETMOD
workgroup: NETMOD Working Group
keyword:
 - YANG
 - path
 - JSON
 - JSON instance path
 - YANG path
venue:
  group: NETMOD
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: jgcumming/draft-jgc-netmod-yang-path
  latest: https://example.com/LATEST

author:
 -
    fullname: James Cumming
    organization: Nokia
    email: james.cumming@nokia.com

normative:

informative:

[RFC9535](https://datatracker.ietf.org/doc/rfc9535/)
[XPATH](http://www.w3.org/TR/1999/REC-xpath-19991116)

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
