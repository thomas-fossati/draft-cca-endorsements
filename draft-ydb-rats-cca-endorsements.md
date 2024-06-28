---
title: Arm's Confidential Computing Architecture (Arm CCA) Attestation Verifier Endorsements
abbrev: Arm CCA Endorsements
docname: draft-ydb-rats-cca-endorsements-latest
date: {DATE}
category: info
ipr: trust200902
area: Security
workgroup: RATS

stand_alone: yes
pi:

  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  text-list-symbols: -o*+
  docmapping: yes

author:

-
  name: Yogesh Deshpande
  org: Arm Ltd
  email: yogesh.deshpande@arm.com

-
  name: Thomas Fossati
  org: Linaro
  email: thomas.fossati@linaro.org


normative:

  CoRIM: I-D.ietf-rats-corim

informative:
  RATS-ARCH: RFC9334

  PSA-CERTIFIED:
   target: https://www.psacertified.org
   title: PSA Certified
   date: 2021

  CCA-TOKEN:
   target: https://documentation-service.arm.com/static/63a16f163f28e5456434c719?token=
   title: Realm Management Monitor specification
   date: 2022

--- abstract

Arm Confidential Computing Architecture (Arm CCA) Endorsements comprise of reference values and cryptographic key material that a Verifier needs in order to
appraise Attestation Evidence produced by Arm CCA system.

--- middle

# Introduction

Arm CCA Endorsements include reference values and cryptographic key material 
information that a Verifier needs in order to appraise
attestation Evidence produced by a CCA System {{CCA-TOKEN}}.  This memo defines
such CCA Endorsements as a profile of the CoRIM data model {{CoRIM}}.

# Conventions and Definitions

{::boilerplate bcp14}

The reader is assumed to be familiar with the terms defined in Section A7.2.1 of
{{CCA-TOKEN}} and in Section 4 of {{RATS-ARCH}}.

