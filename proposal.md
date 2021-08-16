# Eclipse Arrowhead X.509 Certificate Profiles

## Abstract

This document specifies how X.509 certificates must be configured and issued to facilitate secure communication and identification within Eclipse Arrowhead local clouds.
It describes the certificates of Certificate Authorities (CAs) and end entities, relevant certificate security details, recommended procedures for the creation and distribution of certificates, as well as how DNS overlaps with X.509 in the context of Arrowhead.

## 1. Introduction

TODO

### 1.1 Relation to the IETF RFC 5280 X.509 Profile

All certificate profiles in this document, with the exception of the _Manufacturer_ and _Transfer_ profiles of Sections 3.9 and 3.10, are meant to be strict subsets of the RFC 5280 X.509 profile, which regulates the use of X.509 certificates on the World Wide Web.
Most significantly, this makes our profiles compatible with many implementations the TLS and DTLS standards for secure communication, even though complying to this document will require additional validation steps, such as ensuring that certain extensions are being or not being used.
Those steps are outlined in Section 6.

Building on top of RFC 5280 and X.509 means that also X.501 and ASN.1 become relevant to this document, as those standards define the subject/issuer naming schema and syntax notation used by X.509.
In particular, the interface description language of ASN.1 is used here to describe relevant certificate constituents.
Please refer to ASN.1 or a relevant learning resource if wanting to know more about its syntax.

### 1.2 Significant Terminology

The following subsections represent technical domains with particular bearing on this document.
Each subsection briefly describes the domain and lists terms relevant to our purposes.

#### 1.2.1 Eclipse Arrowhead

Service-oriented architecture for Industry 4.0 automation.

- __Device__: A physical machine that could be capable of hosting Arrowhead _systems_.
- __Local Cloud__: A physical or logical protected network consisting of communicating _systems_.
- __Service__: An explicitly defined network application interface accessible to authorized _systems_.
- __System__: A software application providing Arrowhead-compliant _services_ that runs on a _device_.

#### 1.2.2 X.509

Certificate standard for establishing trust between devices over untrusted computer networks.

- __Certificate Authority (CA)__: Entity issuing (signing) other certificates to endorse their validity.
- __End Entity__: Entity having but not issuing certificates.
- __Entity__: Any thing or being potentially able to hold and use an X.509 certificate.
- __Intermediary CA__: CA that _did not_ issue its own certificate and, therefore, can be trusted by explicitly trusting another certificate further up its issuance hierarchy.
- __Issuer__: The CA that signed a given certificate.
- __Public Key Infrastructure (PKI)__: The structure of entities, each having a certain role, required to facilitate secure certificate distribution.
- __Root CA__: CA that _did_ issue (self-sign) its own certificate and, therefore, must be explicitly trusted.
- __Subject__: The entity owning and being described by a given certificate.
- __Trust Anchor__: Another name for _Root CA_.

#### 1.2.3 X.501

Naming schema for X.500 directories.
The standard is used to name X.509 certificates.

- __Distinguished Name (DN)__: A hierarchical naming format composed consisting of RDNs. An example of a DN could be `O=My Company,CN=Robert Robertson+E=robert@mail.com`. The `O` RDN is at the highest hierarchical level, while the `CN+E` RDN is at the level below it. The comma character `,` is used to delimit the RDNs.
- __Relative Distinguished Name (RDN)__: A list of attribute/value pairs belonging to the same hierarchical level in a DN. Examples of RDNs could be `O=My Company` and `CN=Robert Robertson+E=robert@mail.com`. The first RDN consists of a single pair while the second consists of two delimited by `+`.

#### 1.2.4 ASN.1

Interface description language for describing messages that can be sent or received over computer networks using several associated encodings.
The standard is used to describe the structure of X.509 certificates, which must be encoded using ASN.1 DER, as defined below.

- __Basic Encoding Rules (BER)__: Binary ASN.1 encoding that appends basic type and length information to each encoded value, which means that decoding a given message does not require knowledge of its original ASN.1 description. Defined in X.690.
- __Distinguished Encoding Rules (DER)__: A subset of BER that guarantees canonical representation, which is to say that every pair of structurally equivalent ASN.1 messages can be represented in DER in exactly one way. Must be used when encoding X.509 certificates. Defined in X.690.
- __Object Identifier (OID)__: A structured universally unique identifier, useful for identifying parts of ASN.1 messages.

### 1.3 Conventions

The words __must__, __must not__, __required__, __shall__, __shall not__, __should__, __should not__, __recommended__, __may__, and __optional__ in this document are to be interpreted as described in IETF RFC 2119.
In short, this means that __must__, __required__ and __shall__ denote absolute requirements; __must not__ and __shall not__ denote absolute prohibitions; __should__, __should not__ and __recommended__ denote recommendations; and, finally, __may__ and __optional__ denote the subject of concern being truly optional.

