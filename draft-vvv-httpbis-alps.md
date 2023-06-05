---
title: "Using TLS Application-Layer Protocol Settings (ALPS) in HTTP"
abbrev: "HTTP ALPS"
docname: draft-vvv-httpbis-alps-latest
category: std

ipr: trust200902
area: General
workgroup: HTTP Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: V. Vasiliev
    name: Victor Vasiliev
    organization: Google
    email: vasilvv@google.com
 -
    ins: V. Tan
    name: Victor Tan
    organization: Google LLC
    email: victortan@google.com

normative:
  RFC2119:
  HTTP3:
    title: "Hypertext Transfer Protocol Version 3 (HTTP/3)"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-http-latest
    author:
      -
          ins: M. Bishop
          name: Mike Bishop
          org: Akamai Technologies
          role: editor
  ALPS:
    title: "TLS Application-Layer Protocol Settings Extension"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-vvv-tls-alps-latest
    author:
      -
          ins: V. Vasiliev
          name: Victor Vasiliev
          org: Google

informative:


--- abstract

This document describes the use of TLS Application-Level Protocol Settings
(ALPS) in HTTP/2 and HTTP/3.  Additionally, it defines a set of additional HTTP
SETTINGS parameters that would normally be impractical without ALPS.

--- middle

# Introduction

HTTP/2 defines a mechanism for exchanging the protocol settings using a
SETTINGS frame ({{!RFC7540}}, Section 6.5).  HTTP/3 uses a similar mechanism
([HTTP3], Section 7.2.4).  One of the properties of the mechanism as defined by
both of those protocols is that the parties start out without having access to
the entirety of the peer's settings.  This means that they have to initially
operate using the default settings, and after receiving the SETTINGS frame,
they have to find a way to transition from the default to the exchanged
settings.

HTTP is commonly used in conjunction with TLS.  TLS performs its own handshake
that precedes any data being exchanged by the HTTP layer itself.  The TLS
Application-Level Protocol Settings extension [ALPS] allows settings
negotiation to be performed within the TLS handshake, thus making the result
immediately available to the HTTP layer as soon as the handshake completes.
This removes the need for synchronizing settings, and makes them available
earlier than they would be otherwise.

This document defines how ALPS is used with HTTP/2 and HTTP/3, and introduces
certain new settings that would not be practical without ALPS.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Use of ALPS in HTTP/2

When configured for HTTP/2, the ALPS payload for both peers SHALL be a sequence
of HTTP/2 frames.  The sender MUST NOT send a frame in ALPS unless the
corresponding value in the "Allowed in ALPS" column in the "HTTP/2 Frame Type"
registry ({{iana}}) is "Yes". This document only allows the SETTINGS frame
({{!RFC7540}}, Section 6.5.1). Frames defined in later documents may be allowed
in ALPS, so the receiver MUST ignore unrecognized frames in the ALPS payload.

If ALPS is successfully negotiated during a TLS handshake for an HTTP/2
connection, the protocol is updated as follows:

Sending a SETTINGS frame without the ACK flag in ALPS supersedes the
requirement to send a SETTINGS frame at the beginning of the connection. All
settings exchanged via ALPS SHALL be automatically treated as acknowledged.
Implementations MAY continue to send additional SETTINGS frames over the
connection. See {{early-data}} for guidance.

Since settings exchanged through ALPS are always available at the beginning of
the connection, HTTP/2 extensions MAY opt to require those to be sent through
ALPS.

Implementations MUST NOT send SETTINGS frames with the ACK flag in ALPS. If
the peer sends such a frame in ALPS, the receiver MUST treat it as a connection
error of type PROTOCOL_ERROR.

[[OPEN ISSUE: Is SETTINGS in ALPS mandatory or optional? Should it be the
first frame?]]

# Use of ALPS in HTTP/3

When configured for HTTP/3, the ALPS payload for both peers SHALL be a sequence
of HTTP/3 frames.  The sender MUST NOT send a frame in ALPS unless the
corresponding value in the "Allowed in ALPS" column in the "HTTP/3 Frame Type"
registry ({{iana}}) is "Yes". This document only allows the SETTINGS frame
([HTTP3], Section 7.2.4).  Frames defined in later documents may be allowed in
ALPS, so the receiver MUST ignore unrecognized frames in the ALPS payload.

