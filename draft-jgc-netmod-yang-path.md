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

 -
    fullname: "Robert Wilton"
    organization: Cisco
    email: "rwilton@cisco.com"

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
enumeration.

--- middle

# Introduction

A number of path formats are used with YANG.  The YANG built-in type
`instance-identifier` {{RFC7950}} identifies data nodes, while XPath is used by
YANG constraint statements.  These formats have different purposes and
semantics.  This document defines a concise, self-describing syntax for naming
nodes in a YANG schema tree or instance data tree.  It is intended to make
paths clear and consistent.  A path still needs to be resolved against a YANG
schema.

This document defines ypath (short for YANG path).  It specifies absolute and
relative paths to schema and instance nodes.  It also defines a limited filter
path extension for selecting list entries by their keys.  This extension is
optional and does not define a general query language.  Ypath does not replace
XPath in YANG 1.1 `must` and `when` statements, and this document does not
define an expression language for those statements.  A future YANG revision
could define an expression language that uses ypath paths; doing so would need
to specify expression semantics separately.

## Applicability

This document defines the ypath syntax for locations in a YANG schema tree or
instance data tree, including an optional filter path extension.  It does not
specify how ypaths are carried, stored, or processed by a protocol or API.
Appendix {{potential-uses}} describes possible applications; any protocol
binding must be defined by the relevant specification.

## Out of Scope {#out-of-scope}

Ypath identifies nodes in a YANG schema or instance tree.  Predicates in
filter paths apply only to list key values.  An instance path MAY identify an
individual leaf-list entry by its exact value (see {{leaf-lists}}); no other
non-key node value is used as a predicate.  The following are out of scope for
this document:

* selection or matching based on the value of any non-key leaf (for example,
  matching a `description` or `mtu` leaf value);
