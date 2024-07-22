---
v: 3

title: CoRIM profile for AMD SEV-SNP attestation report
abbrev: CoRIM-SEV
docname: draft-ietf-rats-amd-sev-snp-corim-profile-latest
category: std
consensus: true
submissiontype: IETF

ipr: trust200902
area: "Security"
workgroup: "Remote ATtestation ProcedureS"
keyword: RIM, RATS, attestation, verifier, supply chain

stand_alone: true
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes
  tocdepth: 6

author:
- ins: D. Glaze
  name: Dionna Glaze
  org: Google LLC
  email: dionnaglaze@google.com

contributor:
- ins: Y. Deshpande
  name: Yogesh Deshpande
  organization: arm
  email: yogesh.deshpande@arm.com
  contribution: >
      Yogesh Deshpande contributed to the data model by providing advice about CoRIM founding principles.

normative:
  RFC8174:
  RFC8610: cddl
  RFC9334: rats-arch

informative:
  I-D.ietf-rats-corim: rats-corim
  SEV-SNP.API:
    title: SEV Secure Nested Paging Firmware ABI Specification
    author:
      org: Advanced Micro Devices Inc.
    seriesinfo: Revision 1.55
    date: September 2023
    target: https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/specifications/56860.pdf
  GHCB:
    title: SEV-ES Guest-Hypervisor Communication Block Standardization
    author:
      org: Advanced Micro Devices Inc.
    seriesinfo: Revision 2.03
    date: July 2023
    target: https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/specifications/56421.pdf

entity:
  SELF: "RFCthis"

--- abstract

AMD Secure Encrypted Virtualization with Secure Nested Pages (SEV-SNP) attestation reports comprise of reference values and cryptographic key material that a Verifier needs in order to appraise Attestation Evidence produced by an AMD SEV-SNP virtual machine.
This document specifies the information elements for representing SEV-SNP Reference Values in CoRIM format.

--- middle

# Introduction {#sec-intro}

TODO: write after content.

#  Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only when, they appear in all capitals, as shown here.

The reader is assumed to be familiar with the terms defined in {{-rats-corim}} and Section 4 of {{-rats-arch}}.
The syntax of data descriptions is CDDL as specified in {{-cddl}}.

# AMD SEV-SNP Attestation Reports

The AMD SEV-SNP attestation scheme in [SEV-SNP.API] contains measurements of security-relevant configuration of the host environment and the launch configuration of a SEV-SNP VM.
This draft documents the normative representation of attestation report Evidence as a CoRIM profile.

AMD-SP:
  AMD Secure Processor.
  A separate core that provides the confidentiality and integrity properties of AMD SEV-SNP.
  The function that is relevant to this document is its construction of signed virtual machine attestation reports.

VCEK:
  Versioned Chip Endorsement Key.
  A key for signing the SEV-SNP Attestation Report.
  The key is derived from a unique device secret as well as the security patch levels of relevant host components.

VLEK:
  Version Loaded Endorsement Key.
  An alternative SEV-SNP Attestation Report signing key that is derived from a secret shared between AMD and a Cloud Service Provider.
  The key is encrypted with a per-device per-version wrapping key that is then decrypted and stored by the AMD-SP.

## AMD SEV-SNP CoRIM Profile

AMD SEV-SNP launch endorsements are carried in one or more CoMIDs inside a CoRIM.

The profile attribute in the CoRIM MUST be present and MUST have a single entry set to the uri http://amd.com/please-permalink-me as shown in {#figure-profile}.


{% figure caption:"SEV-SNP attestation profile version 1, CoRIM profile" %}

~~~cbor-diag
/ corim-map / {
  / corim.profile / 3: [
    32("http://amd.com/please-permalink-me")
  ]
  / ... /
}
~~~
{% endfigure %}

### AMD SEV-SNP Target Environment

The `ATTESTATION_REPORT` structure as understood in the RATS Architecture [RFC9334] is a signed collection of Claims that constitute Evidence about the Target Environment.
The Attester for the `ATTESTATION_REPORT` is specialized hardware that will only run AMD-signed firmware.

The `class-id` for the Target Environment measured by the AMD-SP is the tagged OID `#6.111(1.3.6.1.4.1.3704.2.1)`.
The launched VM on SEV-SNP has an ephemeral identifier `REPORT_ID`.
If the VM is the continuation of some instance as carried by a migration agent, there is also a possible `REPORT_ID_MA` value to identify the instance.
The attester, however, is always on the same `CHIP_ID`.
Given that the `CHIP_ID` is not uniquely identifying for a VM instance, it is better classified as a group.
The `CSP_ID` is similarly better classified as a group.
Either the `CHIP_ID` or the `CSP_ID` may be represented in the `group` codepoint as a tagged-bytes.
If the `SIGNING_KEY` bit of the attestation report is 1, then the `group` MUST be the `CSP_ID` of the VLEK.

~~~cbor-diag
/ environment-map / {
/ class-map / {
   / class-id: / 0 => #6.111(1.3.6.1.4.1.3704.2.1)
 }
/ instance: / 1 => #6.563({
  / report-id: / 0 => REPORT_ID,
  / report-id-ma: / 1 => REPORT_ID_MA
  })
