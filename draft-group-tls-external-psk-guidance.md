---
title: Guidance for External PSK Usage in TLS
abbrev: Guidance for External PSK Usage in TLS
docname: draft-group-tls-external-psk-guidance-latest
category: info

ipr: trust200902
area: General
workgroup: tls
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: F. Bar
    name: Foo Bar
    organization: Baz, Inc.
    email: foobar@baz.com


normative:
  RFC1035:
  RFC2119:
  RFC6234:

informative:
  Selfie:
     title: "Selfie: reflections on TLS 1.3 with PSK"
     author:
         -
             ins: N. Drucker
             name: Nir Drucker
         -
             ins: S. Gueron
             name: Shay Gueron
     date: 2019
     target: https://eprint.iacr.org/2019/347.pdf

--- abstract

TODO

--- middle

# Introduction

TODO

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Use Cases and Provisioning Processes

Pre-shared Key (PSK) ciphersuites were first specified for TLS in 2005. Now, PSK is an integral part
of TLS 1.3 specification [RFC8846]. TLS 1.3 also uses PSKs for session resumption. It distinguishes
these resumption PSKs from external PSKs which have been provisioned out-of-band (OOB). Below, we list
some example use-cases where pair-wise external PSKs with TLS have been used for authentication.
Note that the following list is non-exhaustive:

- Device-to-device communication with out-of-band synchronized keys. PSKs provisioned out-of-band for
communicating with known identities, wherein the identity to use is discovered via a different online protocol.
- Intra-data-center communication. Machine-to-machine communication within a single data center or PoP
may use externally provisioned PSKs, primarily for the purposes of supporting TLS connections
with fast open (0-RTT data).
- Internet of Things (IoT) and devices with limited computational capabilities. The Constrained Application
Protocol (CoAP) {{?RFC7252}} is a web transfer protocol intended for resource-constrained IoT devices. It suggests
use of the TLS_PSK_WITH_AES_128_CCM_8 ciphersuite with DTLS for securing communication between CoAP clients
and servers. {{?RFC7925}} defines TLS and DTLS profiles for resource-constrained devices and requires
compliant devices to support the TLS_PSK_WITH_AES_128_CCM_8 ciphersuite.
- Wireless Networks. The Control And Provisioning of Wireless Access Points (CAPWAP) protocol
specification {{?RFC5415}} mandates support of TLS_PSK_WITH_AES_128_CBC_SHA and TLS_DHE_PSK_WITH_AES_128_CBC_SHA
ciphersuites with DTLS for authentication between the centralized Access Controller (AC) and distributed
Wireless Termination Points (WTPs). Use of PSK ciphersuites are Ooptional when securing RADIUS {{?RFC2865}} with
TLS as specified in {{?RFC6614}}. The Generic Authentication Architecture (GAA) defined by 3GGP in TR 133 919 V12.0.0
(https://www.etsi.org/deliver/etsi_tr/133900_133999/133919/12.00.00_60/tr_133919v120000p.pdf) mentions that TLS-PSK
can be used between a server and user equipment for authentication.
- Smart Cards. The electronic German ID (eID) card supports authentication of a card holder to online
services with TLS-PSK. The German Federal Office for Information Security (BSI) describes how TLS-PSK
authentication is utilized in the eCard-API-Framework specification
(TR-03112-7:https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Publikationen/TechnischeRichtlinien/TR03112/TR-03112-api_teil7.pdf?__blob=publicationFile&v=1).
EMV support for TLS-PSK {{?I-D.urien-tls-psk-emv}} discusses how the identities and PSK attributes for
TLS-PSK can be extracted from EMV cards.

There are also use cases where PSKs are shared between more than two entities. Some examples are
below:

- Group chats.s In this use-case, the membership of a group is confirmed by the possession of a PSK
distributed out-of-band to the group participants. Members can then establish peer-to-peer connections
with each other using the external PSK. It is important to note that any node of the group can behave
as a TLS client or server.
- Internet of Things (IoT). In this use-case, resource-constrained IoT devices act as TLS clients and
share the same PSK. The devices use this PSK for quickly establishing connections with a central server.
Such a scheme ensures that the client IoT devices are legitimate members of the system. To perform rare
system specific operations that require a higher security level, the central server can request
resource-intensive client authentication with the usage of a certificate after successfully establishing
the connection with a PSK.

## Provisioning Examples

- Many industrial protocols assume that PSKs are distributed and assigned by manually via
one of the following approaches: typing the PSK into the devices, or via a web server
masks (using a TOFU approach with a device completely unprotected before the first login did take place).
Many devices have very limited GUI functionalities. For example, they only a numeric keypad or even only five
buttons. So when the TOFU approach is not suitable in a setting, entering the key would require typing it on a
constrained HMI interface. Moreover, PSK production lacks guidance, similar to user passwords.
- Some devices are provisioned PSKs via an out-of-band, cloud-based syncing protocol.
- Secrets may be baked into or hardware or software device components, such as secure enclaves.s

## Provisioning Constraints

PSK provisioning systems are often constrained in application-specific ways. For example,
although one goal of provisioning is to ensure that each pair of nodes has a unique key
pair, some systems do not want to distribute pair-wise shared keys to achieve this.
As another example, some systems require the provisioning process to embed application-specific
information in either PSKs or their identities. Identities may sometimes need to be routable, as
is currently under discussion for EAP-TLS.

# Security and Privacy Properties

Against a passive attacker AdvP or active attacker AdvA, desired privacy properties might include:

- Peer Authentication. The client’s view of the peer identity should reflect the server’s identity.
If the client is authenticated, the server’s view of the peer identity should match the client’s identity.
Moreover, the client’s view of the peer identity should not match its own identity {{Selfie}}.
- Channel Binding. External PSKs should include channel bindings from any protocol run a priori to
provision the PSKs.
- Identity Confidentiality. AdvP should learn no information about the external PSK or its identifier.
Similarly, AdvA should be unable to replay ClientHello messages and learn information about the PSK.
- Identity Unlinkability. AdvP should be unable to link two or more connections together that use the
same external PSK.

Unlinkability for the PSK receiver, i.e., ensuring that the recipient of a ClientHello with a given PSK
k cannot link k to a prior session, is out of scope. (PSKs are explicitly designed to support mutual
authentication.)

## Security Assumptions

TODO(jonathan): writeme

- Shared session keys:
- Session key secrecy:
- Peer authentication:
- Session key uniqueness:
- Downgrade protection:
- Forward secrecy:
- KCI protection:
- Endpoint identity protection:

# Known Attacks

TODO

# Recommendations for External PSK Usage

TODO

# IANA Considerations {#IANA}

TODO

--- back

# Acknowledgements

TODO