* comparison operators, ranges, or expressions over leaf or leaf-list values;
* content-based queries across the datastore (for example, "all interfaces
  where `enabled` is `true`"); and
* any form of value predicate attached to a non-key node in the path.

Such selection belongs in expression or query languages, not in ypath.  A
ypath MAY identify a leaf node (for example, `/.../description`), but MUST NOT
specify a condition on that leaf's value.  This restriction concerns what a
path predicate may test; it does not prevent a `must` or `when` expression
from referring to other leaf nodes.

Ypath does not select nodes based on values of non-key nodes.  This restriction
keeps ypath a path syntax; selecting nodes based on arbitrary data values
requires an expression language.  Whether a future version should add such an
expression language is left for separate work.

## Relationship to Other Path Formats {#relationship-to-other-path-formats}

Ypath instance paths identify YANG data nodes and share concepts with the YANG
`instance-identifier` built-in type defined in {{RFC7950}}.  Unlike
`instance-identifier`, ypath also defines paths to schema nodes and relative
paths.  Filter path extensions are described in {{filter-paths}}.  Appendix
{{potential-uses}} records possible applications of ypath.

## Document Structure

{{conventions}} describes terms used in this document.  {{out-of-scope}}
states what ypath does not cover.  {{format-definition}} defines the ypath
format, including relative paths and the optional filter path extension.
{{formal-syntax}} provides an ABNF grammar.
{{conformance}} defines conformance requirements.  {{examples}} gives worked
examples.  {{security-considerations}} discusses security implications.
{{iana-considerations}} specifies IANA actions.  Appendix {{potential-uses}}
describes potential applications of ypath.

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

A ypath MAY be absolute or relative.  An absolute path begins at the root and
starts with `/`.  A relative path is interpreted from the context node defined
by the specification or application using it.  A relative path MUST NOT be
interpreted without such a context.  A consumer that supports only absolute
paths MUST reject relative paths.

When a specification splits a path into a prefix and a sub-path, it MUST define
the context for the sub-path and how the two parts are combined.  If the
sub-path is defined relative to the prefix, the consumer MUST resolve the
resulting path as if it had been evaluated from the root.  This rule ensures
that splitting a path does not change the node it identifies.

## YPath Expressions {#ypath-expressions}

This document defines path references, not expressions.  In YANG 1.1,
`must` and `when` continue to use XPath as specified by {{RFC7950}}.  A ypath
MUST NOT be substituted for an XPath expression in those statements.  A future
YANG revision could define expressions that use ypath as a path-reference
syntax, but it would also need to define relative context, navigation,
operators, type conversion, and evaluation semantics.  Those expression
semantics are outside the scope of this document.  Relative ypaths defined
here provide a possible path-reference building block; by themselves, they do
not define such an expression language.



## Module-Qualified Names {#module-qualified-names}

A ypath uses a `module-name:identifier` form where `module-name` is the name of
the YANG module that defines the node.  This follows the module-qualified
identifier terminology in {{RFC7950}}; the separator is a colon, not a hyphen.
For example:

```
/ietf-interfaces:interfaces
```

The module name is inherited along the path from left to right.  If the
defining module does not change (as is typical within a single module, and
unlike at augment boundaries) the module name does not need to be repeated.
For example:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets
```

It is valid to provide the module name on every path segment.  For example:

```
/ietf-routing-policy:routing-policy/ietf-routing-policy:defined-sets/ietf-routing-policy:prefix-sets
```

An absolute path MUST include a module name on its first segment.  A relative
path MAY omit the module name on its first segment when the context node
provides an unambiguous module.  A specification that permits another default
module MUST define it.  A producer SHOULD include a module name at the first
segment and at each subsequent module change; it MAY include the module name on
every segment.

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
portion of a ypath therefore remains the module that defines the node.  An
identityref value is different: the identity name in the value is qualified by
the module that defines that identity, even when the identity is used through
an import.  That value qualification does not change the namespace of the
leaf node identified by the ypath.

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

YANG choice and case nodes are not instantiated in the data tree.  A ypath
therefore does not include a segment for the choice or case node when considering
instance-data.  The path proceeds directly to the node within the selected case.

When walking a schema tree to enumerate paths, implementations MUST emit a
path segment for each explicitly declared `choice` and `case` node.  A
shorthand case as described in Section 7.9.2 of {{RFC7950}} is implicit and
does not add a path segment; the child node is named directly.

For example, consider the following YANG model:

```
module example {
  yang-version 1.1;
  namespace "urn:example:example";
  prefix "ex";
  revision "2026-08-12";
  container server {
    choice protocol {
      case tcp {
        container tcp {
          leaf port {
            type uint8;
          }
        }
      }
      case udp {
        container udp {
          leaf port {
            type uint8;
          }
        }
      }
    }
  }
}
```

Instance paths:

```
/example:server/tcp/port
/example:server/udp/port
```

Schema paths:

```
/example:server/protocol/tcp/tcp/port
/example:server/protocol/udp/udp/port
```

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

Schema paths identify schema nodes, not list entries.  A list key is an
ordinary child schema node and is addressed by its child segment, just like
other leaves.  For example, the following are valid schema paths:

```
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set
/ietf-routing-policy:routing-policy/defined-sets/prefix-sets/prefix-set/name
```

Square-bracket key predicates MUST NOT be used in schema paths.  They are used
only to identify or select list entries in instance and filter paths.



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
whose YANG type is numeric (including `decimal64`) or boolean MAY appear
without quotes, provided that the unquoted form is unambiguous.  A consumer
uses the schema to determine the key type.  For example:

```
/openconfig-interfaces:interfaces/interface[name="1/1/1"]/subinterfaces/subinterface[index=0]
```

### Keyless lists {#keyless-lists}

A keyless list in a schema path is addressed by its node name, without a key
predicate:

Consider the `ietf-isis@2022-10-19.yang` model and the `adjacency-state` grouping:

```
container adjacencies {
  config false;
  list adjacency {
    leaf neighbor-sys-type {
      type level;
      ...
    }
    leaf neighbor-sysid {
      type system-id;
      ...
    }
    ...
    description
      "List of operational adjacencies.";
  }
}
```

In this model, `adjacency` is a keyless list.  Therefore, the ypath to the
`neighbor-sys-type` is:

```
/ietf-routing:routing/control-plane-protocols/control-plane-protocol[name]/ietf-isis:isis/interfaces/interface[name]/adjacencies/adjacency/neighbor-sys-type
```

A keyless list in instance data may be addressed by its node name alone when
the path identifies the list node as a whole.  To identify a particular entry,
a positional predicate MAY be used.  The position is a positive integer, as in
the keyless-list form of `instance-identifier` defined in Section 9.13 of
{{RFC7950}}:

```
/ietf-routing:routing/control-plane-protocols/control-plane-protocol[name="isis1"]/ietf-isis:isis/interfaces/interface[name="ethernet1"]/adjacencies/adjacency[1]/neighbor-sys-type
```

The position is one-based and refers to the order of the list entries in the
instance data.  A filter path MUST NOT use a positional predicate because its
selection semantics could change when entries are inserted or removed.

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

A `leaf-list-predicate` accepts a literal value only (`quoted-string` or
`unquoted-value`).  Wildcards, regular expressions, and key value sets MUST
NOT be used in leaf-list predicates.

**TODO: This is a first approach to this issue.  It could be considered out-of-scope
to identify specific leaf-list entries or this same approach could be used to allow for
matching of other specific values (such as leafs) as a solution to that problem**

## Metadata Annotations

Metadata annotations defined in {{RFC7952}} are not part of the ypath syntax.
A ypath identifies a data tree node only; it does not reference metadata
annotations attached to nodes.

## Actions

A schema path MAY identify an `action`, `rpc`, or `notification` node, in
addition to data nodes.  These are schema nodes and do not appear as
instantiated nodes in the data tree.  An instance path MUST NOT address an
`rpc` or `notification`.  An instance path to an `action` identifies the
action node to invoke; the protocol operation still defines how to invoke it.
For example:

```
/example:mycontainer/do-something
```

For a schema path that continues below an `action` or `rpc`, its `input` and
`output` nodes are explicit path segments.  For example,
`/example:mycontainer/do-something/input/name` and
`/example:mycontainer/do-something/output/name` identify distinct schema
nodes.

## Filter Paths {#filter-paths}

Filter paths are an optional extension to instance paths.  They select
instance data nodes by matching list key values.  A specification that uses
filter paths MUST define how selected nodes are returned or acted upon.  This
document defines no transport or filter payload.

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

Each key in a multi-key list is matched independently.  A wildcard MAY be
combined with a literal, regular expression, or key value set in another key
predicate.  For example, `/module:list[key1="foo"][key2="*"]` is valid.

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

**TODO: Consider UTF-8 characters and if they need a special mention or special
handling**

### Multi-Key Lists

Unlike wildcards, regular expressions MAY be combined with literal key values
or other regular expressions in different key predicates of the same list entry.
For example:

```
/module:routes/route[ip-prefix=r'^192\.0\.2\.'][route-type="unicast"]
```

### Leaf Lists

Regular expression matching is defined for list keys only.  Leaf-list entry
selection uses literal values with the `instance-identifier` form (for
example, `[.="value"]`).

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

# Formal Syntax {#formal-syntax}

The grammar below uses ABNF as defined in {{RFC7950}}, Section 14.  A ypath is
either an absolute path or a relative path.  Relative paths use `.` for the
context node and `..` for its parent; their resolution context is defined by
the specification or application using them (see {{ypath-expressions}}).  The rules
`identifier`, `node-identifier`, `quoted-string`, `WSP`, `SQUOTE`, and
`DIGIT` are used as defined there.  Path segments and key names in
predicates use `node-identifier`, as in the `instance-identifier` type in
{{RFC7950}}, and MAY include a module prefix (for example,
`/prefix:node` or `[prefix:key="value"]`).  An `unquoted-value` is a numeric
or boolean key value (see {{lists-in-instance-data}}); all other key values
use `quoted-string`.

~~~~ abnf
{::include formal-syntax.abnf}
~~~~

A `filter-predicate` is syntactically identical to an `instance-predicate`.
Filter-specific `key-value` forms (wildcards, regular expressions, and key
value sets) are defined in {{wildcards}}, {{regular-expressions}}, and
{{key-value-sets}}.  Literal key values remain valid in filter paths unless a
rule in those sections forbids them (for example, wildcard mixing in
{{wildcards}}).  A `leaf-list-predicate` uses `leaf-list-key-value`, which
permits only `quoted-string` or `unquoted-value`; filter extensions MUST NOT
be used in leaf-list predicates.  A positional predicate contains a one-based
list entry position and MUST be used only with a keyless list in an instance
path (see {{keyless-lists}}).

# Conformance {#conformance}

This section defines conformance requirements for specifications and
implementations that use ypath.

## Producers

A ypath producer (for example, a tool that emits schema paths or an interface
that returns the current context path) MUST:

- generate paths that conform to the grammar in {{formal-syntax}};
- use an absolute path unless the specification defines a relative path
context;
- include a module name on the first segment of an absolute path and at module
changes; and
- use child node segments for schema paths and key or position predicates only
on instance and filter paths.

A producer generating filter paths MUST use `*` only as specified in
{{wildcards}}, regular expressions only as specified in
{{regular-expressions}}, and key value sets only as specified in
{{key-value-sets}}.

## Consumers

A ypath consumer (for example, a management API or path parser) MUST:

- reject paths that do not conform to the grammar in {{formal-syntax}};
- resolve module inheritance from left to right along the path; and
- apply module name changes at augment boundaries; and
- resolve relative paths only when the applicable context is defined.

A consumer that does not support filter paths MUST reject paths containing the
wildcard `*`, a `regex-value`, or a `key-value-set` in key predicates.

## Path Enumeration

An implementation that walks a YANG schema and returns ypaths MUST return
schema paths with list keys as child node segments and MUST NOT return schema
paths with key predicates.

# Examples {#examples}

This section provides non-normative examples of ypath syntax.

## Filter Paths with Regular Expressions

A filter path can use a regular expression in a key predicate to match a
subset of list entries.  For example:

```
/ietf-interfaces:interfaces/interface[name=r'^[aA].*']/description
```

An implementation evaluates the regular expression against each candidate list
key value and includes matching entries in the result set.

## Filter Paths with Key Value Sets

A filter path can use a key value set to match multiple list entries with
explicit key values.  For example:

```
/ietf-interfaces:interfaces/interface[name={"ethernet1", "ethernet3"}]/description
```

An implementation includes each list entry whose key value equals any member
of the set.

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

# Appendix A. Potential Uses of Ypath {#potential-uses}

**RFC Editor:** Please remove this appendix before publication.

This appendix records possible applications of ypath to guide discussion of
future work.  It is non-normative.  Listing an application here does not define
its protocol binding, processing model, or conformance requirements.

## Identifying a Specific Instance-Data Node

An instance path can identify one node in YANG instance data.  For a path to
identify one specific list entry, every list in the path MUST have all of its
keys specified with valid values for the corresponding YANG types.  Filter
predicates such as wildcards, regular expressions, and key value sets select
sets of entries and therefore do not identify a single entry.

One possible use is for a YANG Push notification to identify the data node
object being returned.  Another use already illustrated in this document is
identifying the node associated with a NETCONF `rpc-error` `error-path`
({{RFC6241}}), for example:

```
/ietf-interfaces:interfaces/interface[name="eth0"]/mtu
```

An API can also use an instance path to check whether a node exists by
resolving the path against a datastore.  The API defines the resolution
mechanism and how it reports whether a node is present.

## Identifying a Set of Instance-Data Nodes

A filter path can identify a set of entries by matching list keys.  A possible
use is a YANG Push version 2 subscription filter.  YANG Packages could use
ypath to identify schema mount points or to express a `parent-reference`.
Specifications defining these uses would need to specify the expected path
form and how matching nodes are used.

The document's NETCONF filter example illustrates a related use: a ypath can
name the root targeted by a NETCONF subtree filter {{RFC6241}}, while the
NETCONF specification or implementation defines the filter payload and
processing:

```
/ietf-interfaces:interfaces[name="*"]
```

## Schema-Path Enumeration and Display

Implementations can enumerate or display schema paths for tools, diagnostics,
or operator interfaces.  The conformance requirements for schema-path
enumeration are specified in {{conformance}}.

## Future YANG Expressions

A future YANG revision might use ypath as the node-reference syntax within
`must2` or `when2` expressions.  Such a feature would require a more complete
expression language, including rules for navigation, operators, type
conversion, and evaluation.  This document defines no such language; YANG 1.1
`must` and `when` continue to use XPath (see {{ypath-expressions}}).

## Other Possible Uses

Ypaths could be used in authorization policies to identify the nodes to which
a rule applies.  The security considerations in {{security-considerations}}
discuss path normalization for this use.  These and the other applications
listed above are examples for discussion, not endorsements or protocol
specifications.

--- back

{:numbered="false"}

# Acknowledgments

The author would like to thank the Network Modeling (NETMOD) working
group for its discussion of YANG path formats.



