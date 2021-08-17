# Eclipse Arrowhead X.509 Certificate Profiles

## Abstract

This document specifies how X.509 certificates are to be configured, issued and validated to facilitate secure identification and communication within and between Eclipse Arrowhead local clouds, when such security is desired and the kind of certificate is relevant.

## 1. Introduction

TODO

### 1.1 Relation to the IETF RFC 5280 X.509 Profile

All certificate profiles in this document, with the exception of the _Manufacturer_ and _Transfer_ profiles of Sections X and Y, are _required_ to be strict subsets of the RFC 5280 X.509 profile, which regulates the use of X.509 certificates on the World Wide Web.
Most significantly, this makes our profiles compatible with many implementations of the TLS and DTLS standards for secure communication, as well as many with many web programming frameworks.

### 1.2 Significant Terminology

The following subsections represent technical domains with particular bearing on this document.
Each subsection briefly describes the domain and lists terms and abbreviations relevant to our purposes.

#### 1.2.1 Eclipse Arrowhead

Service-oriented architecture for Industry 4.0 automation.

- __Device__: A physical machine that could be capable of hosting Arrowhead _systems_.
- __Local Cloud__: A physical protected network consisting of communicating _systems_.
- __Service__: An explicitly defined network application interface accessible to authorized _systems_.
- __System__: A software application providing Arrowhead-compliant _services_ that runs on a _device_.

#### 1.2.2 X.509

Certificate standard for establishing trust between devices over untrusted computer networks.

- __Certificate Authority (CA)__: Entity issuing (signing) other certificates to endorse their validity.
- __Certificate Chain__: A chain consisting of an _end entity_ certificate, its _issuer_'s certificate, that _issuer_'s _issuer_'s certificate, and so on up to the _root CA_'s certificate.
- __Certificate Revocation List (CRL)__: A list of certificates, maintained their _issuer_, that identifies certificates who no longer are to be considered valid, even though they are yet to expire.
- __End Entity__: Entity having but not issuing certificates.
- __Entity__: Any thing or being potentially able to hold and use an X.509 certificate.
- __Intermediary CA__: CA that _did not_ issue its own certificate and, therefore, can be trusted by explicitly trusting another certificate further up its issuance hierarchy.
- __Issuer__: The CA that signed a given certificate.
- __Public Key Infrastructure (PKI)__: The structure of entities, each having a certain role, required to facilitate secure certificate distribution.
- __Root CA__: CA that _did_ issue (self-sign) its own certificate and, therefore, must be explicitly trusted.
- __Subject__: The entity owning and being described by a given certificate.
- __Trust Anchor__: Another name for _root CA_.

#### 1.2.3 X.501

Naming schema for X.500 directories.
The standard is used to name the subjects and issuers of X.509 certificates.

- __Distinguished Name (DN)__: A hierarchical naming format composed consisting of RDNs. An example of a DN could be `O=My Company,CN=Robert Robertson+E=robert@mail.com`. The `O` RDN is at the highest hierarchical level, while the `CN+E` RDN is at the level below it. The comma character `,` is used to delimit the RDNs.
- __Relative Distinguished Name (RDN)__: A list of attribute/value pairs belonging to the same hierarchical level in a DN. Examples of RDNs could be `O=My Company` and `CN=Robert Robertson+E=robert@mail.com`. The first RDN consists of a single pair while the second consists of two delimited by `+`.

#### 1.2.4 ASN.1

Interface description language for describing messages that can be sent or received over computer networks using several associated encodings.
The standard is used to describe the structure of X.509 certificates, which _must_ be encoded using ASN.1 DER, as defined below.

