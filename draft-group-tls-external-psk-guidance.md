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
  RFC2119:
  RFC8446:

informative:
  RFC6614:
  RFC7925:
  RFC2865:
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
  Akhmetzyanova:
     title: "Continuing to reflect on TLS 1.3 with external PSK"
     author:
         -
             ins: L. Akhmetzyanova
             name: Liliya Akhmetzyanova
         -
             ins: E. Alekseev
             name: Evgeny Alekseev
         -
             ins: E. Smyshlyaeva
             name: Ekaterina Smyshlyaeva
         -
             ins: A. Sokolov
             name: Alexandr Sokolov
     date: 2019
     target: https://eprint.iacr.org/2019/421.pdf
  GAA:
    title: "TR33.919 version 12.0.0 Release 12"
    target: https://www.etsi.org/deliver/etsi_tr/133900_133999/133919/12.00.00_60/tr_133919v120000p.pdf
  SmartCard:
    title: "Technical Guideline TR-03112-7 eCard-API-Framework – Protocols"
    target: https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Publikationen/TechnischeRichtlinien/TR03112/TR-03112-api_teil7.pdf?__blob=publicationFile&v=1
    date: 2015
  Krawczyk:
     title: "SIGMA: The ‘SIGn-and-MAc’ Approach to Authenticated Diffie-Hellman and Its Use in the IKE Protocols"
     author:
         -
             ins: H. Krawczyk
             name: Hugo Krawczyk
     date: 2003
     seriesinfo: Annual International Cryptology Conference. Springer, Berlin, Heidelberg
     target: https://link.springer.com/content/pdf/10.1007/978-3-540-45146-4_24.pdf     
  Sethi:
     title: "Misbinding Attacks on Secure Device Pairing and Bootstrapping"
     author:
     author:
         -
             ins: M. Sethi
             name: Mohit Sethi
         -
             ins: A. Peltonen
             name: Aleksi Peltonen
         -
             ins: T. Aura
             name: Tuomas Aura          
     date: 2019
     seriesinfo: Proceedings of the 2019 ACM Asia Conference on Computer and Communications Security
     target: https://arxiv.org/pdf/1902.07550    
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

# Use Cases and Provisioning Processes {#use-cases}

Pre-shared Key (PSK) ciphersuites were first specified for TLS in 2005. Now, PSK is an integral part of the TLS version 1.3 specification {{RFC8446}}. TLS 1.3 also uses PSKs for session resumption. It distinguishes these resumption PSKs from external PSKs which have been provisioned out-of-band (OOB). Below, we list some example use-cases where pair-wise external PSKs with TLS have been used for authentication. Note that the following list is non-exhaustive:

- Device-to-device communication with out-of-band synchronized keys. PSKs provisioned out-of-band for communicating with known identities, wherein the identity to use is discovered via a different online protocol.

- Intra-data-center communication. Machine-to-machine communication within a single data center or PoP may use externally provisioned PSKs, primarily for the purposes of supporting TLS connections with fast open (0-RTT data).

- Internet of Things (IoT) and devices with limited computational capabilities. {{RFC7925}} defines TLS and DTLS profiles for resource-constrained devices and suggests the use of PSK ciphersuites for compliant devices.

- Use of PSK ciphersuites are optional when securing RADIUS {{RFC2865}} with TLS as specified in {{RFC6614}}. 

- The Generic Authentication Architecture (GAA) defined by 3GGP mentions that TLS-PSK can be used between a server and user equipment for authentication {{GAA}}.

- Smart Cards. The electronic German ID (eID) card supports authentication of a card holder to online services with TLS-PSK {{SmartCard}}.


There are also use cases where PSKs are shared between more than two entities. Some examples below (as noted by Akhmetzyanova et al.{{Akhmetzyanova}}):

- Group chats. In this use-case, the membership of a group is confirmed by the possession of a PSK distributed out-of-band to the group participants. Members can then establish peer-to-peer connections with each other using the external PSK. It is important to note that any node of the group can behave as a TLS client or server.

