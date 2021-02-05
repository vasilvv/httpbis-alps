---
title: "Using TLS Application-Layer Protocol Settings (ALPS) in HTTP"
abbrev: "HTTP ALPS"
docname: draft-vvv-httpbis-alps
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

# Use of ALPS in HTTP

If ALPS is successfully negotiated during the TLS handshake for an HTTP/2
connection, the ALPS payload for both peers SHALL be a sequence of HTTP/2
frames.  Senders MUST NOT send frames in ALPS unless the "Allowed in ALPS"
column in the "HTTP/2 Frame Type" registry ({{iana}}). This document only
allows the SETTINGS frame ({{!RFC7540}}, Section 6.5.1).  Frames defined in
later documents may be allowed in ALPS, so receivers MUST ignore unrecognized
frames in the ALPS payload. Sending a SETTINGS frame in ALPS supersedes the
requirement to send a SETTINGS frame at the beginning of the connection.  All
settings exchanged via ALPS SHALL be automatically treated as acknowledged.

If ALPS is successfully negotiated during TLS handshake for an HTTP/3
connection, the ALPS payload for both peers SHALL be a sequence of HTTP/3
frames.  Senders MUST NOT send frames in ALPS unless the "Allowed in ALPS"
column in the "HTTP/3 Frame Type" registry ({{iana}}). This document only
allows the SETTINGS frame ([HTTP3], Section 7.2.4).  Frames defined in later
documents may be allowed in ALPS, so receivers MUST ignore unrecognized frames
in the ALPS payload. Sending a SETTINGS frame in ALPS supersedes the
requirement to send a SETTINGS frame at the beginning of the control stream.

[[OPEN ISSUE: Unlike HTTP/2, HTTP/3 sends exactly one SETTINGS frame, so we
need to be a bit more precise here.]]

Since settings exchanged through ALPS are always available at the beginning of
the connection, some HTTP extensions may opt to require those to be sent
through ALPS.  Such extensions are exempt from the initialization requirements
of the Section 7.2.4.2 of [HTTP3].

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
David Benjamin,
Nick Harper,
David Schinazi,
and many others.