- __Basic Encoding Rules (BER)__: Binary ASN.1 encoding that appends basic type and length information to each encoded value, which means that decoding a given message does not require knowledge of its original ASN.1 description. Defined in X.690.
- __Distinguished Encoding Rules (DER)__: A subset of BER that guarantees canonical representation, which is to say that every pair of structurally equivalent ASN.1 messages can be represented in DER in exactly one way. Must be used when encoding X.509 certificates. Defined in X.690.
- __Object Identifier (OID)__: A structured universally unique identifier, useful for identifying parts of ASN.1 messages.
- __Octet__: An 8-bit byte.

### 1.3 Conventions

The words __must__, __must not__, __required__, __should__, __should not__, __recommended__, __may__, and __optional__ in this document are to be interpreted as follows: __must__ and __required__ denote absolute requirements that must be adhered to by profile adherents; __must not__ denotes an absolute prohibition; __should__, __should not__ and __recommended__ denote recommendations that should be deviated from only if special circumstances make it relevant; and, finally, __may__ and __optional__ denote something being truly optional.

## 2. Certificate Profiles

TODO

## 2.1 Certificate Format

This section introduces the X.509 certificate format in its ASN.1 syntax.
It describes each of its fields and states how they should be used, if at all, by conforming certificates.
It does not rigorously define all fields, however.
Advanced learners and certificate software implementors should consult the official sources for more details.

The ASN.1 syntax of the third version of the X.509 certificate is defined in as follows in RFC 5280:

```asn1
    Certificate ::= SEQUENCE {
        tbsCertificate       TBSCertificate,
        signatureAlgorithm   AlgorithmIdentifier,
        signatureValue       BIT STRING
    }

    TBSCertificate ::= SEQUENCE {
        version         [0]  EXPLICIT Version DEFAULT v1,
        serialNumber         INTEGER,
        signature            AlgorithmIdentifier,
        issuer               Name,
        validity             Validity,
        subject              Name,
        subjectPublicKeyInfo SubjectPublicKeyInfo,
        issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL,
        subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
        extensions      [3]  EXPLICIT Extensions OPTIONAL
    }
```

The `Certificate` structure itself consists only of a `TBSCertificate`, a signature algorithm identifier and a concrete signature.
The signature is meant to be produced by the issuer of the certificate and serves to protect the integrity of the certificate under the assumption that the issuer is trusted.
All data protected by the signature is collected in the `TBSCertificate`.
Each of its fields is described in the following subsections.

#### 2.1.1 `version`

X.509 version of the certificate.
Must be `v3(2)` for all certificates that utilize certificate extensions, as described in Section 2.1.9.
Supporting `v1(0)` and `v2(1)` at all is _optional_.

#### 2.1.2 `serialNumber`

A positive integer of no more than 20 octets uniquely assigned by the issuer of the certificate.

__Security recommendation__.
The serial number should be exactly 20 octets long and be produced by a cryptographic random number generator.
This serves both to (A) make the certificate more resistant to collision attacks and (B) to prevent the serial number from leaking important details about the certificate issuer, such as how many certificates it has issued, how long the signing process took, and so on.
If certificate size would be a major concern, for example when certificates are used by significantly constrained devices, fewer octets _may_ be used.
Note that the serial number is signed and must be positive, which means that the most significant bit of its first ASN.1 BER `INTEGER` octet must be 0.

#### 2.1.3 `signature`

An identifier associated with the cryptographic signature algorithm used to produce the `signatureValue` in the enclosing `Certificate`, as well as any parameters made relevant by the algorithm type.
Examples of algorithms could be _RSA with SHA256_ or _ED25519_.

This field must contain the same algorithm as the `signatureAlorithm` field in the enclosing `Certificate`, as described in the beginning of Section 2.1.

#### 2.1.4 `issuer`

The DN of the issuer of the certificate in question.
More specifically, this field contains an exact copy of the `subject` field of Section 2.1.6 from the certificate of its issuer.

#### 2.1.5 `validity`

The period during which the certificate _may_ remain active and its issuer is bound to know whether or not the certificate has been revoked or not.
Denoted as a duration between two timestamps, `notBefore` and `notAfter`, as show below.