## 2. Certificate Profiles

TODO

## 2.1 Basic Certificate Fields

The X.509 version 3 certificate has the below ASN.1 syntax, as defined in RFC 5280.
Note that RFC 5280 demands that the certificate be encoded using DER.

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

In this section, we will go through each of these fields and clarify what they signify and how they must or may be set in conforming Arrowhead X.509 certificates.
The section is by no means an attempt to rigorously define these fields.
Please consult RFC 5280 and X.509 for more exact details.

### 2.1.1 `Certificate`

A certificate and its signature.

#### 2.1.1.1 `tbsCertificate`

The part of the certificate whose integrity is guaranteed by the `signatureAlgorithm` and `signatureValue` fields.

#### 2.1.1.2 `signatureAlgorithm`

The cryptographic signature algorithm used to produce `signatureValue` and any parameters made relevant by the algorithm type.
Examples of algorithms could be _RSA with SHA256_ or _ED25519_.

This field must contain the same algorithm as the `signature` field in the `tbsCertificate`, as described in Section 2.1.2.3.

#### 2.1.1.3 `signatureValue`

A string of arbitrary bytes whose length is determined by the output of the `signatureAlgorithm`.
The value is used together with `signatureAlgorithm` and the public key of the issuer of the certificate to validate its integrity.

### 2.1.2 `TBSCertificate`

A certificate covered by, but not including, a signature.

#### 2.1.2.1 `version`

X.509 version of the certificate.
Must always be `v3(2)`.

RFC 5280 recommends that `v1(0)` and `v2(1)` certificates be used when the additional fields of later versions are unused.
It does only require that `v3(2)` be supported, however.
As we rely on certificate extensions for all of our profiles, we always require `v3(2)`.

#### 2.1.2.2 `serialNumber`

A positive integer of no more than 20 octets uniquely assigned by the issuer of the certificate.

__Recommendation__.
The serial number should be exactly 20 octets long and be produced by a cryptographic random number generator.
This serves both to (A) make the certificate more resistant to collision attacks and (B) to prevent the serial number from leaking important details about the certificate issuer, such as how many certificates it has issued, how long the signing process took, and so on.
Note that the number is signed and must be positive, which means that the most significant bit of the first DER `INTEGER` octet must be 0.

#### 2.1.2.3 `signature`

This field must contain the same algorithm as the `signatureAlorithm` field in the `Certificate`, as described in Section 2.1.1.2.

#### 2.1.2.4 `issuer`

The DN of the issuer of the certificate in question.
More specifically, this field contains an exact copy of the `subject` field of Section 2.1.2.6 from the certificate of its issuer.

#### 2.1.2.5 `validity`

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

#### 2.1.2.6 `subject`

The DN used to describe the owner of the certificate.
The field is defined as follows:

```asn1
Name ::= CHOICE {
    rdnSequence      RDNSequence
}

RDNSequence ::= SEQUENCE OF RelativeDistinguishedName

RelativeDistinguishedName ::= SET SIZE (1..MAX) OF AttributeTypeAndValue

AttributeTypeAndValue ::= SEQUENCE {
    type             AttributeType,
    value            AttributeValue
}

AttributeType ::= OBJECT IDENTIFIER

-- Concrete type determined by associated `AttributeType`.
-- In our case that type will always be `DirectoryString`.
AttributeValue ::= ANY

DirectoryString ::= CHOICE {
     teletexString    TeletexString   (SIZE (1..MAX)),
     printableString  PrintableString (SIZE (1..MAX)), -- ASCII subset. Preferred.
     universalString  UniversalString (SIZE (1..MAX)),
     utf8String       UTF8String      (SIZE (1..MAX)), -- Use when `PrintableString` is insufficient.
     bmpString        BMPString       (SIZE (1..MAX)) 
}
```

The below `AttributeType`s must be supported, in the sense that their OIDs are known to be associated with values of type `DirectoryString`, as defined above.

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

In contrast to RFC 5280, we _require_ always that each `AttributeValue` associated with any type of this list is concretely represented as a `PrintableString` or a `UTF8String`.
The former is preferred when its limited character set is sufficient.
See RFC 5280 Section 4.1.2.4 for more attributes that _should_ be supported.

__Precedence__.
Being _hierarchical_, a DN describes a path from some arbitrary root to the subject or issuer in question, where the root is the leftmost RDN and the subject or issuer is the rightmost.
The root does not have to be associated with the Root CA of the certificate in question.
While the above listed attributes _should not_ appear more than once in a subject or issuer DN, if they do, the rightmost takes precedence during the authorization procedure described in Section 6.

__Recommendation__.
Every certificate implementing a Section 3 profile _should_ only contain the attributes associated with that profile.
Each attribute _should_ be contained in its own RDN, and the RDNs _should_ be in the same order as they are presented in Section 3.
If a Section 3 attribute is declared as not required, it _may_ be omitted.

