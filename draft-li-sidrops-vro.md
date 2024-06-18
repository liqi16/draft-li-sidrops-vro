---
title: "A Profile for RPKI Signed Group of Multiple-Origin AS"
abbrev: "rpki-moasgroup"
category: info

docname: draft-li-sidrops-rpki-moasgroup-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "SIDR Operations"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
    venue:
    group: "SIDR Operations"
    type: "Working Group"
    mail: "sidrops@ietf.org"
    arch: "https://mailarchive.ietf.org/arch/browse/sidrops/"
    github: "liqi16/draft-li-sidrops-vro"
    latest: "https://liqi16.github.io/draft-li-sidrops-vro/draft-li-sidrops-vro.html"

author:
 -
    fullname: "LIQI"
    organization: ZGCLab
    email: "liq23@mail.zgclab.edu.cn"

normative:

informative:


--- abstract

This document defines a "Signed MOAS Group", a Cryptographic Message Syntax (CMS) protected content type for use with the Resource Public Key Infrastructure (RPKI) to authenticate the collective announcement of IP prefixes by multiple Autonomous Systems (ASes). The Signed MOAS Group includes two parts: an IP prefix and a list of ASes authorized to announce the prefix. At least one of these ASes SHOULD be authorized to announce the prefix by the prefix owner through a Route Origin Authorization (ROA). The IP prefix in the ROA must match the IP prefix in the Signed MOAS Group, and the ASN in the ROA must appear in the AS list. The validation of a Signed MOAS Group confirms that the authenticated AS and the listed ASes have collectively agreed to announce the prefix, ensuring that the announcement is legitimate, accurate, and consensually authorized.


--- middle

# Introduction

This document defines a "Signed MOAS Group", a Cryptographic Message Syntax (CMS) {{RFC5652}} {{RFC626}} protected content type to carry an IP prefix and a list of Autonomous Systems (ASes) authorized to announce this prefix. The Signed MOAS Group allows multiple ASes to collaboratively and securely announce an IP prefix, supporting scenarios such as business partnerships, IP address transfers, and DDoS protection services.

At least one ASes in the AS list SHOULD be authorized to announce the prefix by the prefix owner through a Route Origin Authorization (ROA). The IP prefix in the ROA must match the IP prefix in the Signed MOAS Group, and the ASN in the ROA must appear in the AS list. The content is collectively signed by the authorized ASes using a multi-signature technique, and the aggregated global signature is attached by the authenticated AS. This ensures that the announcement is endorsed by all participating ASes and is verifiable by any RPKI-validating remote routing entities.

To validate a MoasGroup object, a relying party (RP) aggregates the public keys of all ASes in the AS list into a global public key, which is then used to verify the Signed MOAS Group. If the Signed MOAS Group is verified and a corresponding ROA is found, the MOAS Group is considered valid. If the Signed MOAS Group is verified but no corresponding ROA is found, it is considered suspicious. If the Signed MOAS Group cannot be verified, it is considered invalid.

The Signed MOAS Group provides a mechanism for securely managing multi-origin AS announcements, offering a robust and flexible solution to handle modern routing requirements. Any prefixes announced by ASes that are not included in a ROA or a validated Signed MOAS Group SHOULD be regarded as suspicious, though their handling is subject to local routing policies. The intent is to offer a secure and authenticated method for managing MOAS scenarios, enhancing the overall security and integrity of the routing system.

# Requirements Language

{::boilerplate bcp14-tagged}

# MoasGroup eContentType

The eContentType for a MoasGroup is defined as id-ct-rpkiSignedMoasGroup, with Object Identifier (OID) 1.2.840.113549.1.9.16.1.TBD.

This OID MUST appear within both the eContentType in the encapContentInfo object and the ContentType signed attribute in the signerInfo object (see {{RFC6488}}).

# MoasGroup eContent

The content of a MoasGroup is a single IP prefix, a list of ASes, and a Route Origin Authorization (ROA). A MoasGroup is formally defined as follows:

```
RpkiSignedMoasGroup-2023
  { iso(1) member-body(2) us(840) rsadsi(113549)
   pkcs(1) pkcs9(9) smime(16) mod(0)
   id-mod-rpkiSignedMoasGroup-2024(TBD) }
   
   DEFINITIONS EXPLICIT TAGS ::=
BEGIN

IMPORTS
  CONTENT-TYPE
  FROM CryptographicMessageSyntax-2010 -- in [RFC6268]
    { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
      pkcs-9(9) smime(16) modules(0) id-mod-cms-2009(58) };

  ct-rpkiSignedMoasGroup CONTENT-TYPE ::=
  { TYPE RpkiSignedMoasGroup
   IDENTIFIED BY id-ct-rpkiSignedMoasGroup }
   
   id-ct-rpkiSignedMoasGroup OBJECT IDENTIFIER ::=
  { iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1)
  pkcs-9(9) id-smime(16) id-ct(1) TBD }
    
  RpkiSignedMoasGroup ::= SEQUENCE {
    version [0]     INTEGER DEFAULT 0,
    prefix					AddressFamilyIPAddress }
    asList      		SEQUENCE (SIZE(0..MAX)) OF ASID,
  }

  ASID ::= INTEGER (1..4294967295)

AddressFamilyIPAddress ::= SEQUENCE {
  addressFamily ADDRESS-FAMILY.&afi ({AddressFamilySet}),
  prefix        ADDRESS-FAMILY.&Prefix ({AddressFamilySet}{@addressFamily}) }

ADDRESS-FAMILY ::= CLASS {
  &afi          OCTET STRING (SIZE(2)) UNIQUE,
  &Prefix
} WITH SYNTAX { AFI &afi PREFIX-TYPE &Prefix }

AddressFamilySet ADDRESS-FAMILY ::= { addressFamilyIPv4 | addressFamilyIPv6 }

addressFamilyIPv4 ADDRESS-FAMILY ::= { AFI afi-IPv4 PREFIX-TYPE AddressesIPv4 }
addressFamilyIPv6 ADDRESS-FAMILY ::= { AFI afi-IPv6 PREFIX-TYPE AddressesIPv6 }

afi-IPv4 OCTET STRING ::= '0001'H
afi-IPv6 OCTET STRING ::= '0002'H

AddressesIPv4 ::= Prefix{32}
AddressesIPv6 ::= Prefix{128}

Prefix {INTEGER: len} ::= SEQUENCE {
  address       BIT STRING (SIZE(0..len)) }

END
    
```

