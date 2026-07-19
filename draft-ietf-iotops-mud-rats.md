---
title: MUD-Based RATS Resources Discovery
abbrev: Muddy Rats
docname: draft-ietf-iotops-mud-rats-latest
stand_alone: true
ipr: trust200902
area: Security
wg: "Remote ATtestation ProcedureS" 
kw: Internet-Draft
cat: std
submissionType: IETF
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes

author:
- ins: H. Birkholz
  name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@ietf.contact
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
- ins: M. Richardson
  name: Michael Richardson
  org: Sandelman Software Works
  email: mcr+ietf@sandelman.ca
  street: ""
  code: ""
  city: ""
  region: ""
  country: Canada
- ins: C. Liu
  name: Chunchi Liu
  org: Huawei Technologies
  email: liuchunchi@huawei.com

normative:
  RFC2119:
  RFC6991:
  RFC7950:
  RFC8071:
  RFC8340:
  RFC8342:
  RFC8520: mud
  RFC8526:
  RFC9334: rats-arch
  I-D.ietf-rats-eat: eat
  I-D.ietf-rats-msg-wrap: cmw
  I-D.ietf-rats-corim: corim
  IANA.cwt:
  IANA.jwt:

informative:
  RFC4949:
  I-D.ietf-spice-sd-cwt: sd-cwt

entity:
  SELF: "RFCthis"

--- abstract

Manufacturer Usage Description (MUD) files and the MUD URIs that point to them are defined in RFC 8520.
This document introduces a new type of MUD file to be delivered in conjunction with a MUD file signature and/or to be referenced via a MUD URI embedded in other documents or messages, such as an IEEE 802.1AR Secure Device Identifier (DevID) or a CBOR Web Token (CWT).
These signed documents can be presented to other entities, e.g., a network management system or network path orchestrator.
If this entity also takes on the role of a verifier as defined by the IETF Remote ATtestation procedureS (RATS) architecture, this verifier can use the references included in the MUD file specified in this document to discover, for example, appropriate reference value providers, endorsement documents or endorsement distribution APIs, trust anchor stores, remote verifier services (sometimes referred to as Attestation Verification Services), or transparency logs.
All theses references in the MUD file pointing to resources and auxiliary RATS services can satisfy general RATS prerequisite by enabling discovery or improve discovery resilience of corresponding resources or services.

--- middle

# Introduction

Verifiers, Endorsers, and Attesters are roles defined in the RATS Architecture {{-rats-arch}}.
In the RATS architecture, the Relying Party roles depend on the Verifier to bear the burden of Evidence appraisal and to generate corresponding Attestation Results for them.
Attestation Results compose a believable chunk of information that can be digested by Relying Parities in order to assess an Attester's trustworthiness.
The assessment of a remote peer's trustworthiness is vital to determine whether any future protocol interaction between a Relying Party and a remote Attester can be considered secure.
To create these Attestation Results to be consumed by Relying Parties, the Attestation Evidence an Attester generates has to be appraised by one or more appropriate Verifiers.

This document defines a procedure that enables the discovery of resources or services in support of RATS, including:

1. Reference Values,
3. Trust Anchors,
2. Endorsements and Endorsement Distribution APIs,
4. (remote) Verifier APIs,
5. Transparency Logs, or
6. Appraisal Policies.

MUD URIs can be embedded in any data item that was signed with trusted key material.
One common way to establish trust in a signed data item is to associate the signing key material with a trust anchor via a certification path (see {{RFC4949}} for trust anchor and certification path).
This document defines the use of MUD URIs embedded in two types of signed data items that typically are trusted via certification paths:

1. Secure Device Identifiers (IEEE 802.1AR DevIDs) as defined by {{-mud}} and
2. Entity Attestation Tokens (EAT) as defined by {{-eat}}.

