---
title: "YANG path format (ypath)"
category: std

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

  RFC7952:
  RFC9535:
  XPATH:
    target: https://www.w3.org/TR/1999/REC-xpath-19991116/
    title: XML Path Language (XPath) Version 1.0
    author:
      - name: W3C
    date: false

...

--- abstract

This draft provides a common path format, the YANG path (ypath), to
provide a self describing method of detailing YANG modelled paths
(both in schema and in instance data).

--- middle

# Introduction

A number of path formats currently exist to describe YANG modelled information,
such as JSON path and Xpath. These path formats serve well for their initial
use cases, however, some of these path formats have shortcomings for YANG
module and tool designers and users and some provide significantly extended
functionality that is not applicable to common use-cases.

There is a need to provide a self-describing single-line generic YANG path
format that can used to describe schema data, instance data and filtering.

This draft defines a self-describing, generic path format for referencing YANG
schema, instance data and filters named ypath (short for YANG path).

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Format definition

This section will define the YANG path format using a number of examples to cover
each situation.

The YANG path format is designed to be applicable to both YANG schema information,
YANG instance data and for the definition of filters.

## Root of the YANG path

The YANG path format is always provided from the root of the YANG tree.  It is
possible that a specification relying on a YANG path may choose to split a path
into two parts; a prefix and a sub-path, however, these MUST always be evaluated
together from the root.

Approach 1:

- Path: /item1/item2/item3/item4

Approach 2:

- Prefix: /item1/item2
- Sub-Path: /item3/item4
- (Yields) Path: /item1/item2/item3/item4

## Module:Field format

The YANG path format uses a `module:field` format where the `module`
is the name of the YANG module.  For example:

```
/ietf-interfaces:interfaces
```

The module is inherited along the path from left to right.  That means that if
the module does not change (as occurs in augments which are discussed later in this
document) the module does not need to be provided again.  For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets
```

It is also valid to provide the module on every element of the path if desired.  For
example:

```
/ietf-routing-policy:routing-policy/ietf-routing-policy:defined-sets/ietf-routing-policy:prefix-sets
```

## Augmentations

When a YANG module is augmented into another the namespace changes.  This is represented
in the YANG path by a change in the `module` field.

```
/ietf-routing:routing/control-plane-protocols/ietf-ospf:ospf/address-family
```

In this example the module inheritance approach remains, the `control-plane-protocols` container
is in the `ietf-routing` YANG module and the `address-family` leaf is in the `ietf-ospf` YANG
module that has been augmented into the `ietf-routing` module.

As mentioned in the `Module:Field` section of this document, the following path in this example
is also valid:

```
/ietf-routing:routing/ietf-routing:control-plane-protocols/ietf-ospf:ospf/ietf-ospf:address-family
```

## Deviations

Deviations do not change the namespace of the YANG nodes and therefore the `module`
portion of the YANG path does not change.

## Import/Include

Imports and includes do not change the namespace of the YANG nodes and therefore the `module`
portion of the YANG path does not change.

## Containers

### Presence containers

### Non-presence containers

## Leafs

## Choice

## Identities and identity references (identityref)

## Lists

When considering the YANG path for lists it is important to consider both schema
and instance data as these are represented in slightly different ways.

### Lists in schema

YANG lists in schema are represented in the YANG path to the top of the list.
For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set
```

The provide the path to the key of a list identify the key inside square
brackets. For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set[name]
```

For multi-key lists these are represented as shown:

```
/module:node1/node2/list[key1][key2]
```

When walking a YANG tree to provide a list of paths, lists are provided with the
keys in square brackets as shown above.  The following path is invalid and will not
be returned:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set/name
```

### Lists in instance data

YANG lists in instance data are represented in the YANG path to the top of the list with the
list key and the list key's value being provided in square brackets.  For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set[name="loopbacks"]
```

This means that the YANG path to a leaf under the `loopbacks` list entry would be:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set[name="loopbacks"]/mode
```

## Leaf lists

The YANG path to a leaf-list is the path to the top of the list.

## Metadata annotations

Metadata annotations as defined in {{RFC7952}} are applied to items in a path.  The YANG path
does not support referencing any metadata annotations attached to any element in the path.

## Actions

### Input

### Output

## Wildcards/Regex

When using the YANG path format, it is possible to reference data using wildcards and regular
expressions.

```
r' '
m' '
```

TODO: Provide details of which regexp semantics and detail what is applicable for simple
wildcarding and what is out of scope.

## Similarity to JSON

Considering this path `/ietf-routing:routing/control-plane-protocols/ietf-ospf:ospf/address-family`
with a value of `ipv4`, the translation to JSON based on {{RFC7952}} would be:

```
{
  "ietf-routing:routing": {
    "control-plane-protocols": {
      "ietf-ospf:ospf": {
        "address-family": "ipv4"
      }
    }
  }
}
```

## Examples

### Referencing YANG paths in NETCONF filters

### Referencing YANG paths in NETCONF rpc-error messages

### Checking existence

# Security Considerations

TODO: Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