# Arm CCA Endorsements
{: #sec-cca-endorsements }

Arm CCA attestation scheme is a composite attestation scheme which comprises a CCA Platform Attestation & a Realm Attestation. Hence appraisal of Arm CCA attestation needs endorsements for both CCA Platform and CCA Realm. This draft documents both the CCA platform and realm endorsements.


## Arm CCA Platform Endorsements

There are three types of CCA Platform Endorsements:

* Reference Values ({{sec-ref-values}}), i.e., measurements of the CCA Platform firmware.
* Attestation Verification Claims ({{sec-keys}}), i.e., cryptographic keys
  that can be used to verify signed attestation token produced by the CCA platform, along
  with the identifiers that bind the keys to their platform instances.
* Certification status of CCA platform implementation

## Arm CCA Platform Endorsement Profile

Arm CCA platform Endorsements are carried in one or more CoMIDs inside a CoRIM.

The profile attribute in the CoRIM MUST be present and MUST have a single entry
set to the uri `http://arm.com/cca/ssd/1` as shown in.

## CCA Platform Endorsements linkage to CCA Platform
{: #sec-cca-rot-id}

Each CCA Platform Endorsement - be it a Reference Value, Attestation Verification Claim
is associated with a unique CCA platform identifier. A CCA platform
identifier known as CCA Platform Implementation ID (see Section A7.2.3.2.3 of {{CCA-TOKEN}}) 
uniquely identifies a class of CCA platform to which the manufacturer/endorser links the supplied
Endorsements (Reference Values & Attestation Verification Keys) for a CCA platform.

In order to support CCA Implementation IDs, the CoMID type
`$class-id-type-choice` is extended as follows:

Besides, a CCA Endorsement can be associated with a specific instance of a
certain CCA Platform implementation - as in the case of Attestation Verification Claims.  A CCA
Attestation Verification Claims are associated with a CCA platform instance by means of the Instance ID
(see Section 3.2.1 of {{CCA-TOKEN}}) and its platform Implementation ID.

These identifiers are typically found in the subject of a CoMID triple, encoded
in an `environment-map` as shown in .


Optional `vendor` and `model` can be specified as well.  Together, they are
interpreted as a unique identifier of the product that embeds the CCA platform.
Consistently providing a product identifier is RECOMMENDED.

## Reference Values
{: #sec-ref-values}

Reference Values carry measurements and other metadata associated with the
CCA platform. The CCA platform is a collective term used to identify all the hardware and firmware components on a CCCA enabled system. This includes
* CCA system security domain
* Monitor security domain
* Realm Management Security domain

When appraising Evidence, the Verifier
compares Reference Values against the values found in the Software Components
of the CCA platform token (see Section 3.4.1 of {{CCA-TOKEN}}).

Each measurement is encoded in a `measurement-map` of a CoMID
`reference-triple-record`.  Since a `measurement-map` can encode one or more
measurements, a single `reference-triple-record` can carry as many measurements
as needed, provided they belong to the same CCA platform identified in the subject of
the "reference value" triple.  A single `reference-triple-record` SHALL
completely describe the CCA platform RoT.

The identifier of a measured software component is encoded in a `arm-swcomp-id`
object as follows:


The semantics of the codepoints in the `arm-swcomp-id` map are equivalent to
those in the `arm-software-component` map defined in Section 3.4.1 of
{{CCA-TOKEN}}.  The `arm-swcomp-id` MUST uniquely identify a given software
component within the CCA platform / product.

In order to support CCA Reference Value identifiers, the CoMID type
`$measured-element-type-choice` is extended as follows:


and automatically bound to the `comid.mkey` in the `measurement-map`.

The raw measurement is encoded in a `digests-type` object in the
`measurement-values-map`.  The `digests-type` array MUST contain at least one
entry.  The `digests-type` array MAY contain more than one entry if multiple
digests (obtained with different hash algorithms) of the same measured
component exist.


## Attestation Verification Claims
{: #sec-keys}

An Attestation Verification Claim carries the verification key associated with
the Initial Attestation Key (IAK) of a CCA platform.  When appraising Evidence,
the Verifier uses the Implementation ID and Instance ID claims (see
{{sec-cca-rot-id}}) to retrieve the verification key that it SHALL use to check
the signature on the CCA platform token.  This allows the Verifier to prove (or disprove)
the Attester's claimed identity.

Each verification key is provided alongside the corresponding device Instance
and Implementation IDs (and, possibly, a product identifier) in an
`attest-key-triple-record`.  Specifically:

* The Instance and Implementation IDs are encoded in the environment-map as
  shown in
* The IAK public key is carried in the `comid.key` entry in the
  `verification-key-map`.  The IAK public key is a PEM-encoded
  SubjectPublicKeyInfo {{!RFC5280}}.  There MUST be only one
  `verification-key-map` in an `attest-key-triple-record`;
* The optional `comid.keychain` entry MUST NOT be set by a CoMID producer that
  uses the profile described in this document, and MUST be ignored by a CoMID
  consumer that is parsing according to this profile.

The example in  shows the CCA Endorsement
of type Attestation Verification Claim carrying a secp256r1 EC public IAK
associated with Instance ID `4ca3...d296`.

# Security Considerations

<cref>TODO</cref>

# IANA Considerations

## CBOR Tag Registrations

IANA is requested to allocate the following tag in the "CBOR Tags" registry
{{!IANA.cbor-tags}}, preferably with the specified value:

| Tag | Data Item | Semantics |
|---
| 600 | tagged bytes | CCA Implementation ID ({{sec-cca-rot-id}} of RFCTHIS) |
| 601 | tagged map | CCA Software Component Identifier ({{sec-ref-values}} of RFCTHIS) |
{: #tbl-psa-cbor-tag title="CoRIM CBOR Tags"}

## CoRIM Profile Registration

IANA is requested to register the following profile value in the
<cref>TODO</cref> CoRIM registry.

| Profile Value | Type | Semantics |
|---
| `http://arm.com/cca/ssd/1` | uri | The CoRIM profile specified by this document |
{: #tbl-psa-corim-profile title="PSA profile for CoRIM"}

## CoMID Codepoints

IANA is requested to register the following codepoints to the "CoMID Triples
Map" registry.

| Index | Item Name | Specification
|---
| 4 | comid.psa-cert-triples | RFCTHIS
| 5 | comid.psa-swrel-triples | RFCTHIS
{: #tbl-psa-comid-triples title="PSA CoMID Triples"}

# Acknowledgements
{: numbered="no"}

<cref>TODO</cref>
