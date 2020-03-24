---
title: Guidance for External PSK Usage in TLS
abbrev: Guidance for External PSK Usage in TLS
docname: draft-dt-tls-external-psk-guidance-latest
category: info

ipr: trust200902
area: General
workgroup: tls
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: R. Housley
    name: Russ Housley
    organization: Vigil Security
    email: housley@vigilsec.com
  -
    ins: J. Hoyland
    name: Jonathan Hoyland
    organization: Cloudflare Ltd.
    email: jonathan.hoyland@gmail.com
  -
    ins: M. Sethi
    name: Mohit Sethi
    organization: Ericsson
    email: mohit@piuha.net
  -
    ins: C.A. Wood
    name: Christopher A. Wood
    email: caw@heapingbits.net

normative:
  RFC2119:


informative:
  RFC6614:
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
  LwM2M:
    title: "Lightweight Machine to Machine Technical Specification"
    target: http://www.openmobilealliance.org/release/LightweightM2M/V1_0-20170208-A/OMA-TS-LightweightM2M-V1_0-20170208-A.pdf
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

This document provides usage guidance for external Pre-Shared Keys (PSKs) in TLS.
It lists TLS security properties provided by PSKs under certain assumptions and
demonstrates how violations of these assumptions lead to attacks. This document
also discusses PSK use cases, provisioning processes, and TLS stack implementation
support in the context of these assumptions. It provides advice for applications
in various use cases to help meet these assumptions.

--- middle

# Introduction

This document provides usage guidance for external Pre-Shared Keys (PSKs) in TLS.
It lists TLS security properties provided by PSKs under certain assumptions and
demonstrates how violations of these assumptions lead to attacks. This document
also discusses PSK use cases, provisioning processes, and TLS stack implementation
support in the context of these assumptions. It provides advice for applications
in various use cases to help meet these assumptions.

The guidance provided in this document is applicable across TLS {{?RFC8446}},
DTLS {{!I-D.ietf-tls-dtls13}}, and Constrained TLS {{!I-D.rescorla-tls-ctls}}.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# PSK Security Properties {#sec-properties}

The external PSK authentication mechanism in TLS implicitly assumes
one fundamental property: each PSK is known to exactly one client and
one server, and that these never switch roles.  If this assumption is
violated, then the security properties of TLS are severely weakened.

As discussed in {{use-cases}}, there are use cases where it is
desirable for multiple clients or multiple servers share a PSK. If
this is done naively by having all members share a common key, then
TLS only authenticates the entire group, and the security of the
overall system is inherently rather brittle. There are a number of
obvious weaknesses here:

1. Any group member can impersonate any other group member.
2. If a group member is compromised, then the attacker can impersonate any group member (this follows from property (1)).
3. If PSK without DH is used, then compromise of any group member allows the attacker to passively read all traffic.

In addition to these, a malicious non-member can reroute handshakes
between honest group members to connect them in unintended ways, as
detailed below.

Let the group of peers who know the key be `A`, `B`, and `C`.
The attack proceeds as follows:

1. `A` sends a `ClientHello` to `B`.
2. The attacker intercepts the message and redirects it to `C`.
3. `C` responds with a `ServerHello` to `A`.
4. `A` sends a `Finished` message to `B`.
`A` has completed the handshake, ostensibly with `B`.
5. The attacker redirects the `Finished` message to `C`.
`C` has completed the handshake, ostensibly with `A`.

This attack violates the peer authentication property, and if `C` supports a weaker set of cipher suites than `B`, this
attack also violates the downgrade protection property.  This rerouting is a type of identity misbinding attack
{{Krawczyk}}{{Sethi}}.  Selfie attack {{Selfie}} is a special case of the rerouting attack against a group member that
can act both as TLS server and client. In the Selfie attack, a malicious non-member reroutes a connection from the
client to the server on the same endpoint.




# Privacy Properties

PSK privacy properties are orthogonal to security properties described in {{sec-properties}}.
Traditionally, TLS does little to keep PSK identity information private. For example,
an adversary learns information about the external PSK or its identifier by virtue of it
appearing in cleartext in a ClientHello. As a result, a passive adversary can link
two or more connections together that use the same external PSK on the wire. Applications should
take precautions when using external PSKs if these risks.

In addition to linkability in the network, external PSKs are intrinsically linkable by PSK receivers.
Specifically, servers can link successive connections that use the same external PSK together. Preventing
this type of linkability is out of scope, as PSKs are explicitly designed to support mutual authentication.

# External PSK Use Cases and Provisioning Processes {#use-cases}

Pre-shared Key (PSK) ciphersuites were first specified for TLS in 2005. Now, PSK is an integral
part of the TLS version 1.3 specification {{!RFC8446}}. TLS 1.3 also uses PSKs for session resumption.
It distinguishes these resumption PSKs from external PSKs which have been provisioned out-of-band (OOB).
Below, we list some example use-cases where pair-wise external PSKs with TLS have been used for
authentication.

- Device-to-device communication with out-of-band synchronized keys. PSKs provisioned out-of-band
for communicating with known identities, wherein the identity to use is discovered via a different
online protocol.

- Intra-data-center communication. Machine-to-machine communication within a single data center
or PoP may use externally provisioned PSKs, primarily for the purposes of supporting TLS
connections with fast open (0-RTT data).

- Certificateless server-to-server communication. Machine-to-machine communication may use externally provisioned PSKs, primarily for the purposes of establishing TLS connections without requiring the overhead of provisioning and managing PKI certificates.

- Internet of Things (IoT) and devices with limited computational capabilities. {{?RFC7925}}
defines TLS and DTLS profiles for resource-constrained devices and suggests the use of PSK
ciphersuites for compliant devices. The Open Mobile Alliance Lightweight Machine to Machine Technical Specification {{LwM2M}} states that LwM2M servers MUST support the PSK mode of DTLS.

- Use of PSK ciphersuites are optional when securing RADIUS {{RFC2865}} with TLS as specified
in {{RFC6614}}.

- The Generic Authentication Architecture (GAA) defined by 3GGP mentions that TLS-PSK can be used
between a server and user equipment for authentication {{GAA}}.

- Smart Cards. The electronic German ID (eID) card supports authentication of a card holder to
online services with TLS-PSK {{SmartCard}}.

There are also use cases where PSKs are shared between more than two entities. Some examples below
(as noted by Akhmetzyanova et al.{{Akhmetzyanova}}):

- Group chats. In this use-case, the membership of a group is confirmed by the possession of a PSK
distributed out-of-band to the group participants. Members can then establish peer-to-peer connections
with each other using the external PSK. It is important to note that any node of the group can behave
as a TLS client or server.

- Internet of Things (IoT). In this use-case, resource-constrained IoT devices act as TLS clients and share the
same PSK. The devices use this PSK for quickly establishing connections with a central server. Such a scheme
ensures that the client IoT devices are legitimate members of the system. To perform rare system specific
operations that require a higher security level, the central server can request resource-intensive client
authentication with the usage of a certificate after successfully establishing the connection with a PSK.

## Provisioning Examples

- Many industrial protocols assume that PSKs are distributed and assigned manually via one of the following
approaches: typing the PSK into the devices, or via web server masks (using a Trust On First Use (TOFU)
approach with a device completely unprotected before the first login did take place). Many devices have very
limited UI. For example, they may only have a numeric keypad or even less number of buttons. When the TOFU
approach is not suitable, entering the key would require typing it on a constrained UI. Moreover, PSK production
lacks guidance unlike user passwords.

- Some devices are provisioned PSKs via an out-of-band, cloud-based syncing protocol.

- Some secrets may be baked into or hardware or software device components. Moreover, when this is done
at manufacturing time, secrets may be printed on labels or included in a Bill of Materials for ease of
scanning or import.

## Provisioning Constraints

PSK provisioning systems are often constrained in application-specific ways. For example, although one goal of
provisioning is to ensure that each pair of nodes has a unique key pair, some systems do not want to distribute
pair-wise shared keys to achieve this. As another example, some systems require the provisioning process to embed
application-specific information in either PSKs or their identities. Identities may sometimes need to be routable,
as is currently under discussion for EAP-TLS-PSK.


# Recommendations for External PSK Usage


Applications MUST use external PSKs that adhere to the following requirements:

1. Each PSK MUST be derived from at least 128 of entropy and MUST be at least 128-bits long unless the TLS handshake is being used with a separate key establishment mechanism such as a Diffie-Hellman exchange. This recommendation protects against passive attacks using exhaustive search of the PSK.
2. Each PSK MUST NOT be shared between with more than two logical nodes. As a result, an agent that acts as both a client and a server MUST use distinct PSKs when acting as the client from when it is acting as the server. This prevents redirection attacks.
3. Nodes SHOULD use external PSK importers {{!I-D.ietf-tls-external-psk-importer}} when configuring PSKs for a pair of TLS client and server. {{!RFC8446}} mandates that each external PSK can only be used with a single hash function. {{!I-D.ietf-tls-external-psk-importer}} overcomes this limitation by producing a set of candidate PSKs, each of which are 
bound to a specific KDF. 
4. {{!I-D.ietf-tls-external-psk-importer}} also helps in mitigating selfie style reflection attacks if a pair of client and server can switch roles. This is achieved by correctly using the node identifiers in the ImportedIdentity.context construct as explained in Appendix B of {{!I-D.ietf-tls-external-psk-importer}}. When a PSK is shared between a client and server that can switch roles, it is not necessary to know the identifier(s) of the other endpoint. Instead, it is sufficient to check that the identifier of the other endpoint in context is not equal to any of one’s own identifiers. To simplify implementation of this check, it is RECOMMENDED that each endpoint selects one globally unique identifier and uses it in all PSK handshakes. The unique identifier can, for example, be one of its MAC addresses or a 32-byte random number. Including node identifiers in the ImportedIdentity.context construct also ensures that condition (2) is met.
5. It is NOT RECOMMENDED to share the same PSK between more than one client and server. However, as discussed in {{use-cases}}, there are application scenarios that may rely on sharing the same PSK among multiple nodes. {{!I-D.ietf-tls-external-psk-importer}} MUST be used in these application scenarios to derive unique keys for each logical node pair. Using node identifiers in the ImportedIdentity.context construct can again help nodes to derive unique keys from the global PSK shared among all nodes. Unlike the case of a server and client pair that can switch roles, when the same PSK is shared among many nodes, the comparison with one’s own identifiers will only prevent selfie attacks but not malicious rerouting of the connection to another group member. To prevent malicious rerouting in groups, each endpoint needs to know the identifier of the other endpoint with which they want to connect and compare it with the other endpoint’s identifier used in ImportedIdentity.context. Of course, this only prevents attacks by non-members; the endpoints that share the group key can always impersonate each other. 
6. Where possible the master PSK (that which is fed into the importer) SHOULD be deleted after the imported keys have been generated. This protects an attacker from bootstrapping a compromise of one node into the ability to attack connections between any node; otherwise the attacker can recover the master key and then re-run the importer itself.



## Stack Interfaces

Most major TLS implementations support external PSKs. And all have a common interface that
applications may use when supplying them for individual connections. Details about existing
stacks at the time of writing are below.

- OpenSSL and BoringSSL: Applications specify support for external PSKs via distinct ciphersuites.
They also then configure callbacks that are invoked for PSK selection during the handshake.
These callbacks must provide a PSK identity (as a character string) and key (as a byte string).
(If no identity is provided, a default one is assumed.)
They are typically invoked with a PSK hint, i.e., the hint provided by the server as per {{?RFC4279}}.
The PSK length is validated to be between \[1, 256\] bytes upon selection.
- mbedTLS: Client applications configure PSKs before creating a connection by providing the PSK
identity and value inline. Servers must implement callbacks similar to that of OpenSSL. PSK lengths
are validate to be between \[1, 16\] bytes.
- gnuTLS: Applications configure PSK values, either as raw byte strings or hexadecimal strings. The PSK size is not validated.
- wolfSSL: Applications configure PSKs with callbacks similar to OpenSSL.

### PSK Identity encoding and comparison

Section 5.1 of {{?RFC4279}} mandates that the PSK identity should be first converted to a character string and then encoded to octets using UTF-8. This was done to avoid interoperability problems (especially when the identity is configured by human users). On the other hand, {{?RFC7925}} advises  implementations against assuming any structured format for PSK identities and recommends byte-by-byte comparison for any operations. TLS version 1.3 {{!RFC8446}} follows the same practice of specifying the psk identity as a sequence of opaque bytes (shown as opaque identity<1..2^16-1>).

{{!RFC8446}} does not place strict requirements on the format of PSK identities. We do however note that the format of PSK identities can vary depending on the deployment:

- The PSK identity MAY be a user configured string when used in protocols like Extensible Authentication Protocol (EAP) {{?RFC3748}}. gnuTLS for example treats PSK identities as usernames.
- PSK identities MAY have a domain name suffix for roaming and federation. 
- The maximum length of a PSK identity is 65535 bytes. However, there are no restrictions on the minimum length. Nonetheless, deployments should take care that the length is sufficient to avoid obvious collisions. {{identity-collision}} discusses some implication of collisions between external and resumption PSKs.


#### PSK Identity collisions {#identity-collision}


# IANA Considerations {#IANA}

This document makes no IANA requests.

--- back

# Acknowledgements

This document is the output of the TLS External PSK Design Team, comprised of the following members:
Benjamin Beurdouche,
Björn Haase,
Chris Wood,
Colm MacCárthaigh,
Eric Rescorla,
Jonathan Hoyland,
Martin Thomson,
Mohamad Badra,
Mohit Sethi,
Oleg Pekar,
Owen Friel, and
Russ Housley.