If ALPS is successfully negotiated during a TLS handshake for an HTTP/2
connection, the protocol is updated as follows:

ALPS updates the initial value of each setting as described in Section 7.2.4.2
of [HTTP3]. In both the client and server, the initial value of each setting
is the value in the ALPS SETTINGS frame, or the default value if not specified.
This allows messages to incorporate non-default setting values without waiting
for the peer's SETTINGS frame.

Unlike with HTTP/2, a SETTINGS frame in ALPS does not replace the SETTINGS
frame on the control stream. HTTP/3 implementations MUST continue to send a
SETTINGS frame as the first frame on the control stream. This frame MAY be
empty, or it MAY contain additional settings advertised outside of ALPS. See
{{early-data}} for guidance. This frame MUST NOT reduce any limits or alter any
values that might be violated by a peer using the initial values from ALPS. If
the peer sends an incompatible SETTINGS frame, the receiver MUST treat it as a
connection error of type H3_SETTINGS_ERROR.

ALPS additionally replaces the 0-RTT mechanisms in Section 7.4.2.4 of [HTTP3].
Clients and servers SHOULD NOT separately remember settings with the TLS
session. Instead, clients attempting 0-RTT MUST comply with the initial values
of each setting, as determined above. Note that these values will incorporate
the server ALPS value saved with the TLS session. If a client receives a TLS
NewSessionTicket message before the control stream SETTINGS frame, it SHOULD
NOT wait for the SETTINGS frame before processing the NewSessionTicket.

A server MAY accept 0-RTT independently of the setting values of the previous
connection. Instead, the mechanism described in Section 4.2 of {{ALPS}} ensures
compatibility with the client state. This removes the need for HTTP/3 to be
directed integrated with 0-RTT acceptance.

Since settings exchanged through ALPS are always available at the beginning of
the connection, HTTP/3 extensions MAY opt to require those to be sent through
ALPS.

[[OPEN ISSUE: Is SETTINGS in ALPS mandatory or optional? Should it be the
first frame?]]

# Interaction with Early Data {#early-data}

With ALPS, there are two ways to send HTTP/2 and HTTP/3 setting values: inside
ALPS and on the control stream (stream identifier zero in HTTP/2). This allows
implementations to trade off reliable ordering and volatility.

Values placed in ALPS are available to the peer before sending messages. This
allows improved performance and removes the need to transition from the default
settings. However, achieving this with 0-RTT requires these values be carried
over to subsequent connections. If any ALPS settings have since changed, 0-RTT
will be rejected to allow the new settings to take effect. This means values
that change on a per-connection basis will reduce 0-RTT acceptance.

In contrast, values sent over the control stream can change over time without
affected 0-RTT. However, they are not available at the start of the connection
and may not be applied to all messages in a connection, notably those sent over
0-RTT.

It is expected that most setting values reflect implementation configuration or
capabilities, which do not change frequently. These values SHOULD be sent
inside ALPS. However, setting values that vary per connection would not perform
well and SHOULD be sent on the control stream. In particular, when including
reserved settings identifiers (see Section 7.2.4.1 of [HTTP3]), the
identifiers SHOULD either be consistent across 0-RTT or sent on the control
stream.

[[OPEN ISSUE: We could relax the story for reserved settings if we wanted to.
See https://github.com/vasilvv/tls-alps/issues/11]]

# Security Considerations

In ALPS, both client and server settings are sent encrypted.  Settings
communicated through ALPS are presented to all clients before they are
authenticated; thus, if a server relies on TLS client authentication and
considers its settings private, it MUST NOT use the mechanism defined in this
document.

# IANA Considerations {#iana}

IANA will add an "Allowed in ALPS" column to the "HTTP/2 Frame Type" registry
{{RFC7540}}, with a value set to "Yes" for SETTINGS (0x4), and to "No" for all
other previously defined settings.

IANA will add an "Allowed in ALPS" column to the "HTTP/3 Frame Type" registry
{{HTTP3}}, with a value set to "Yes" for SETTINGS (0x4), and to "No" for all
other previously defined settings.

--- back

# Acknowledgments
{:numbered="false"}

This document has benefited from contributions and suggestions from
Bence Béky,
David Benjamin,
Nick Harper,
David Schinazi,
and many others.