## Version

The version number of the RpkiSignedMoasGroup MUST be 0.

## asList

This field contains the AS number that is authorized to originate routes to the given IP address prefixes. The AS number that ROA authorizes SHOULD be put in front of other AS numbers. The AS numbers MUST not dupliate.

## prefix

This field contains a AddressFamilyIPAddress which contains one instance of addressFamily and one instance of prefix.

### addressFamily
This field contains a OCTET STRING which is either '0001'H (IPv4) or '0002'H (IPv6).

### prefix

This field contains a BIT STRING, its length bounded through the addressFamily field. The type is a BIT STRING, see Section 2.2.3.8 of {{RFC3779}} for more information.

# MoasGroup Validation

To validate an MoasGroup, the relying party MUST perform all the validation checks specified in [RFC6488]. In addition, the RP MUST perform the following validation steps:

1. The contents of the CMS eContent field MUST conform to all of the constraints described in Section 3.
2. The RP MUST verify the signatures of the Signed MOAS Group. This involves aggregating the public keys of all ASes listed in the AS list into a global public key. The aggregated global public key is then used to verify the Signed MOAS Group's signature.
3. The RP MUST check for the existence of a corresponding Route Origin Authorization (ROA) for the IP prefix in the Signed MOAS Group. The IP prefix in the ROA MUST match the IP prefix in the Signed MOAS Group, and the ASN in the ROA MUST appear in the AS list. 
4. The Signed MOAS Group has three kinds of validation outcomes.
 - Valid: If the Signed MOAS Group is verified and at least one corresponding ROA is found, the MOAS Group is considered valid.
 - Suspicious: If the Signed MOAS Group is verified but no corresponding ROA is found, the MOAS Group is considered suspicious.
 - Invalid: If the Signed MOAS Group cannot be verified, it is considered invalid.

# Operational Considerations

To aggregate the signatures of all ASes in the AS list, the Signed MOAS Group MUST use BLS Signatures [draft-irtf-cfrg-bls-signature-05]. This ensures that the signatures can be efficiently combined into a single global signature.

The ASes in the asList that is authorized by the ROA SHOULD be placed at the beginning of the AS list, ahead of any non-authorized ASes. This ordering can improve the efficiency of the RP's validation process. It is highly RECOMMENDED that the RP only needs to verify that the first AS and the prefix can be validated by the ROA.

Multiple valid MOAS Group objects can exist which contain the same IP prefix. However, it is highly RECOMMENDED that an AS should participate in only one MOAS Group for the same IP prefix. If there is a need to modify the AS list in a MOAS Group, it is highly RECOMMENDED to revoke the current MOAS Group and sign a new one.

The construction of an 'allowlist' for a given EBGP session using RPKI MOAS Group(s) complements best practices [RFC7454] and rejecting RPKI-invalid BGP route announcements [RFC6811]. In other words, if a given BGP route is covered by an RPKI MOAS Group, but is also "invalid" from a Route Origin Validation perspective, it is recommended to reject the route announcement.

# Security Considerations

Despite the highly recommendation that a Signed MOAS Group be endorsed by at least one ROA, the data contained in a MOAS Group is self-asserted by the AS holder. This means that the presence of an AS in the MOAS Group does not inherently imply any authority from the IP prefix holder for the AS to originate a route for any prefixes. Such authority is separately conveyed in the RPKI through a ROA.


# IANA Considerations

## SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1)

IANA is requested to allocated the following in the "SMI Security for S/MIME CMS Content Type (1.2.840.113549.1.9.16.1)" registry:

| Decimal | Description               | Reference   |
| ------- | ------------------------- | ----------- |
| TBD     | Id-ct-rpkiSignedMoasGroup | draft-xxxxx |

## RPKI Signed Objects

IANA is requested to register two OIDs in the "RPKI Signed Objects" registry {{RFC6488}} as follows:

| Name | OID               | Reference   |
| ------- | ------------------------- | ----------- |
| Signed MoasGroup     | 1.2.840.113549.1.9.16.1.TBD | draft-xxxxx |

--- back

## RPKI Repository Name Schemes

IANA is requested to add the Signed MoasGroup file extension to the "RPKI Repository Name Schemes" registry {{RFC6481}} as follows:

| Filename Extension | RPKI Object               | Reference   |
| ------- | ------------------------- | ----------- |
| .smg     | Signed MoasGroup | draft-xxxxx |

## SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0)
IANA is requested to allocate the following in the "SMI Security for S/MIME Module Identifier (1.2.840.113549.1.9.16.0)" registry:

| Decimal | Description | Reference   |
| ------- | ------------------------- | ----------- |
| .TBD     | id-mod-rpkiSignedMoasGroup-2024 | draft-xxxxx |

# Appendix A: Acknowledgments
{:numbered="false"}

TODO acknowledge.

# Appendix B: Example payloads

TODO payloads.