```asn1
Validity ::= SEQUENCE {
    notBefore   Time,
    notAfter    Time
}

Time ::= CHOICE {
    utcTime     UTCTime,
    generalTime GeneralizedTime
}
```

For each of these two dates, the date in question _must_ be encoded as a `UTCTime` if its year is less than or equal to 2049, or as a `GeneralizedTime` if the year is equal to or greater than 2050.
More details about the validity format may be read in Section 4.1.2.5 of RFC 5280.

#### 2.1.6 `subject`

The DN used to describe the owner of the certificate.
The field is defined as follows:

```asn1
Name ::= CHOICE {
    rdnSequence RDNSequence
}

RDNSequence ::= SEQUENCE OF RelativeDistinguishedName

RelativeDistinguishedName ::= SET SIZE (1..MAX) OF AttributeTypeAndValue

AttributeTypeAndValue ::= SEQUENCE {
    type  AttributeType,
    value AttributeValue
}

AttributeType ::= OBJECT IDENTIFIER

-- Concrete type determined by associated `AttributeType`.
-- In our case that type will always be `DirectoryString`.
AttributeValue ::= ANY

DirectoryString ::= CHOICE {
     teletexString   TeletexString   (SIZE (1..MAX)),
     printableString PrintableString (SIZE (1..MAX)),
     universalString UniversalString (SIZE (1..MAX)),
     utf8String      UTF8String      (SIZE (1..MAX)),
     bmpString       BMPString       (SIZE (1..MAX)) 
}
```

The below `AttributeType`s _must_ be supported by compliant software implementations, as required by RFC 5280.
Supporting them means that that their OIDs are known to be associated with values of type `DirectoryString`, as defined above.
Failing to support them means that the certificate validation procedure of RFC 5280, Section 6, cannot be executed.
See RFC 5280 Section 4.1.2.4 for more attributes that _should_ be supported.

| Attribute Type               | Abbreviation | OID |
|:-----------------------------|:-------------|:----|
| Common Name                  | `CN`         | `2.5.4.3`
| Country                      | `C`          | `2.5.4.6` 
| Distinguished Name Qualifier | `DN`         | `2.5.4.46`
| Domain Component             | `DC`         | `0.9.2342.19200300.100.1.25`
| Organization                 | `O`          | `2.5.4.10`
| Organizational Unit          | `OU`         | `2.5.4.11`
| Serial Number                | `SN`         | `2.5.4.5`
| State or Province Name       | `ST`         | `2.5.4.8`

#### 2.1.7 `subjectPublicKeyInfo`

The public key of the subject, as well as the identity of the algorithm associated with it.
The field is defined as follows.

```asn1
SubjectPublicKeyInfo ::= SEQUENCE {
    algorithm        AlgorithmIdentifier,
    subjectPublicKey BIT STRING
}
```

See Section 3 for more information about what public key algorithms should be used and supported.

#### 2.1.8 `issuerUniqueID` and `subjectUniqueID`

Optional identifiers uniquely associated with the issuer or subject of the certificate.

These fields _must not_ be used.
When a certain certificate, or its subject, needs to be identified, the cryptographic hash of the certificate, or its _fingerprint_, _should_ be used.
If desired to identify the subject by its public key, which _may_ be used in multiple certificates, the cryptographic hash of the value bytes of the `subjectPublicKey` field of `subjectPublicKeyInfo` _should_ be used.
See Section 3 for more information about what hashing algorithms should be used for this and other purposes.

#### 2.1.9 `extensions`

A list of certificate extensions, as defined below.

```asn1
Extensions ::= SEQUENCE SIZE (1..MAX) OF Extension

Extension  ::= SEQUENCE  {
    extnID     OBJECT IDENTIFIER,
    critical   BOOLEAN DEFAULT FALSE,
    extnValue  OCTET STRING
}
```

RFC 5280 explicitly outlines 17 different X.509 extensions, listed below.
They are categorized for the sake of clarity.

| Extension                    | `extnID` (OID)       | Brief Description |
|:-----------------------------|:---------------------|:------------------|
| __Key Extensions__           |                      |
| Authority Key Identifier     | `2.5.29.35`          | Identifier uniquely associated with certificate issuer.
| Subject Key Identifier       | `2.5.29.14`          | Identifier uniquely associated with certificate subject.
| Key Usage                    | `2.5.29.15`          | Public key usage restrictions.
| Extended Key Usage           | `2.5.29.37`          | Additional public key usage restrictions.
| __Policy Extensions__        |                      |
| Certificate Policies         | `2.5.29.32`          | List of policies accepted by the certificate subject.
| Policy Mappings              | `2.5.29.33`          | List of policy equivalence relations.
| Policy Constraints           | `2.5.29.36`          | Policy validation constraints (e.g. policy X must be accepted).
| Inhibit `anyPolicy`          | `2.5.29.54`          | Policy validation constraint related to use of `anyPolicy`.
| __Name Extensions__          |                      |
| Subject Alternative Name     | `2.5.29.17`          | Additional names associated with the certificate subject.
| Issuer Alternative Name      | `2.5.29.18`          | Additional names associated with the certificate issuer.
| Name Constraints             | `2.5.29.30`          | List of subject name restrictions for issued certificates.
| __CRL Extensions__           |                      |
| CRL Distribution Points      | `2.5.29.31`          | List of where relevant CRLs can be found.
| Freshest CRL                 | `2.5.29.46`          | List of where delta CRLs can be found.
| __Information Extensions__   |                      |
| Authority Information Access | `1.3.6.1.5.5.7.1.1`  | List of where issuer information and services can be found.
| Subject Information Access   | `1.3.6.1.5.5.7.1.11` | List of where subject information and services can be found.
| __Other Extensions__         |                      |
| Subject Directory Attributes | `2.5.29.9`           | Additional subject identification attributes (e.g. nationality).
| Basic Constraints            | `2.5.29.19`          | Description of where a certificate belongs in its CA hierarchy.

Each category of extensions is described in the following subsections.

#### 2.1.9.1 Key Extensions

The _Authority Key Identifier_ and _Subject Key Identifier_ extensions are used to identify the public key of the issuer and subject of a given certificate, respectively.
The extensions are defined as follows:

```asn1
-- Required for all but self-signed CA certificates.
AuthorityKeyIdentifier ::= SEQUENCE {
    keyIdentifier             [0] KeyIdentifier           OPTIONAL, -- Required.
    authorityCertIssuer       [1] GeneralNames            OPTIONAL, -- Optional.
    authorityCertSerialNumber [2] CertificateSerialNumber OPTIONAL  -- Optional.
}

-- Required for all but end entity certificates.
SubjectKeyIdentifier ::= KeyIdentifier

KeyIdentifier ::= OCTET STRING
```

RFC 5280 _requires_ that `AuthorityKeyIdentifier` extension is included in all CA certificates except for self-signed such.
The `keyIdentifier` of the `AuthorityKeyIdentifier` extension of a given certificate _must_ match the `SubjectKeyIdentifier` of its issuer's certificate, or _may_ be omitted if the certificate is self-signed.
Use of the `authorityCertIssuer` and `authorityCertSerialNumber` fields of `AuthorityKeyIdentifier` is optional.
If a self-signed certificate leaves the `authorityCertIssuer` and `authorityCertSerialNumber` fields unspecified, it may omit the `AuthorityKeyIdentifier` extension entirely.
RFC 5280 further _requires_ that all but end entity certificates use the `SubjectKeyIdentifier` extension.
Its value _should_ be the the cryptographic hash of the `subjectPublicKey` value (excluding the tag, length, and number of unused bits) of the `subjectPublicKeyInfo` field.

The _Key Usage_ and _Extended Key Usage_ extensions are defined as follows:

```asn1
KeyUsage ::= BIT STRING {
    digitalSignature (0),
    nonRepudiation   (1), -- Also known as `contentCommitment`.
    keyEncipherment  (2),
    dataEncipherment (3),
    keyAgreement     (4),
    keyCertSign      (5),
    cRLSign          (6),
    encipherOnly     (7),
    decipherOnly     (8)
}

ExtKeyUsageSyntax ::= SEQUENCE SIZE (1..MAX) OF KeyPurposeId

KeyPurposeId ::= OBJECT IDENTIFIER
```

The `KeyUsage` extension is a set of bit flags used to indicate various ways in which a certificate _may_ be used.
Please refer to RFC 5280 Section 4.2.1.3 for descriptions of each bit flag.

The `ExtKeyUsageSyntax` extension has the same purpose, but is open-ended in the sense that it allows for any OID to be used as an indication of what a certain certificate _may_ be used for.
RFC 5280 lists six such identifiers, out of which two has particular bearing on this document, namely the `serverAuth` (OID `1.3.6.1.5.5.7.3.1`) and `clientAuth` (OID `1.3.6.1.5.5.7.3.2`) identifiers, which indicate that the certificate holder should be allowed to act as a World Wide Web TLS server and client, respectively.

#### 2.1.9.2 Policy Extensions

In the context of X.509 and RFC 5280, a _policy_ is a legal document that binds the owner of a certificate and, potentially, all certificates issued by that certificate to certain legal obligations.
The four policy extensions defined by RFC 5280 _may_ be used to ensure that every certificate in a given certificate chain have accepted some policies of concern.
Please refer to RFC 5280 for more details about these extensions.
Their use is _optional_.

#### 2.1.9.3 Name Extensions

The _Subject Alternative Name_ and _Issuer Alternative Name_ allows for additional identities to be associated with a given `subject` or `issuer` name.
Such additional identities significantly include DNS names, IP addresses and e-mail addresses.
For example, given that some system receives a signed message and the certificate associated with that signature, the system can verify that it received the message via one of the identities listed as subject alternative name in that certificate.
The extensions are defined as follows:

```asn1
SubjectAltName ::= GeneralNames
IssuerAltName  ::= GeneralNames

GeneralNames ::= SEQUENCE SIZE (1..MAX) OF GeneralName

GeneralName ::= CHOICE {
    otherName                 [0] OtherName,
    rfc822Name                [1] IA5String,
    dNSName                   [2] IA5String,
    x400Address               [3] ORAddress,
    directoryName             [4] Name,
    ediPartyName              [5] EDIPartyName,
    uniformResourceIdentifier [6] IA5String,
    iPAddress                 [7] OCTET STRING,
    registeredID              [8] OBJECT IDENTIFIER
}
```

The _Name Constraints_ extension makes it possible for a CA to restrict the set of allowed `subject` and `SubjectAltName` that may be specified in certificates it issues.
Please refer to RFC 5280 Section 4.2.1.10 for more details.

#### 2.1.9.4 CRL Extensions

The CRL extensions facilitate a mechanism for revoking certificates that are still valid and distributing information about those revocations.
These extensions _should not_ be used to facilitate the revocation of end entity certificates.
They _may_ be used for revoking CA certificates.

Eclipse Arrowhead comes with its own certificate revocation procedure via its _Certificate Authority_ system.
Its use is _recommended_ for revoking and verifying the validity of end entity certificates.

#### 2.1.9.5 Information Extension

The information extensions allows various types of data sources or services to be associated with the certificate holder.
These extensions _should not_ be used.

Eclipse Arrowhead a service-oriented architecture with its own provisions for metadata and service management.
Those provisions _should_ be used when possible.

#### 2.1.9.6 Other Extensions

The _Subject Directory Attributes_ allows for arbitrary identification attributes, such as nationality, to be associated with the `subject` of a certificate.
It _may_ be used.

The _Basic Constraints_ extension allows for it to be denoted whether or not a given certificate belongs to a CA, as well as how many intermediary CAs may exist below it in any given certificate chain.
The extension is defined as follows:

```asn1
BasicConstraints ::= SEQUENCE {
    cA                BOOLEAN DEFAULT FALSE,
    pathLenConstraint INTEGER (0..MAX) OPTIONAL
}
```

The extension _must_ be used by all Arrowhead-compliant certificates.
The `pathLenConstraint` _must_ be set by all CA certificates.

### 2.2 Certificate Hierarchy

There are eight _primary_ and two _secondary_ certificate profiles defined in this document, depicted in the diagram further below.
The boxes of the diagram represent certificate profiles.
The arrows in the diagram are to be read as "issued by", meaning that the certificate profile from which the arrow extends must be issued (signed) by a certificate with the profile pointed to.

```
    Secondary                                      Primary

                                              + - - - - - - - +
                                              .    (Root)     .
                                              + - - - - - - - +
                                                      A
                                                      .
                                                      .
                                              +---------------+
                                              |    Master     |
                                              +---------------+
                                                      A
                                                      |
                                                      +--------------------+
                                                      |                    |
                                              +-------+-------+    +-------+-------+ 
                                              | Organization  |    |     Gate      | 
                                              +---------------+    +---------------+ 
                                                      A
                                                      |
+-------+-------+                             +-------+-------+
| Manufacturer  |                             |  Local Cloud  |
+---------------+                             +---------------+
        A                                             A
        |                                             |
        |                  +-----------------+--------+--------+-----------------+
        |                  |                 |                 |                 | 
+-------+-------+   +------+------+   +------+------+   +------+------+   +------+------+
|   Transfer    |   | On-Boarding |   |   Device    |   |   System    |   |  Operator   |  End Entity Certificates
+---------------+   +-------------+   +-------------+   +-------------+   +-------------+
```

The certificate constraints presented throughout the rest of this section only apply to the primary certificates.
The secondary certificates are considered only for their indirect relation to the _On-Boarding_ certificate.
Their only constraints are that the _Manufacturer_ certificate _must_ have issued the _Transfer_ certificate and that they pass all basic X.509 validation checks.

### 2.3 CA Certificates

#### 2.3.1 Common Fields

#### 2.3.1.1 `validity`

TODO: Write something about sensible validity periods for certs.

#### 2.3.1.2 `subject`

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| Organization (O)  | `2.5.4.6`  | No       | Human-readable name of owning organization.
| DN Qualifier (DN) | `2.5.4.46` | Yes      | _Profile identifier_ specific to each certificate profile.
| Common Name (CN)  | `2.5.4.3`  | Yes      | Human-readable name of certificate.

#### 2.3.1.3 `extensions`

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Authority Key Identifier | `2.5.29.35` | No       | See Section 2.1.9.1.
| Subject Key Identifier   | `2.5.29.14` | No       | See Section 2.1.9.1.
| Basic Constraints        | `2.5.29.19` | Yes      | See Section 2.1.9.6.
| Key Usage                | `2.5.29.15` | Yes      | Bits `keyCertSign(5)` and `cRLSign(6)` _must_ be set. See Section 2.1.9.1.

### 2.3.2 Profiles

#### 2.3.2.1 Master

Issued by any suitable RFC 5280 certificate, or none at all (self-signed).

Validity?

`subject`

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `master` | 

`extensions`

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 2` |

### 2.3.2.2 Organization

Issued by a _Master_ certificate.

Validity

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| Organization (O)  | `2.5.4.6`  | No       | Human-readable name of owning organization. |
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `organization` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | Human-readable name of certificate. |

#### Extensions

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 1` |
| KeyUsage         | `2.5.29.15` | Yes      | `keyCertSign(5), cRLSign(6)` |

### 3.4 Local Cloud

#### Issuer

An _Organization_ certificate.

#### Subject

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `localcloud` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

#### Extensions

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 0` |
| KeyUsage         | `2.5.29.15` | Yes      | `keyCertSign(5), cRLSign(6)` |


### 2.3 End Entity Certificates

#### 2.2.1 `validity`

TODO: Write something about sensible validity periods for certs.

#### 2.2.2 `subject`

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| Organization (O)  | `2.5.4.6`  | No       | Human-readable name of owning organization. |
| DN Qualifier (DN) | `2.5.4.46` | Yes      | _Identifier_ specified for each profile. | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

__Entity naming__.
Some end entity certificates, as mentioned in their respective profile sections, must contain a subject Common Name (CN) that is a valid DNS label (https://datatracker.ietf.org/doc/html/rfc1035#section-2.3.1) of no more than 62 characters.
While up to 62 characters are allowed, however, we recommend that names be kept under 20 characters when possible to avoid exceeding the 255 character limit of DNS when including them in domain names.
The reason for the DNS adherence is to guarantee compatibility with DNS-SD and other naming schemas related to Arrowhead (https://ieeexplore.ieee.org/document/8972250).
Concretely, when a subject CN in this chapter is required to be a _valid DNS label_, it is to be understood that it matches the following regular expression: `^(?![0-9]+$)(?!.*-$)(?!-)[a-zA-Z0-9-]{1,62}$`.
This means that full domain names are _not_ allowed as subject CN, such as `my-device.company.arrowhead.eu`.
In the case of that example, it should rather have been `my-device`.
Full domain names are specified as subject alternative names where relevant.

#### 2.2.3 `extensions`

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the device owning the certificate. |

### 2.3 Transfer Certificates

## 3. Arrowhead X.509 Profiles


### 3.5 On-Boarding

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `onboarding` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

#### Extensions

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the device owning the certificate. |

### 3.6 Device

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `device` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

#### Extensions

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the device owning the certificate. |

### 3.7 System

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| Organization (O)  | `2.5.4.6`  | No       | Human-readable name of owning organization. |
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `system` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

#### Extensions

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the device owning the certificate. |

### 3.8 Operator

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| Attribute Type    | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `operator` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | Human-readable name of operator. |

#### Extensions

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the devices from which the operator administers its local cloud. |

### 3.9 Manufacturer

#### Issuer

Any or none at all.

### 3.10 Transfer

#### Issuer

A _Manufacturer_ certificate.

## 3. Algorithms, Key Lengths and Other Security Details

- Use fingerprints to identify certificate owners, not subject names.

## 4. Certificate Creation and Distribution

## 5. Authentication and Authorization

__Precedence__.
Being _hierarchical_, a DN describes a path from some arbitrary root to the subject or issuer in question, where the root is the leftmost RDN and the subject or issuer is the rightmost.
The root does not have to be associated with the root CA of the certificate in question.
While the above listed attributes _should not_ appear more than once in a subject or issuer DN, if they do, the rightmost takes precedence during the authorization procedure described in Section 6.

## 6. DNS Support and its Security Implications

## 7. Known Limitations

### 7.1 Subject Alternative Names and Device Mobility

Devices and systems are assumed not to change IP addresses or DNS names during their lifetimes, as these are recorded in their certificates.
This makes it a bit more challenging when devices need to move between networks and, as a consequence, may be assigned new IP addresses.
However, as mobility at a scale that makes this setup a problem is currently deemed of marginal utility, no provisions are given for it directly.
It could be working around by using proxies, negotiating a new certificate every network handover, recording all relevant IP addresses in advance in each certificate, and so on.

## References

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 1` |
| KeyUsage         | `2.5.29.15` | Yes      | `keyCertSign(5), cRLSign(6)` |