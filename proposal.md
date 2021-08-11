# Eclipse Arrowhead X.509 Certificate Profiles

## Abstract

This document specifies how X.509 certificates must be configured and issued to facilitate secure communication and identification within Eclipse Arrowhead local clouds.
It describes the certificates of Certificate Authorities (CAs) and end entities, relevant certificate security details, recommended procedures for the creation and distribution of certificates, as well as how DNS overlaps with X.509 in the context of Arrowhead.

## 1. Introduction

### 1.1 The X.509 Standard

### 1.2 Relation to the IETF RFC 5280 X.509 Profile

### 1.3 Conventions

## 2. Arrowhead X.509 Profiles

### 2.1 Master

#### Issuer

Any suitable RFC 5280 certificate, or none at all.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `master` | 
| Common Name (CN) | 2.5.4.3 | Yes | Human-readable name of certificate. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: true, pathLenConstraint: 2 |
| KeyUsage         | Yes | keyCertSign, cRLSign |

### 2.2 Organization

#### Issuer

A _Master_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `organization` | 
| Common Name (CN) | 2.5.4.3 | Yes | Human-readable name of certificate. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: true, pathLenConstraint: 1 |
| KeyUsage         | Yes | keyCertSign, cRLSign |

### 2.3 Gatekeeper

#### Issuer

A _Master_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `system` | 
| Common Name (CN) | 2.5.4.3 | Yes | Machine-readable name of gatekeeper system matching `[a-z](-[0-9a-z]+)*[0-9a-z]?`. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: false |
| KeyUsage         | Yes | digitalSignature, keyEncipherment |
| ExtendedKeyUsage | Yes | serverAuth, clientAuth |

### 2.4 Gateway

#### Issuer

A _Master_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `system` | 
| Common Name (CN) | 2.5.4.3 | Yes | Machine-readable name of gateway system matching `[a-z](-[0-9a-z]+)*[0-9a-z]?`. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: false |
| KeyUsage         | Yes | digitalSignature, keyEncipherment |
| ExtendedKeyUsage | Yes | serverAuth, clientAuth |

### 2.5 Local Cloud

#### Issuer

An _Organization_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `localcloud` | 
| Common Name (CN) | 2.5.4.3 | Yes | Human-readable name of certificate. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: true, pathLenConstraint: 0 |
| KeyUsage         | Yes | keyCertSign, cRLSign |

### 2.6 On-Boarding

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `onboarding` | 
| Common Name (CN) | 2.5.4.3 | Yes | Machine-readable name of on-boarded device matching `[a-z](-[0-9a-z]+)*[0-9a-z]?`. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: false |
| KeyUsage         | Yes | digitalSignature, keyEncipherment |
| ExtendedKeyUsage | Yes | serverAuth, clientAuth |

### 2.7 Device

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `device` | 
| Common Name (CN) | 2.5.4.3 | Yes | Machine-readable name of device matching `[a-z](-[0-9a-z]+)*[0-9a-z]?`. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: false |
| KeyUsage         | Yes | digitalSignature, keyEncipherment |
| ExtendedKeyUsage | Yes | serverAuth, clientAuth |

### 2.8 System

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `system` | 
| Common Name (CN) | 2.5.4.3 | Yes | Machine-readable name of system matching `[a-z](-[0-9a-z]+)*[0-9a-z]?`. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: false |
| KeyUsage         | Yes | digitalSignature, keyEncipherment |
| ExtendedKeyUsage | Yes | serverAuth, clientAuth |

### 2.9 Operator

#### Issuer

A _Local Cloud_ certificate.

#### Subject

| RDN | OID | Required | Value |
|:----|:----|:---------|:------|
| Organization (O) | 2.5.4.6 | Yes | Human-readable name of owning organization. |
| DN Qualifier (DNQ) | 2.5.4.46 | Yes | `operator` | 
| Common Name (CN) | 2.5.4.3 | Yes | Name of operator. |

#### Extensions

| Extension | Critical | Value |
|:----------|:---------|:------|
| BasicConstraints | Yes | cA: false |
| KeyUsage         | Yes | digitalSignature, keyEncipherment |
| ExtendedKeyUsage | Yes | serverAuth, clientAuth |

### 2.10 Manufacturer

#### Issuer

Any or none at all.

### 2.11 Transfer

#### Issuer

A _Manufacturer_ certificate.

## 3. Algorithms, Key Lengths and Other Security Details

## 4. Certificate Creation and Distribution

## 5. DNS Support and Security Implications

## References