DevIDs and EATs (essentially CWTs ) are two very prominent examples of "trustworthy documents" (TDs) with a binary format and the embedding of MUD URIs in theses TDs can be applied to other TD types, for example, Selective Disclosure CWTs {{-sd-cwt}}.
Other TDs are out-of-scope of this specification, though.
The TDs are typically enrolled on Attesters by manufacturers or provisioned by supply chain entities with appropriate authority.
The TDs can be presented to local Network Management Systems, AAA-services (e.g., via IEEE 802.1X), or other points of first contact (POFC), for example, {{RFC8071}}.
These POFC are typically trusted third parties (TTP) that can digest the TDs and then base trust decisions on the associated certification paths and trust anchors.
If a TD presented by the Attester is deemed to be trusted by a local trust authority, the MUD URI embedded is considered to be a trusted source for viable resources and services in support of remote attestation of the Attester.

This specification does not define the shape or format of any resource or service that is referenced by the MUD file.
In support of a unified mechanism to categorize the formats of referenced resources, a conceptual message wrapper (CMW, {{-cmw}} is used for each type of resource.
An example of a referenced resource is a CoRIM tag {{-corim}}.

## Requirements Notation

{::boilerplate bcp14-tagged}

# MUD URIs in Trusted Documents (TDs)

This document does neither modify nor augment the definition about how to compose a MUD URI.
The two types of trusted documents (TDs) covered by this specification are Secure Device Identifiers and Entity Attestation Tokens.

## MUD URIs in DevIDs

{{-mud}} defines the format of how to embed MUD URIs in DevIDs and that specification is used in this document.

## MUD URIs in EATs {#eat-mud-uri}

To embed a MUD URI in an EAT, the `mud-uri` claim specified in this document MUST be used.

# MUD File Signatures

As the resources required by a Verifier's appraisal procedures have to be trustworthy, a MUD signature file for a corresponding MUD File MUST be available.
The MUD File MUST include a reference to its MUD signature file via the 'mud-signature' statement.
The MUD File Signature generation as specified in {{Section 13.1 of -mud}} applies.
If a MUD file changed (i.e., the checking of the MUD File Signature fails) or the corresponding MUD File Signer certificate is expired (see {{Section 13.2 of -mud}}, the reference in the changed MUD File MUST point to a new valid MUD Signature File and that new MUD File Signature MUST be available.
If a corresponding MUD File Signer certificate is expired (see {{Section 13.2 of -mud}}) or a MUD File Signature referenced by a MUD File cannot be checked successfully, the MUD File MUST NOT be trusted.

## MUD File Signer in DevIDs

{{-mud}} defines the format of how to embed a reference to the signing certificate in DevIDs and that specification applies to this specification.

## MUD File Signer in EATs {#eat-mud-signer}

To embed a reference to a MUD File Signer in an EAT, the `mud-signer` claim specified in this document MUST be used and the `mud-uri` claim MUST be present.
The value of the `mud-signer` claim is a CBOR byte-wrapped subject field of the signing certificate of the MUD File as specified in {{Section 11 of -mud}}.

# Trusting MUD URIs and MUD Files

The level of assurance about the authenticity of a MUD URI embedded in a TD is based on the level of trust put into the corresponding trust anchor associated with the key material that signed the TD.
If it is not possible to establish a level of trust towards the entity that signed a TD, the embedded MUD URI SHOULD NOT be trusted.
In some usage scenarios it might suffice to trust a MUD File, if the referenced MUD File Signer's certificate is not expired, but that behavior is NOT RECOMMENDED.

The level of assurance about the authenticity of a MUD file is based on the level of trust put into the entity that created the corresponding MUD File Signer's certificate.
If it is not possible to establish a level of trust into the corresponding trust anchor associated with the MUD Signer's certificate, the MUD File that references that MUD Signer MUST NOT be trusted.

## Trusting RATS Resources Referenced by a MUD File

Resources, e.g., RATS Conceptual Messages, that are referenced by a MUD File MUST be signed (e.g., via a COSE_Sign1 envelope).
The signing procedures, the format of corresponding identity documents, and the establishment of trust relationships associated with these resources are out-of-scope of this document.

# Specification of RATS MUD Files Referenced by MUD URIs

The MUD URI embedded in a TD presented by an Attester points to a MUD File.
MUD URIs typically point to a piece of data that is a YANG-modeled XML file with a structure specified in the style of a YANG module definition ({{RFC7950}} and corresponding updates: {{RFC8342}}, {{RFC8526}}).
This document specifies a YANG module augment definition for generic MUD files to create RATS MUD files.
The following definition MUST be used, if a MUD URI points to a RATS MUD file.

## Tree Diagram

The following tree diagram {{RFC8340}} provides an overview of the data model for the "ietf-mud-rats" module augment.

~~~~
<CODE BEGINS>
{::include ietf-mud-rats.tree}
<CODE ENDS>
~~~~

## YANG Module

This YANG module has normative references to {{RFC6991}} and augments {{-mud}}.

~~~~ YANG
<CODE BEGINS> file ietf-mud-rats@2025-02-09.yang
{::include ietf-mud-rats.yang}
<CODE ENDS>
~~~~

# Privacy Considerations

The privacy considerations of RFC 9334 apply.

# Security Considerations

The trust model and Security Considerations of RFC 8520 and RFC 9334 apply.

# IANA Considerations

[^rfced] Please replace "{{&SELF}}" with the RFC number assigned to this document.

[^rfced] This document uses the CPA (code point allocation) convention described in {{?I-D.bormann-cbor-draft-numbers}}. For each usage of the term "CPA", please remove the prefix "CPA" from the indicated value and replace the residue with the value assigned by IANA; perform an analogous substitution for all other occurrences of the prefix "CPA" in the document. Finally, please remove this note.

## CWT `mud-uri` Claim Registration {#iana-mud-uri}

IANA is requested to add the new `mud-uri` CBOR Web Token claim to the "CBOR Web Token (CWT) Claims" registry {{IANA.cwt}} in the Standards Action Range as follows:

* Claim Name: mud-uri
* Claim Description: A CBOR byte-wrapped MUD URI as specified in {{-mud}}
* JWT Claim Name: mud-uri
* Claim Key: CPA109
* Claim Value Type(s): CBOR byte string
* Change Controller: IETF
* Specification Document(s): {{eat-mud-uri}} of {{&SELF}}

## CWT `mud-signer` Claim Registration {#iana-mud-signer}

IANA is requested to add the new `mud-signer` CBOR Web Token claim to the "CBOR Web Token (CWT) Claims" registry group {{IANA.cwt}} in the Standards Action Range as follows:

* Claim Name: mud-uri
* Claim Description: A CBOR byte-wrapped subject field of the signing certificate for a MUD file as specified in {{-mud}}
* JWT Claim Name: mud-signer
* Claim Key: CPA110
* Claim Value Type(s): CBOR byte string
* Change Controller: IETF
* Specification Document(s): {{eat-mud-signer}} of {{&SELF}}

## JWT `mud-uri` Claim Registration {#iana-mud-uri-json}

IANA is requested to add the new `mud-signer` JSON Web Token Claim to the "JSON Web Token (JWT)" registry group {{IANA.jwt}} as follows:

* Claim Name: mud-signer
* Claim Description: A MUD signer reference represented via a URI text string as defined by {{-mud}}
* Change Controller: IETF
* Specification Document(s): {{eat-mud-uri}} of {{&SELF}}

## JWT `mud-signer` Claim Registration {#iana-mud-signer-json}

IANA is requested to add the new `mud-signer` JSON Web Token Claim to the "JSON Web Token (JWT)" registry group {{IANA.jwt}} as follows:

* Claim Name: mud-signer
* Claim Description: A MUD signer reference represented via a URI text string as defined by {{-mud}}
* Change Controller: IETF
* Specification Document(s): {{eat-mud-uri}} of {{&SELF}}


--- back

[^rfced]: RFC Editor:
