# Eclipse Arrowhead X.509 Certificate Profiles

## Abstract

This document specifies how X.509 certificates must be configured and issued to facilitate secure communication and identification within Eclipse Arrowhead local clouds.
It describes the certificates of Certificate Authorities (CAs) and end entities, relevant certificate security details, recommended procedures for the creation and distribution of certificates, as well as how DNS overlaps with X.509 in the context of Arrowhead.

## 1. Introduction

### 1.1 The X.509 Standard

### 1.2 Significant Terminology

#### 1.2.1 Terms Related to Eclipse Arrowhead

| Term         | Definition |
|:-------------|:-----------|
| Arrowhead    | Service-oriented architecture for Industry 4.0 automation. |
| Device       | A physical machine that could be capable of hosting Arrowhead _systems_. |
| Local Cloud  | A physical or logical protected network consisting of communicating _systems_.  |
| Service      | An explicitly defined network application interface accessible to authorized _systems_. |
| System       | A software application providing Arrowhead-compliant _services_ that runs on a _device_. |

#### 1.2.2 Terms Related to X.509

| Term                         | Abbreviation | Definition |
|:-----------------------------|:-------------|:-----------|
| Certificate Authority        | CA           | Entity issuing (signing) other certificates to endorse their validity. |
| Distinguished Encoding Rules | DER          | An encoding format defined for ASN.1. |
| Distinguished Name           | DN           | A hierarchical naming format defined by X.500. A DN contains RDNs. |
| End Entity                   |              | Entity having but not issuing certificates. |
| Entity                       |              | Any thing or being potentially able to hold and use an X.509 certificate. |
| Intermediary CA              |              | CA that _did not_ issue its own certificate and, therefore, can be trusted by explicitly trusting another certificate further up its issuance hierarchy. |
| Relative Distinguished Name  | RDN          | A list of attribute/value pairs belonging to the same hierarchical level in a DN. |
| Root CA                      |              | CA that _did_ issue (self-sign) its own certificate and, therefore, must be explicitly trusted. |

### 1.3 Relation to the IETF RFC 5280 X.509 Profile

### 1.4 Conventions

## 2. Certificate Profiles

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

The cryptographic signature algorithm used to produce `signatureValue` of `Certificate` and any parameters made relevant by the algorithm type.
Examples of algorithms could be _RSA with SHA256_ or _ED25519_.

This field must contain the same algorithm as the `signatureAlorithm` field in the `Certificate`, as described in Section 2.1.1.2.

#### 2.1.2.4 `issuer`

The DN of the issuer of the certificate in question.
More specifically, this field contains a copy of the `subject` field of Section 2.1.2.6 from the issuer's certificate.

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

#### 2.1.2.7 `subjectPublicKeyInfo`

```asn1
SubjectPublicKeyInfo ::= SEQUENCE {
    algorithm         AlgorithmIdentifier,
    subjectPublicKey  BIT STRING
}
```

#### 2.1.2.8 `issuerUniqueID`

An optional identifier uniquely associated with the issuer of the certificate. This field must not be used.

#### 2.1.2.9 `subjectUniqueID`

An optional identifier uniquely associated with the subject of the certificate. This field must not be used.

#### 2.1.2.10 `extensions`

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

## 5. Certificate Creation and Distribution

## 6. DNS Support and Security Implications

## 7. Known Limitations

### 7.1 Subject Alternative Names and Device Mobility

Devices and systems are assumed not to change IP addresses or DNS names during their lifetimes, as these are recorded in their certificates.
This makes it a bit more challenging when devices need to move between networks and, as a consequence, may be assigned new IP addresses.
However, as mobility at a scale that makes this setup a problem are currently deemed of marginal utility, no provisions are given for it directly.
It could be working around by using proxies, negotiating a new certificate every network handover, recording all relevant IP addresses in advance in each certificate, and so on.

## References
