# Eclipse Arrowhead X.509 Certificate Profiles

## Abstract

This document specifies how X.509 certificates must be configured and issued to facilitate secure communication and identification within Eclipse Arrowhead local clouds.
It describes the certificates of Certificate Authorities (CAs) and end entities, relevant certificate security details, recommended procedures for the creation and distribution of certificates, as well as how DNS overlaps with X.509 in the context of Arrowhead.

## 1. Introduction

### 1.1 The X.509 Standard

### 1.2 Relation to the IETF RFC 5280 X.509 Profile

### 1.3 Significant Terminology

| Term | Definition |
|:-----|:-----------|
| Arrowhead | |
| CA   | Certificate Authority, which may issue (sign) other certificates to endorse their validity. |
| Intermediary | |
| End Entity   | |
| Local Cloud  | |

### 1.4 Conventions

## 2. Arrowhead X.509 Profile Categories

```asn1
TBSCertificate  ::=  SEQUENCE  {
    version         [0]  EXPLICIT Version DEFAULT v1,        -- Must be v3(2)
    serialNumber         CertificateSerialNumber,            -- Should (must?) be cryto rand number
    signature            AlgorithmIdentifier,                -- Must be trusted crypto
    issuer               Name,                               -- Taken from issuer cert
    validity             Validity,
    subject              Name,
    subjectPublicKeyInfo SubjectPublicKeyInfo,               -- Must be trusted crypto
    issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL, -- Must NOT be used
    subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL, -- Must NOT be used
    extensions      [3]  EXPLICIT Extensions OPTIONAL
}
```

Prefer `PrintableString` in `Name`s; `UTF8String` also OK.

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
