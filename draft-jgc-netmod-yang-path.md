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

  RFC7950:

informative:

  RFC6241:
  RFC7951:
  RFC7952:
  RFC8040:
  RFC9535:
  XPATH:
    target: https://www.w3.org/TR/1999/REC-xpath-19991116/
    title: XML Path Language (XPath) Version 1.0
    author:
      - name: W3C
    date: false

...

--- abstract

This document defines ypath (YANG path), a single-line, self-describing path
format for referencing nodes in YANG schema trees, YANG instance data, and
data filters.  A ypath identifies YANG nodes using module-qualified names and
list key predicates.  The format is closely related to the YANG
`instance-identifier` built-in type but additionally supports schema paths,
filter wildcards, regular expression key matching, key value sets, and path
enumeration.  Ypath is intended for management APIs,
path enumeration tools, and filtering specifications where a compact,
human-readable representation of a YANG location is required.  Additional uses
for this path format can easily been forseen as path selection for streaming
telemetry and for YANG reference statements, such as `when` and `must` statements,
in future YANG versions.  This document
specifies the ypath syntax, formal grammar, and conformance requirements.  It
does not define a protocol, API, or encoding.  Selection based on the contents
of node values (other than list keys) is out of scope.

--- middle

# Introduction

A number of path formats currently exist to describe YANG modeled information.
XPath {{XPATH}} is used for constraints and derived values within YANG modules
{{RFC7950}}.  The YANG built-in type `instance-identifier` {{RFC7950}} defines
a path subset for referencing data tree nodes in instance encodings.  RESTCONF
{{RFC8040}} defines URI paths for accessing data resources.  JSONPath
{{RFC9535}} provides a query language for JSON documents, including those
produced by the JSON encoding of YANG data {{RFC7951}}.

These path formats serve well for their initial use cases.  However, some have
shortcomings for YANG module authors, tool implementers, and operators who need
a single, generic, human-readable path format that can refer equally to schema
locations, instance data, and filter expressions without the full expressiveness
(and complexity) of XPath or JSONPath.

There is a need for a self-describing path format that can be used to describe
schema data, instance data, and filtering in a consistent manner.  Deployed
implementations already use such a format (commonly referred to as a JSON
instance path) in management interfaces; however, no IETF specification
currently defines this format.

This document defines ypath (short for YANG path), a self-describing, generic
path format for referencing YANG schema, instance data, and filters.

## Applicability

Ypath is a string syntax for identifying locations in a YANG data tree.  It is
intended for use in specifications and implementations that need to:

- enumerate or display paths through a YANG schema;
- identify specific nodes in YANG instance data;
- express filter expressions that select data subsets; and
- convey path information in management protocol error reporting.

This document defines the ypath format only.  It does not specify how ypaths are
carried on the wire, stored, or processed by a particular protocol such as
NETCONF {{RFC6241}} or RESTCONF {{RFC8040}}.

## Out of Scope {#out-of-scope}

Ypath identifies nodes in a YANG schema or instance tree.  Predicates in filter
paths apply only to list key values.  The following are out of scope for this
document:

* selection or matching based on the value of any node other than a list key
  (for example, matching a `description` or `mtu` leaf value);