/ group: / 2 => #6.560(CHIP_ID)
 }
~~~

### AMD SEV-SNP Attestation Report measurement values extensions

The fields of an attestation report that have no direct analog in the base CoRIM CDDL are given negative codepoints to be specific to this profile.

The `GUEST_POLICY` field's least significant 16 bits represent a Major.Minor minimum version number:

```cddl
{:include cddl/sevsnpvm-policy-record.cddl}
```

The policy's minimum ABI version is assigned codepoint -1:

```cddl
{:include cddl/sevsnpvm-policy-abi-ext.cddl}
```

The attestation report's `VMPL` field is assigned codepoint -2:

~~~cddl
{:include cddl/sevsnpvm-vmpl-ext.cddl}
~~~

The attestation report's `HOST_DATA` is assigned codepoint -3:

~~~cddl
{:include cddl/sevsnpvm-hostdata-ext.cddl}
~~~

The SEV-SNP firmware build number and Minor.Minor version numbers are provided for both the installed and committed versions of the firmware to account for firmware hotloading.
The three values are captured in a record type `sevsnphost-sp-fw-version-record`:

~~~cddl
{:include cddl/sevsnphost-sp-fw-version-record.cddl}
~~~

The current build/major/minor of the SP firmware is assigned codepoint -4:

~~~cddl
{:include cddl/sevsnphost-spfw-current-ext.cddl}
~~~

The committed build/major/minor of the SP firmware is assigned codepoint -5:

~~~cddl
{:include cddl/sevsnphost-spfw-committed-ext.cddl}
~~~

The host components other than AMD SP firmware are relevant to VM security posture, so a combination of host components' security patch levels are included as TCB versions.
The TCB versions are expressed as a 64-bit number where each byte corresponds to a different component's security patch level.
Reference value providers may provide minimum values for each component, or they can provide overall minimum values with a `sevsnphost-tcb-version-type-choice`:

~~~cddl
{:include cddl/sevsnphost-tcb-version-type-choice.cddl}
{:include cddl/sevsnphost-tcb-map.cddl}
~~~

The `svn` value in each component must be within `0..255`.

The `CURRENT_TCB` version of the host is assigned codepoint -6:

~~~cddl
{:include cddl/sevsnphost-current-tcb-ext.cddl}
~~~

The `COMMITTED_TCB` version of the host is assigned codepoint -7:

~~~cddl
{:include cddl/sevsnphost-committed-tcb-ext.cddl}
~~~

The `LAUNCH_TCB` version of the host is assigned codepoint -8:

~~~cddl
{:include cddl/sevsnphost-launch-tcb-ext.cddl}
~~~

The `REPORTED_TCB` version of the host is assigned codepoint -9:

~~~cddl
{:include cddl/sevsnphost-launch-tcb-ext.cddl}
~~~

The `GUEST_POLICY` boolean flags are added as extensions to `$$flags-map-extension`.

There are 47 available bits for selection when the mandatory 1 in position 17 and the ABI Major.Minor values are excluded from the 64-bit `GUEST_POLICY`.
The `PLATFORM_INFO` bits are host configuration that are added as extensions to `$$flags-map-extension` starting at `-49`.

TODO: flags extensions


#### Notional Instance Identity {#sec-id-tag}

A CoRIM instance identifier is universally unique, but there are different notions of identity within a single attestation report that are each unique within their notion.
A notional instance identifier is a tagged CBOR map from integer codepoint to opaque bytes.

~~~ cddl
{:include cddl/int-bytes-map.cddl}
~~~

Profiles may restrict which integers are valid codepoints, and may restrict the respective byte string sizes.
For this profile, only codepoints 0 and 1 are valid.
The expected byte string sizes are 32 bytes.
For the `int-bytes-map` to be an interpretable extension of `$instance-id-type-choice`, there is `tagged-int-bytes-map`:

~~~ cddl
{:include cddl/tagged-int-bytes-map.cddl}
~~~

### AMD SEV-SNP Evidence Translation

A Verifier MAY choose to elide `CHIP_ID` from the environment.

## AMD SEV-SNP Launch Event Log

The composition of a SEV-SNP VM may be comprised of measurements from multiple principals, such that no one principal has absolute authority to endorse the overall measurement value represented in the attestation report.
If one principal does have that authority, the `ID_BLOCK` mechanism provides a convenient launch configuration endorsement mechanism without need for distributing a CoRIM.
This section documents an event log format the Virtual Machine Monitor may construct at launch time and provide in the data pages of an extended guest request, as documented in [GHCB].

# IANA Considerations

## New CBOR Tags

IANA is requested to allocate the following tags in the "CBOR Tags" registry {{!IANA.cbor-tags}}.
The choice of the CoRIM-earmarked value is intentional.

| Tag | Data Item   | Semantics                                                        | Reference |
| --- | ---------   | ---------                                                        | --------- |
| 563 | `map`       | Keys are always int, values are opaque bytes, see {{sec-id-tag}} | {{&SELF}} |

