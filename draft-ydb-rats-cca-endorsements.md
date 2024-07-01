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

### Arm CCA Platform Endorsement Profile

Arm CCA platform endorsements are carried in one or more CoMIDs inside a CoRIM.

The profile attribute in the CoRIM MUST be present and MUST have a single entry
set to the uri `http://arm.com/cca/ssd/1` as shown in {{ex-cca-platform-profile}}.

~~~
{::include examples/platform-profile.diag}
~~~
{: #ex-cca-platform-profile title="CCA platform profile version 1, CoRIM profile" }

### Arm CCA Platform Endorsements linkage to CCA Platform
{: #sec-cca-rot-id}

Each CCA Platform Endorsement - be it a Reference Value, Attestation Verification Claim
is associated with a unique CCA platform identifier. A CCA platform
identifier known as CCA Platform Implementation ID (see Section A7.2.3.2.3 of {{CCA-TOKEN}}) 
uniquely identifies a class of CCA platform to which the manufacturer/endorser links the supplied
Endorsements (Reference Values & Attestation Verification Keys) for a CCA platform.

In order to support CCA Implementation IDs, the CoMID type
`$class-id-type-choice` is extended as follows {{ex-cca-platform-impl-id}}:

~~~
{::include cca-ext/tagged-cca-impl-id.cddl}
~~~
{: #ex-cca-platform-impl-id title="Example CCA Platform Implementation ID" }

Besides, a CCA Endorsement can be associated with a specific instance of a
certain CCA Platform implementation - as is the case of Attestation Verification Claims.  A CCA
Attestation Verification Claims are associated with a CCA platform instance by means of the Instance ID
(see Section A7.2.3.2.4 of {{CCA-TOKEN}}) and its platform Implementation ID.

These identifiers are typically found in the subject of a CoMID triple, encoded in an `environment-map` as shown in {{ex-cca-platform-id}}.

~~~
{::include examples/cca-platform-identification.diag}
~~~
{: #ex-cca-platform-id title="Example CCA Platform Identification" }

Optional `vendor` and `model` can be specified as well. Together, they are
interpreted as a unique identifier of the CCA platform.
Consistently providing a product identifier is RECOMMENDED.

### Reference Values
{: #sec-ref-values}

Reference Values carry measurements and other metadata associated with the updatable firmware of
CCA platform. The CCA platform is a collective term used to identify all the hardware and firmware components on a CCCA enabled system. This includes
* CCA system security domain
* Monitor security domain
* Realm Management Security domain

When appraising Evidence, the Verifier compares Reference Values against:
 -  the values found in the Software Components of the CCA platform token (see Section A7.2.3.2.7 of {{CCA-TOKEN}}).
 - the value set in the platform configuration of the CCA platform token (see Section A7.2.3.2.5 of {{CCA-TOKEN}}).

Each measurement is encoded in a `measurement-map` of a CoMID
`reference-triple-record`.  Since a `measurement-map` can encode one or more
measurements, a single `reference-triple-record` can carry as many measurements
as needed, provided they belong to the same CCA platform identified in the subject of
the "reference value" triple.  A single `reference-triple-record` SHALL
completely describe the CCA platform measurements.

The identifier of a measured software component is encoded in a `arm-swcomp-id` object as follows {{ex-swcomp-id}}:

~~~
{::include cca-ext/swcomp-id.cddl}
~~~
{: #ex-swcomp-id title="Example SW Component ID" }

The semantics of the codepoints in the `arm-swcomp-id` map are equivalent to those in the `cca-platform-sw-component` map defined in Section A7.2.3.2.7 of {{CCA-TOKEN}}.  The `arm-swcomp-id` MUST uniquely identify a given software component within the CCA platform / product.

In order to support CCA Reference Value identifiers, the CoMID type
`$measured-element-type-choice` is extended as follows{{ex-swcomp-id-ext}}:

~~~
{::include cca-ext/swcomp-id-ext.cddl}
~~~
{: #ex-swcomp-id-ext title="Example SW Component ID Extension" }

and automatically bound to the `comid.mkey` in the `measurement-map`.

The raw measurement is encoded in a `digests-type` object in the
`measurement-values-map`.  The `digests-type` array MUST contain at least one entry. The `digests-type` array MAY contain more than one entry if multiple digests (obtained with different hash algorithms) of the same measured component exist. Refer below {{ex-cca-platform-ident}}.

~~~
{::include examples/cca-platform-identification.diag}
~~~
{: #ex-cca-platform-ident title="Example SW Component ID Extension" }

### Attestation Verification Claims
{: #sec-keys}

An Attestation Verification Claim carries the verification key associated with
the Initial Attestation Key (IAK) of a CCA platform. When appraising Evidence,
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

## Arm CCA Realm Endorsements

Arm CCA Realm provides a protected execution environment for applications executing within a Realm. A Realm Endorsements comprise of:

* Reference Values ({{sec-realm-ref-values}}), i.e., measurements of the configuration and contents of a Realm at the time of its activation along with measurements of software running inside Realm, which can be extended during the lifetime of a Realm.

Please note that there are no Realm Trust Anchor Endorsements needed from supply chain as they are present inline in the Attestation Evidence.

### Realm Endorsements linkage to Realm

Each Realm has a unique execution context and hence a unique Realm instance. Each Realm is uniquely identified in the Arm CCA system. For a realm its Endorsements are associated to this unique instance. The realm instance is be a vendor defined variable length identifier. Hence in this profile, it is represented in a CoMID inside an `environment-map` with `$instance-id-type-choice` set to `tagged-bytes`, i.e. An opaque, variable-length byte string. In this profile of CCA Endorsements, the Realm Initial Measurements are set in `tagged-bytes` to represent Realm instance.

When supplying Realm Endorsements, a supplier of one or more Realms may wish to identify itself. Hence the following class related elements in the `environment-map` of a  `comid` can be used.

In the `class-map` select 
`vendor` name and/or `class-id` set as `UUID` representing unique identity for the Realm owner.

$class-id-type-choice /= tagged-uuid-type

vendor => `tstr` to represent vendor name

### Arm CCA Realm Endorsement Profile

Arm CCA realm Endorsements are carried in a CoMID inside a CoRIM.

The profile attribute in the CoRIM MUST be present and MUST have a single entry
set to the uri `http://arm.com/cca/realm/1` as shown in {{ex-cca-realm-profile}}.

~~~
{::include examples/realm-profile.diag}
~~~
{: #ex-cca-realm-profile title="CCA realm profile version 1, CoRIM profile" }

### Reference Values
{: #sec-realm-ref-values}

Reference Values carry measurements and other metadata associated with the
CCA Realm.

Realm reference values comprise of:
1. Realm Initial Measurements (RIM)
2. Realm Extended Measurements (REMs)  
3. Realm Personalization Value (RPV)

RIM and REMs are encoded in a `measurement-values-map` (in a `measurement-map`) of a CoMID `reference-triple-record`. Inside `measurement-values-map` these measurements are carried as `integrity-registers` map. Integrity Registers map is used to group together one or more measured objects pertaining to an environment. Please refer {{CoRIM}} for details about Integrity Register map.

All the measured objects in an Integrity Registers map are explicitly named. In the context of Realms, the measured objects are RIM and REMs. Inside Integrity Register map, RIM is uniquely identified by the name "rim", while REMs which is an array of measurements from 1..4 are uniquely identified by the coresponding name "rem0".."rem3".

Realm Personalization Value, (RPV) is an optional identity used by a Realm endorser to uniquely identify multiple Realms which all have same RIM. RPV if provided is a fixed length 64 bytes identifier. In this profile, RPV is represented using Raw Value Measurements in a `measurement-values-map`, with raw value type choice set to `tagged-bytes`.

$raw-value-type-choice /= tagged-bytes

Given below is the complete example of a Realm Endorsements.

# Security Considerations

<cref>TODO</cref>

# IANA Considerations

## CBOR Tag Registrations

IANA is requested to allocate the following tags in the "CBOR Tags" registry
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
{: #tbl-cca-corim-platform-profile title="CCA platform profile for CoRIM"}


| Profile Value | Type | Semantics |
|---
| `http://arm.com/cca/realm/1` | uri | The CoRIM profile specified by this document |
{: #tbl-cca-corim-realm-profile title="CCA realm profile for CoRIM"}

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