#### 2.1.2.7 `subjectPublicKeyInfo`

The public key of the subject, as well as the identity of the algorithm associated with it.
The field is defined as follows.

```asn1
SubjectPublicKeyInfo ::= SEQUENCE {
    algorithm         AlgorithmIdentifier,
    subjectPublicKey  BIT STRING
}
```

#### 2.1.2.8 `issuerUniqueID` and `subjectUniqueID`

Optional identifiers uniquely associated with the issuer or subject of the certificate.

These fields _must not_ be used.
When a certain certificate, or its subject, needs to be identified, the cryptographic hash, or _fingerprint_, of the certificate _should_ be used.
See Section 4 for more information about what hashing algorithms should be used for this and other purposes.

#### 2.1.2.9 `extensions`

```asn1
Extensions ::= SEQUENCE SIZE (1..MAX) OF Extension

Extension  ::= SEQUENCE  {
    extnID     OBJECT IDENTIFIER,
    critical   BOOLEAN DEFAULT FALSE,
    extnValue  OCTET STRING
}
```

### 2.1 CA Certificates

#### 2.1.1 Validity

TODO: Write something about sensible validity periods for certs.

#### 2.1.2 Subject

| RDN               | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| Organization (O)  | `2.5.4.6`  | No       | Human-readable name of owning organization. |
| DN Qualifier (DN) | `2.5.4.46` | Yes      | _Identifier_ specified for each profile. | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | Human-readable name of certificate. |

#### 2.1.3 Extensions

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: n*` |
| KeyUsage         | `2.5.29.15` | Yes      | `keyCertSign(5), cRLSign(6)` |

* each profile specifies its own `n`.

### 2.2 End Entity Certificates

#### 2.2.1 Validity

TODO: Write something about sensible validity periods for certs.

#### 2.2.2 Subject

| RDN               | OID        | Required | Value |
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

#### 2.2.3 Extensions

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the device owning the certificate. |

### 2.3 Transfer Certificates

## 3. Arrowhead X.509 Profiles

### 3.1 Master

#### 3.1.1 Issuer

Issued by any suitable RFC 5280 certificate, or none at all (self-signed).

#### 3.1.2 Validity

#### 3.1.3 Subject

| RDN               | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `master` | 

#### 2.1.3 Extensions

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 2` |

### 3.2 Gate



Issued by a _Master_ certificate.

#### Subject

| RDN               | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| Organization (O)  | `2.5.4.6`  | No       | Human-readable name of owning organization. |
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `gate` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

#### Extensions

| Extension               | OID         | Critical | Value |
|:------------------------|:------------|:---------|:------|
| KeyUsage                | `2.5.29.15` | Yes      | `digitalSignature(0), keyEncipherment(2)` |
| ExtendedKeyUsage        | `2.5.29.37` | No       | `serverAuth(1.3.6.1.5.5.7.3.1), clientAuth(1.3.6.1.5.5.7.3.2)` |
| SubjectAlternativeNames | `2.5.29.17` | Yes      | IPv4, IPv6 and DNS addresses/names associated with the device owning the certificate. |

### 3.3 Organization

#### Issuer

A _Master_ certificate.

#### Subject

| RDN               | OID        | Required | Value |
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

| RDN               | OID        | Required | Value |
|:------------------|:-----------|:---------|:------|
| DN Qualifier (DN) | `2.5.4.46` | Yes      | `localcloud` | 
| Common Name (CN)  | `2.5.4.3`  | Yes      | A valid DNS name label of 1 to 62 characters. |

#### Extensions

| Extension        | OID         | Critical | Value |
|:-----------------|:------------|:---------|:------|
| BasicConstraints | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 0` |
| KeyUsage         | `2.5.29.15` | Yes      | `keyCertSign(5), cRLSign(6)` |

### 3.5 On-Boarding

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| RDN               | OID        | Required | Value |
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

| RDN               | OID        | Required | Value |
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

| RDN               | OID        | Required | Value |
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

| RDN               | OID        | Required | Value |
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

## 4. Algorithms, Key Lengths and Other Security Details

- Use fingerprints to identify certificate owners, not subject names.

## 5. Certificate Creation and Distribution

## 6. Authentication and Authorization

## 7. DNS Support and its Security Implications

## 8. Known Limitations

### 8.1 Subject Alternative Names and Device Mobility

Devices and systems are assumed not to change IP addresses or DNS names during their lifetimes, as these are recorded in their certificates.
This makes it a bit more challenging when devices need to move between networks and, as a consequence, may be assigned new IP addresses.
However, as mobility at a scale that makes this setup a problem are currently deemed of marginal utility, no provisions are given for it directly.
It could be working around by using proxies, negotiating a new certificate every network handover, recording all relevant IP addresses in advance in each certificate, and so on.

## References