- Internet of Things (IoT). In this use-case, resource-constrained IoT devices act as TLS clients and share the same PSK. The devices use this PSK for quickly establishing connections with a central server. Such a scheme ensures that the client IoT devices are legitimate members of the system. To perform rare system specific operations that require a higher security level, the central server can request resource-intensive client authentication with the usage of a certificate after successfully establishing the connection with a PSK.

## Provisioning Examples

- Many industrial protocols assume that PSKs are distributed and assigned manually via one of the following approaches: typing the PSK into the devices, or via web server masks (using a Trust On First Use (TOFU) approach with a device completely unprotected before the first login did take place). Many devices have very limited UI. For example, they may only have a numeric keypad or even less number of buttons. When the TOFU approach is not suitable, entering the key would require typing it on a constrained UI. Moreover, PSK production lacks guidance unlike user passwords. 

- Some devices are provisioned PSKs via an out-of-band, cloud-based syncing protocol.

- Secrets may be baked into or hardware or software device components, such as secure enclaves.

## Provisioning Constraints

PSK provisioning systems are often constrained in application-specific ways. For example, although one goal of provisioning is to ensure that each pair of nodes has a unique key pair, some systems do not want to distribute pair-wise shared keys to achieve this. As another example, some systems require the provisioning process to embed application-specific information in either PSKs or their identities. Identities may sometimes need to be routable, as is currently under discussion for EAP-TLS-PSK.

## Stack Interfaces

Most major TLS implementations support external PSKs. And all have a common interface that
applications may use when supplying them for individual connections. Details about existing
stacks at the time of writing is below.

- OpenSSL and BoringSSL: Applications specify support for external PSKs via distinct ciphersuites.
They also then configure callbacks that are invoked for PSK selection during the handshake.
These callbacks must provide the PSK identity (as a character string) and key (as a byte string).
They are typically invoked with a PSK hint, i.e., the hint provided by the server as per {{?RFC4279}}.
The PSK length is validated to be between \[1, 256\] bytes upon selection.
- mbedTLS: Client applications configure PSKs before creating a connection by providing the PSK
identity and value inline. Servers must implement callbacks similar to that of OpenSSL. PSK lengths
are validate to be between \[1, 16\] bytes.
- gnuTLS: Applications configure PSK values, either as raw byte strings or hexadecimal strings. The PSK size is not validated.
- wolfSSL: Applications configure PSKs with callbacks similar to OpenSSL.

### PSK Identity encoding and comparison 

Section 5.1 of {{?RFC4279}} mandates that the PSK identity should be first converted to a character string and then encoded to octets using UTF-8. This was done to avoid interoperability problems (especially when the identity is configured by human users). On the other hand, {{RFC7925}} advises implementations against assuming any structured format for PSK identities and recommends byte-by-byte comparison for any operations. TLS version 1.3 {{RFc8446}} follows the same practice of specifying the psk identity as a sequence of opaque bytes (shown as opaque identity<1..2^16-1>). 

- Implementations such as OpenSSL and mbedTLS nonetheless treat psk_identities as character strings and use string operators such as strcmp on the psk identity. 
- Implementations also assign default identities to PSKs (for example, the string 'Client_identity') if none are configured.
- gnuTLS treats psk identities as usernames.
- OpenSSL TLS 1.3 servers accept connections from clients that have a valid PSK even if the identity provided by the client is incorrect. 


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

The TLS external PSK authentication makes the implicit assumption that each PSK is known only to one client and one server, which do not switch roles with the same PSK.

As discussed in {{use-cases}}, there are use cases where multiple clients or multiple servers share a PSK. In such cases, TLS only authenticates the entire group. Not only can a compromised group member impersonate another group member, but a malicious non-member can reroute handshakes between honest group members to connect them in unintended ways. This rerouting is a type of identity misbinding attack {{Krawczyk}}{{Sethi}}.

Selfie attack {{Selfie}} is a special case of the rerouting attack against a group member that can act both as TLS server and client. In the selfie attack, a malicious non-member reroutes a connection from the client to the server on the same endpoint.

# Recommendations for External PSK Usage

TODO

# IANA Considerations {#IANA}

TODO

--- back

# Acknowledgements

TODO