* comparison operators, ranges, or expressions over leaf or leaf-list values;
* content-based queries across the datastore (for example, "all interfaces
  where `enabled` is `true`"); and
* any form of value predicate attached to a non-key node in the path.

Such selection belongs in query languages (for example, XPath or JSONPath) or in
protocol-specific filter mechanisms, not in ypath.  A ypath MAY identify a
leaf node (for example, `/.../description`), but MUST NOT specify a condition on
that leaf's value.

**TODO: Whilst this section is the initial position, there are use-cases such as a
short-form subtree-filter representation for NETCONF that may be useful to include,
for example, all interfaces in a YANG list where the `enabled` child field is
set to `True`.

## Relationship to Other Path Formats {#relationship-to-other-path-formats}

Ypath is closely related to, but not identical with, the lexical representation
of the YANG `instance-identifier` built-in type defined in {{RFC7950}}.
Instance-identifiers are used to reference data tree nodes in instance
encodings and require key values for list entries.  Ypath additionally defines
a schema path form (with key names but not values) and filter forms (including
wildcards) that are outside the scope of `instance-identifier`.

Where ypath instance path syntax aligns with `instance-identifier`, the
definitions in {{RFC7950}} form the basis for interpretation unless this
document explicitly specifies otherwise.  The principal differences for instance
paths are:

- module names MAY be inherited along the path rather than repeated on every
node (see {{module-qualified-names}});
- numeric and boolean list key values MAY appear without quotes (see
{{lists-in-instance-data}}); and
- filter paths MAY use the wildcard `*` in key values (see {{wildcards}});
- filter paths MAY use regular expressions in key values (see
  {{regular-expressions}}); and
- filter paths MAY use key value sets to match multiple explicit key values
  (see {{key-value-sets}}).

Ypath is not a query language.  It does not provide the expression evaluation,
node set operations, or document traversal capabilities of XPath or JSONPath.

## Document Structure

{{conventions}} describes terms used in this document.  {{out-of-scope}}
states what ypath does not cover.  {{format-definition}} defines the ypath
format.  {{formal-syntax}} provides an ABNF grammar.
{{conformance}} defines conformance requirements.  {{examples}} gives worked
examples.  {{security-considerations}} discusses security implications.
{{iana-considerations}} specifies IANA actions.

# Conventions and Definitions {#conventions}

{::boilerplate bcp14-tagged}

## Terminology

The following terms are used in this document:

ypath:
: A path string conforming to the format defined in this document.  Short for
  YANG path.

schema path:
: A ypath that identifies a location in a YANG schema tree.  List key names
  appear in square brackets without values (for example, `[name]`).

instance path:
: A ypath that identifies a location in a YANG instance data tree.  List keys
  and their values appear in square brackets (for example,
  `[name="eth0"]`).

filter path:
: A ypath used to select a set of instance data nodes.  Filter paths use
  wildcards, regular expressions, or key value sets in one or more list key
  values only.  Predicates on non-key node values are out of scope (see
  {{out-of-scope}}).

module-qualified name:
: A YANG node identifier prefixed with the defining module name and a colon
  (for example, `ietf-interfaces:interfaces`).

path segment:
: A single component of a ypath, consisting of a node name and optional key
  predicates, separated from adjacent segments by a `/` character.

The terminology for YANG data nodes (for example, leaf, list, container) is
defined in {{RFC7950}}.

# Format Definition {#format-definition}

This section defines the ypath format.  The normative grammar is given in
{{formal-syntax}}.

The ypath format applies to YANG schema information, YANG instance data, and
filter expressions.  Predicates in filter paths are limited to list key values;
selection based on any other node value is out of scope (see {{out-of-scope}}).

## Root of the YANG Path

A ypath is always expressed from the root of the YANG data tree.  A
specification that splits a path into a prefix and a sub-path MUST evaluate
both parts together from the root.

Approach 1:

- Path: `/item1/item2/item3/item4`

Approach 2:

- Prefix: `/item1/item2`
- Sub-path: `/item3/item4`
- (Yields) Path: `/item1/item2/item3/item4`



## Module-Qualified Names {#module-qualified-names}

A ypath uses a `module:identifier` form where `module` is the name of the YANG
module that defines the node.  For example:

```
/ietf-interfaces:interfaces
```

**TODO: Do we need to handle different module versions/semantic versions in the path?**

The module name is inherited along the path from left to right.  If the
defining module does not change (as is typical within a single module, and
unlike at augment boundaries) the module name does not need to be repeated.
For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets
```

It is also valid to provide the module name on every path segment.  For example:

```
/ietf-routing-policy:routing-policy/ietf-routing-policy:defined-sets/ietf-routing-policy:prefix-sets
```

The first path segment of a ypath SHOULD include a module name.  This ensures
that the path is self-describing without external schema context.  A
specification that allows omission of the module name on the first segment
MUST state the default module to be assumed.

**TODO: Is the SHOULD correct here or should it be a MUST?**

## Augmentations

When a node is defined by an augmenting module, the module name in the path
MUST change to the augmenting module at the augmented node.  For example:

```
/ietf-routing:routing/control-plane-protocols/ietf-ospf:ospf/address-family
```

In this example, `control-plane-protocols` is defined in `ietf-routing` and
`ospf` is defined in `ietf-ospf`, which augments `ietf-routing`.

The following fully qualified form is equally valid:

```
/ietf-routing:routing/ietf-routing:control-plane-protocols/ietf-ospf:ospf/ietf-ospf:address-family
```

## Deviations

Deviations do not change the namespace of YANG nodes.  The module portion of a
ypath therefore does not change at a deviated node.

## Import/Include

Imports and includes do not change the namespace of YANG nodes.  The module
portion of a ypath therefore does not change solely because a node is accessed
through an import or include relationship.

## Containers

### Presence Containers

A ypath to a presence container references the container node directly:

```
/module:node1/node2/container1/presence-container1
```

In instance data, the path exists only when the presence container has been
instantiated.  In schema paths, the container is always addressable.

### Non-Presence Containers

A ypath to a non-presence container references the container node directly:

```
/module:node1/node2/container1/non-presence-container1
```

In instance data, the path exists when any child node has been instantiated.
In schema paths, the container is always addressable.

## Leaves

A path to a leaf uses the same final segment in schema and instance paths.  The
parent path segments differ between schema and instance forms when lists appear
above the leaf.

Schema example:

```
/ietf-interfaces:interfaces[name]/description
```

Instance example:

```
/ietf-interfaces:interfaces[name="my_interface"]/description
```



## Choices

YANG choice nodes are not instantiated in the data tree.  A ypath therefore does
not include a segment for the choice node itself.  The path proceeds directly
to the node within the selected case.

For example, if `protocol` is a choice under `server` and the `tcp` case is
active, the path to the `port` leaf is:

Schema path:

```
/example:server/tcp/port
```

Instance path:

```
/example:server/tcp/port
```

When walking a schema tree to enumerate paths, implementations MUST NOT emit a
path segment for the choice node.

## Identities and Identity References (identityref)

An `identityref` leaf is referenced by a path to the leaf itself.  The identity
value is not encoded in the path; it appears in the instance data value.

Schema path:

```
/ietf-routing:routing/control-plane-protocols/control-plane-protocol[type]/type
```

Instance path:

```
/ietf-routing:routing/control-plane-protocols/control-plane-protocol[name="main"]/type
```

Where the leaf value uses the module-qualified identity name (for example,
`ietf-ospf:ospf`) in the data encoding, that value is not duplicated in the
ypath.

## Lists

List handling differs between schema paths and instance paths.

### Lists in Schema

A schema path to a list names the list node.  To address the list keys, each key
name appears in its own bracket pair without a value.  For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set[name]
```

For multi-key lists:

```
/module:node1/node2/list[key1][key2]
```

When enumerating schema paths, a path to a list entry MUST include all key
names in brackets.  A path that uses the key name as a child segment is
invalid.  For example, the following is not a valid schema path:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set/name
```



### Lists in Instance Data {#lists-in-instance-data}

An instance path to a list entry includes each key and its value in square
brackets.  For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set[name="loopbacks"]
```

A leaf under that list entry:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set[name="loopbacks"]/mode
```

For multi-key lists, each key appears in a separate bracket pair:

```
/module:node1/node2/list[key1="value1"][key2="value2"]
```

Key values that are strings MUST be enclosed in double quotes.  Key values
that are numeric or boolean MAY appear without quotes, provided that the
unquoted form is unambiguous.  For example:

```
/openconfig-interfaces:interfaces/interface[name="1/1/1"]/subinterfaces/subinterface[index=0]
```



## Leaf Lists

A schema path to a leaf-list names the leaf-list node:

```
/ietf-system:system/dns/servers
```

An instance path to a specific leaf-list entry uses the same predicate form as
`instance-identifier` {{RFC7950}}:

```
/ietf-system:system/dns/servers[.="192.0.2.1"]
```

An instance path that references the leaf-list node without selecting a
particular entry names the leaf-list node without a predicate.

## Metadata Annotations

Metadata annotations defined in {{RFC7952}} are not part of the ypath syntax.
A ypath identifies a data tree node only; it does not reference metadata
annotations attached to nodes.

## Actions

A ypath to a YANG action names the action node.  For example:

```
/example:mycontainer/do-something
```

The action `input` and `output` nodes are not included in the path used to
invoke the action.  Input parameters are supplied separately in the protocol or
API operation that invokes the action.

## Wildcards {#wildcards}

Filter paths MAY use a wildcard, a regular expression (see
{{regular-expressions}}), or a key value set (see {{key-value-sets}}) in list
key predicates.  These forms apply only to filter paths; they are not used in
schema paths.

For string-typed keys, the wildcard MUST appear inside double quotes:

```
/ietf-interfaces:interfaces[name="*"]
```

For numeric or boolean keys, the unquoted form MAY be used:

```
/example:items/item[index=*]
```

If any key in a multi-key list uses a wildcard, all keys in that list entry
MUST use wildcards.  Mixing wildcards and specific values for different keys of
the same list entry is not permitted.  For example:

Valid:

```
/module:list[key1="*"][key2="*"]
```

Not valid:

```
/module:list[key1="foo"][key2="*"]
```

## Regular Expressions {#regular-expressions}

Filter paths MAY use a regular expression as a list key value to match multiple
list entries.  Regular expressions apply only to list key predicates in filter
paths.  They MUST NOT be used in schema paths or in instance paths that identify
a single known node.

A regular expression key value uses the form `r'pattern'`, where `pattern` is
the regular expression body enclosed in single quotes.  For example, to select
the `description` leaf on all interfaces whose name begins with `a` or `A`:

```
/ietf-interfaces:interfaces/interface[name=r'^[aA].*']/description
```

The `r` prefix distinguishes a regular expression from a literal string value.
The pattern is matched against the string representation of the list key value
in the data encoding used by the implementation (for example, the JSON
encoding defined in {{RFC7951}}).

Regular expression key values MUST use the `r'...'` form.  Double-quoted
strings MUST NOT be used to encode regular expressions.

### Regular Expression Dialect

The regular expression dialect MUST be POSIX Extended Regular Expressions (ERE)
as defined in Section 9.3.6 of IEEE Std 1003.1-2008.  Implementations MAY
support additional regular expression dialects only if the enclosing
specification or API documents the dialect in use.

### Escaping

Within the single-quoted pattern, a single quote character is escaped as `\'`
and a backslash is escaped as `\\`.  All other characters are treated
literally.

### Multi-Key Lists

Unlike wildcards, regular expressions MAY be combined with literal key values
or other regular expressions in different key predicates of the same list entry.
For example:

```
/module:routes/route[ip-prefix=r'^192\.0\.2\.'][route-type="unicast"]
```

### Leaf Lists

Regular expression matching is defined for list keys only.  Leaf-list entry
selection continues to use literal values with the `instance-identifier` form
(for example, `[.="value"]`) or wildcards if supported by the enclosing
specification.

## Key Value Sets {#key-value-sets}

Filter paths MAY use a key value set to match a list entry when the key value
equals any one of a given set of explicit values.  Key value sets apply only to
list key predicates in filter paths.  They MUST NOT be used in schema paths or
in instance paths that identify a single known node.

A key value set uses curly braces enclosing a comma-separated list of key
values.  For example, to select the `description` leaf on interfaces named
`ethernet1` or `ethernet3`:

```
/ietf-interfaces:interfaces/interface[name={"ethernet1", "ethernet3"}]/description
```

An implementation matches the list entry if the key value is equal to any
member of the set.  The comparison uses the same string representation of key
values as instance paths (see {{lists-in-instance-data}}).

### Set Member Values

Each member of a key value set is a literal key value.  Members MAY be
expressed as a double-quoted string or, if the value contains only characters
valid for an unquoted value, without quotes.  For example:

```
/ietf-interfaces:interfaces/interface[name={"ethernet1", "ethernet3"}]/description
/module:items/item[id={1, 2, 3}]
/module:items/item[name={"foo", "bar"}]
```

Wildcards, regular expressions, and nested key value sets MUST NOT appear as
set members.

### Multi-Key Lists

A key value set applies to a single key predicate.  Different keys in a
multi-key list MAY each use their own literal value, regular expression,
wildcard, or key value set independently.  For example:

```
/module:routes/route[ip-prefix={"192.0.2.1/32", "192.0.2.2/32"}][route-type="unicast"]
```

### Leaf Lists

Key value sets are defined for list keys only.  Leaf-list entry selection is
not defined for key value sets in this document.

## Relationship to JSON Encoding

Considering the instance path
`/ietf-routing:routing/control-plane-protocols/ietf-ospf:ospf/address-family`
with a value of `ipv4`, the corresponding JSON based on {{RFC7951}} is:

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



# Formal Syntax {#formal-syntax}

The grammar below uses ABNF as defined in {{RFC7950}}, Section 14.  The rules
`identifier`, `identifier-ref-arg`, and `quoted-string` are used as defined
there.

~~~~ abnf
{::include formal-syntax.abnf}
~~~~

A `filter-predicate` is an `instance-predicate` in which one or more
`key-value` elements are a `wildcard`, a `quoted-string` containing only `*`,
a `regex-value`, or a `key-value-set`.

# Conformance {#conformance}

This section defines conformance requirements for specifications and
implementations that use ypath.

## Producers

A ypath producer (for example, a tool that emits schema paths or an interface
that returns the current context path) MUST:

- generate paths that conform to the grammar in {{formal-syntax}};
- use a leading `/` on every path;
- include a module name on the first path segment unless a specification
defines a default module for the context; and
- use schema predicates for schema paths and value predicates for instance
paths.

A producer generating filter paths MUST use `*` only as specified in
{{wildcards}}, regular expressions only as specified in
{{regular-expressions}}, and key value sets only as specified in
{{key-value-sets}}.

## Consumers

A ypath consumer (for example, a management API or path parser) MUST:

- reject paths that do not conform to the grammar in {{formal-syntax}};
- resolve module inheritance from left to right along the path; and
- apply module name changes at augment boundaries.

A consumer that does not support filter paths MUST reject paths containing the
wildcard `*`, a `regex-value`, or a `key-value-set` in key predicates.

## Path Enumeration

An implementation that walks a YANG schema and returns ypaths MUST return
schema paths with list key names in bracket pairs and MUST NOT return paths
that address list keys as child node segments.

# Examples {#examples}

This section provides non-normative examples of ypath usage.

## Referencing YANG Paths in NETCONF Filters

NETCONF subtree filters {{RFC6241}} select data by structure rather than by a
single path string.  A ypath can nonetheless identify the subtree root that a
filter targets.  For example, to retrieve all interfaces, a client might use
the filter path:

```
/ietf-interfaces:interfaces[name="*"]
```

The enclosing specification or implementation maps this ypath to the
appropriate NETCONF filter payload.

## Filter Paths with Regular Expressions

A filter path can use a regular expression in a key predicate to match a
subset of list entries.  For example, to retrieve the `description` leaf for
all interfaces whose name starts with `a` or `A`:

```
/ietf-interfaces:interfaces/interface[name=r'^[aA].*']/description
```

An implementation evaluates the regular expression against each candidate list
key value and includes matching entries in the result set.

## Filter Paths with Key Value Sets

A filter path can use a key value set to match multiple list entries with
explicit key values.  For example, to retrieve the `description` leaf for
interfaces `ethernet1` and `ethernet3`:

```
/ietf-interfaces:interfaces/interface[name={"ethernet1", "ethernet3"}]/description
```

An implementation includes each list entry whose key value equals any member
of the set.

## Referencing YANG Paths in NETCONF rpc-error Messages

NETCONF `rpc-error` replies include an `error-path` element {{RFC6241}} that
identifies the node associated with the error.  An implementation may
populate `error-path` using ypath instance syntax.  For example:

```
/ietf-interfaces:interfaces[name="eth0"]/mtu
```



## Checking Existence

An implementation can test whether a node exists in a datastore by resolving an
instance path.  For example, the path:

```
/ietf-interfaces:interfaces[name="eth0"]
```

identifies a specific list entry.  If the path resolves to an existing node,
the entry is present; if resolution fails, the entry is absent.  The mechanism
by which resolution is performed is defined by the enclosing API or protocol.

# Security Considerations {#security-considerations}

This section discusses security considerations for implementations and
specifications that use ypath strings.  Ypath itself is a path syntax and does
not define authentication, authorization, or transport security; those
properties depend on the enclosing protocol or API.

## Path Parsing and Ambiguity

Implementations that parse ypath strings MUST treat parsing as security
sensitive when the resulting path is used to access or modify managed data.
Ambiguous, malformed, or unexpectedly complex paths could cause an
implementation to resolve a different node than the operator or application
intended.  Specifications that use ypath SHOULD define error handling for
invalid paths and SHOULD NOT silently accept ambiguous forms (for example,
inconsistent module prefix usage that could identify more than one schema
node).

## Wildcards and Pattern Matching

Filter paths that use wildcards, regular expressions, or key value sets in key
predicates can match large sets of instance data.  An implementation that
expands such a path into concrete instance paths or data retrieval operations
MUST consider the cost of expansion and SHOULD enforce limits on the number of
matched nodes.  Implementations SHOULD enforce a reasonable upper bound on the
number of members in a key value set.

Regular expression evaluation can be subject to excessive resource consumption
(including regular expression denial of service) if arbitrary patterns are
accepted from untrusted input.  Implementations SHOULD enforce limits on
evaluation time and SHOULD reject patterns that are not valid POSIX ERE
expressions.  Specifications that expose regular expression filter paths to
untrusted clients SHOULD document these limits.

## Information Disclosure

Path enumeration (for example, listing all valid ypaths for a schema or
datastore) can reveal the structure of deployed models and, for instance paths,
the values of list keys.  Interfaces that expose path lists SHOULD apply the
same access controls as the underlying data access mechanism so that a client
cannot use path enumeration to discover information it is not authorized to
retrieve directly.

## Use in Authorization Policies

If ypaths are used to express authorization rules (for example, permitting
access only to a given subtree), care is required to ensure that equivalent
paths using different but valid module prefix forms receive consistent
treatment.  Authorization systems SHOULD define a canonical comparison rule or
SHOULD normalize paths before evaluation.

## Privacy

Instance paths embed key values that may identify subscribers, endpoints, or
other privacy-sensitive entities.  Logs, error messages, and telemetry that
include ypath strings SHOULD be protected commensurate with the sensitivity of
the referenced data.

# IANA Considerations {#iana-considerations}

This document has no IANA actions.  Ypath is a string syntax for identifying
YANG schema and instance locations.  It does not define a URI scheme, XML
namespace, YANG module, media type, or other protocol element requiring IANA
registration.

--- back

{:numbered="false"}

# Acknowledgments

The author would like to thank the Network Modeling (NETMOD) working
group for its discussion of YANG path formats.



