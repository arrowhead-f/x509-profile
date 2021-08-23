# Eclipse Arrowhead X.509 Certificate Profiles

## Abstract

This document specifies how X.509 certificates are to be configured, issued and validated to facilitate secure identification and communication within and between Eclipse Arrowhead local clouds, when such security is desired and the kind of certificate is relevant.

## 1. Introduction

TODO

### 1.1 Relation to the IETF RFC 5280 X.509 Profile

All certificate profiles specified in this document are _required_ to be strict subsets of the RFC 5280 X.509 profile, which regulates the use of X.509 certificates on the World Wide Web.

### 1.2 Significant Terminology

The following subsections represent technical domains with particular bearing on this document.
Each subsection briefly describes a domain and defines relevant terms and abbreviations.

#### 1.2.1 Eclipse Arrowhead

Service-oriented communication architecture for Industry 4.0 automation.

- __Device__: A physical machine that could be capable of hosting Arrowhead _systems_.
- __Local Cloud__: A physical protected network consisting of communicating _systems_.
- __Service__: An explicitly defined network application interface accessible to authorized _systems_.
- __System__: A software application providing Arrowhead-compliant _services_ that runs on a _device_.

#### 1.2.2 X.509

Certificate standard for establishing trust between devices over untrusted computer networks.

- __Certificate Authority (CA)__: Entity issuing (signing) other certificates to endorse their validity.
- __Certificate Chain__: A chain consisting of an _end entity_ certificate, its _issuer_'s certificate, that _issuer_'s _issuer_'s certificate, and so on up to the _root CA_'s certificate.
- __Certificate Revocation List (CRL)__: A list identifying certificates that no longer are to be considered valid even though they are yet to expire.
- __Certificate Signing Request (CSR)__: A request for a certificate to be issued by a receiving CA.
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

- __Distinguished Name (DN)__: A hierarchical naming format composed consisting of RDNs. An example of a DN could be `O=My Company,CN=Robert Robertson+E=robert@mail.com`. The `O` RDN is at the highest hierarchical level, while the `CN+E` RDN is at the level below it. The comma character `,` is used to delimit the RDNs in this example.
- __Relative Distinguished Name (RDN)__: A list of attribute/value pairs belonging to the same hierarchical level in a DN. Examples of RDNs could be `O=My Company` and `CN=Robert Robertson+E=robert@mail.com`. The first RDN consists of a single pair while the second consists of two delimited by `+`.

#### 1.2.4 ASN.1

Interface description language for describing messages that can be sent or received over computer networks using several associated encodings.
The standard is used to describe the structure of X.509 certificates, which _must_ be encoded using ASN.1 DER, as defined below.

- __Basic Encoding Rules (BER)__: Binary ASN.1 encoding that appends basic type and length information to each encoded value, which means that decoding a given message does not require knowledge of its original ASN.1 description. Defined in X.690.
- __Distinguished Encoding Rules (DER)__: A subset of BER that guarantees canonical representation, which is to say that every pair of structurally equivalent ASN.1 messages can be represented in DER in exactly one way. Must be used when encoding X.509 certificates. Defined in X.690.
- __Object Identifier (OID)__: A hierarchical and universally unique identifier, useful for identifying parts of ASN.1 messages.
- __Octet__: An 8-bit byte.

### 1.3 Conventions

The words __must__, __must not__, __required__, __should__, __should not__, __recommended__, __may__, and __optional__ in this document are to be interpreted as follows: __must__ and __required__ denote absolute requirements that must be adhered to for a certificate to be considered compliant to a given profile; __must not__ denotes an absolute prohibition; __should__, __should not__ and __recommended__ denote recommendations that should be deviated from only if special circumstances make it relevant; and, finally, __may__ and __optional__ denote something being truly optional.

## 2. Certificate Profiles

There are eight certificate profiles defined in this document, depicted in the following diagram:

```
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
                          +-------+-------+
                          |  Local Cloud  |
                          +---------------+
                                  A
                                  |
       +-----------------+--------+--------+-----------------+
       |                 |                 |                 | 
+------+------+   +------+------+   +------+------+   +------+------+
| On-Boarding |   |   Device    |   |   System    |   |  Operator   |
+-------------+   +-------------+   +-------------+   +-------------+
```
Each diagram box represents a profile.
The arrows in the diagram are to be read as "issued by", meaning that the every certificate adhering to the profile from which the arrow extends must be issued (signed) by a certificate with the profile pointed to.

In this section, we begin by considering the X.509 format itself, after which we outline the Master, Gate, Organization, Local Cloud, On-Boarding, Device, System and Operator profiles.
Each profile is described in terms of its (a) purpose, (b) issuer, (c) subject name and (d) extensions.
The details included in this section are intended to be enough to allow for the correct parameterization of certificates, but are unlikely to be sufficient for implementing software for handling them.
If the latter is relevant, please refer to RFC 5280, X.509, X.501, X.690 and ASN.1.

## 2.1 Certificate Format

The ASN.1 syntax of the third version of the X.509 certificate format is defined in as follows in RFC 5280:

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
Only `v3(2)` supports certificate extensions, which _must_ be used by all profiles described in this document.
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
More specifically, this field contains an exact copy of the `subject` field of Section 2.1.6 from the certificate of its issuer, or a copy of the `issuer` field of the same certificate if it is self-signed.

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
See Section 4.1.2.5 of RFC 5280 for more details.

The time span denoted by this field _should_ be shorter than the expected lifetime of the entity owning it, if provisions exist for renewing it during its lifetime.
More specific recommendations will be published in other documents of the Eclipse Arrowhead project.

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

Every certificate _must_ contain exactly one `subject` _Distinguished Name Qualifier_ (`DN`) that identifies the profile it adheres to.
The actual identifiers are stated in Sections 2.2 to 2.10.

In addition, each certificates _must_ contain exactly one `subject` _Common Name_ (`CN`) that is a valid DNS label (https://datatracker.ietf.org/doc/html/rfc1035#section-2.3.1) of no more than 62 characters.
Note that DNS labels _must not_ contain dots, or any other character than the alphanumeric and dash.
While names _may_ use 62 characters, we _recommend_ that names be kept at 20 characters or less.
The `CN` field value _must_, furthermore, be unique among all certificates issued by the same CA with same Distinguished Name Qualifier (`DN`).

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
If desired to identify the subject by its public key, which _may_ be used in multiple certificates, the cryptographic hash of the `subjectPublicKey` field (excluding the tag, length, and number of unused bits) of `subjectPublicKeyInfo` _should_ be used.
See Section 3 for more information about what hashing algorithms should be used for this and other purposes.

#### 2.1.9 `extensions`

A list of certificate extensions, as defined below.

```asn1
Extensions ::= SEQUENCE SIZE (1..MAX) OF Extension

Extension  ::= SEQUENCE {
    extnID     OBJECT IDENTIFIER,
    critical   BOOLEAN DEFAULT FALSE,
    extnValue  OCTET STRING
}
```

RFC 5280 explicitly outlines 17 different X.509 extensions, listed by category below.

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

The __Authority Key Identifier__ and __Subject Key Identifier__ extensions are used to identify the public key of the issuer and subject of a given certificate, respectively.
The extensions are defined as follows:

```asn1
-- Required for all but self-signed CA certificates (root CAs).
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
Use of the `authorityCertIssuer` and `authorityCertSerialNumber` fields of `AuthorityKeyIdentifier` is _not recommended_.
If a self-signed certificate leaves the `authorityCertIssuer` and `authorityCertSerialNumber` fields unspecified, it may omit the `AuthorityKeyIdentifier` extension entirely.
RFC 5280 further _requires_ that all but end entity certificates use the `SubjectKeyIdentifier` extension.
Its value _should_ be the the cryptographic hash of the `subjectPublicKey` value (excluding the tag, length, and number of unused bits) of the `subjectPublicKeyInfo` field.
The extensions _must not_ be marked as critical.

The __Key Usage__ and __Extended Key Usage__ extensions are defined as follows:

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
The extension _must_ be used in all certificates and _must_ be marked as critical.
How to set it is outlined in the section specific to each particular certificate profile.

The `ExtKeyUsageSyntax` extension has the same purpose, but is open-ended in the sense that it allows for any OID to be used as an indication of what a certain certificate _may_ be used for.
RFC 5280 lists six such purpose identifiers, out of which two has particular bearing on this document, namely the `serverAuth` (OID `1.3.6.1.5.5.7.3.1`) and `clientAuth` (OID `1.3.6.1.5.5.7.3.2`) identifiers, which indicate that the certificate holder must be able to respond to TLS server and client authorization requests, respectively.
The extension _must_ be used in all end entity certificates, as well in the certificates of the CAs that handle CSRs directly via network application interfaces.
The `serverAuth` and `clientAuth` identifiers _must_ be included.
The extension _should not_ be marked as _critical_.
Other purpose identifiers _may_ be used.

#### 2.1.9.2 Policy Extensions

In the context of X.509 and RFC 5280, a _policy_ is a legal document that binds the owner of a certificate and, potentially, all certificates issued by that certificate to certain legal obligations.
The four policy extensions defined by RFC 5280 _may_ be used to ensure that every certificate in a given certificate chain have accepted some policies of concern.
Please refer to RFC 5280 for more details about these extensions.
Their use is _optional_.

#### 2.1.9.3 Name Extensions

The __Subject Alternative Name__ and __Issuer Alternative Name__ allows for additional identities to be associated with a given `subject` or `issuer` name.
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

The `SubjectAltName` extension _must_ be used by all end entity certificates and _must_ identify at least one DNS name or IP address.
The extension _must_ also be used for CA certificates that handles CSRs directly via network application interfaces.
Use of the `IssuerAltName` is _optional_.
Both extensions _should_ be marked as non-critical.

The __Name Constraints__ extension makes it possible for a CA to restrict the set of values allowed for the `subject` and `SubjectAltName` fields in issued certificates.
The extension _may_ be used.
It _must_ be marked as critical if used.
Please refer to RFC 5280 Section 4.2.1.10 for more details.

#### 2.1.9.4 CRL Extensions

The CRL extensions facilitate a mechanism for revoking certificates that are still valid and distributing information about those revocations.
These extensions _should not_ be used to facilitate the revocation of end entity certificates.
They _may_ be used for revoking CA certificates.

Eclipse Arrowhead comes with its own certificate revocation procedure via its _Certificate Authority_ system.
Its use is _recommended_ for revoking and verifying the validity of On-Boarding, Device, System and Operator certificates.

#### 2.1.9.5 Information Extensions

The information extensions allows various types of data sources or services to be associated with the certificate holder.
These extensions _should not_ be used.

Eclipse Arrowhead has its own provisions for metadata distribution and service management, via the _Service Registry_ system, _Gatekeeper_ system, and other.
Those provisions _should_ be used.

#### 2.1.9.6 Other Extensions

The __Subject Directory Attributes__ allows for arbitrary identification attributes, such as nationality, to be associated with the `subject` of a certificate.
It _may_ be used.
It _must_ be marked as non-critical if used.

The __Basic Constraints__ extension allows for it to be denoted whether or not a given certificate belongs to a CA, as well as how many intermediary CAs may exist below it in any given certificate chain.
The extension is defined as follows:

```asn1
BasicConstraints ::= SEQUENCE {
    cA                BOOLEAN DEFAULT FALSE,
    pathLenConstraint INTEGER (0..MAX) OPTIONAL
}
```

The extension _must_ be used and marked critical by all Arrowhead-compliant certificates.
The `pathLenConstraint` _must_ be set by all CA certificates, but _must not_ be set by end entity certificates.

### 2.2 Master Profile

A _Master_ certificate exists to establish trust between organizations that may want to interconnect their Arrowhead systems.
It does this by issuing _Organization_ and _Gate_ certificates.
The former enables organizations to set up their own certificate hierarchies while sharing a common CA with other organizations.
The latter kind enables all those organizations to trust a special kind of broker system, which facilitates negotiating connections between organizations.

The Eclipse Arrowhead project _should_ maintain an authoritative Master certificate that _may_ be used for non-private Arrowhead installations.
Other entities _may_ setup and maintain their own Master certificates as needed.

#### 2.2.1 `issuer`

_May_ be self-signed or issued by an RFC 5280-compliant CA.

__Security notice__.
In the case of a Master being issued by a widely trusted World Wide Web CA, entities with the Master in their certificate chains could be made able to serve HTTP(S) traffic to regular web browsers without any additional certificate distribution.
It could also make it possible for improperly configured systems to establish secure connections with unauthorized systems.
Having CAs above a Master provides for new opportunities and new risks at the same time, for which reason it may or may not be desirable.

#### 2.2.2 `subject`

The subject field DN _must_ contain the following attributes exactly once, either in the same or different RDNs:

| Attribute Type    | OID        | Value |
|:------------------|:-----------|:------|
| DN Qualifier (DN) | `2.5.4.46` | `master`.
| Common Name (CN)  | `2.5.4.3`  |  A valid DNS name label, such as `ltu-rsa-2021`.

#### 2.2.3 `extensions`

The following extensions _must_ be used and configured as described:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Authority Key Identifier | `2.5.29.35` | No       | Hash of issuer public key. Omit field if self-signed. See Section 2.1.9.1.
| Subject Key Identifier   | `2.5.29.14` | No       | Hash of subject public key. See Section 2.1.9.1.
| Basic Constraints        | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 2`. See Section 2.1.9.6.
| Key Usage                | `2.5.29.15` | Yes      | Bits `keyCertSign(5)` and `cRLSign(6)` _must_ be set. See Section 2.1.9.1.

If the certificate will be used to automatically respond to CSRs via a network application interface, the following must also be present:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Key Usage                | `2.5.29.15` | Yes      | Bits `digitalSignature(0)` and `keyEncipherment(2)` _must_ be set in addition to `5` and `6`. See Section 2.1.9.1.
| Extended Key Usage       | `2.5.29.37` | No       | Purposes`serverAuth` and `clientAuth` _must_ be specified. See Section 2.1.9.1.
| Subject Alternative Name | `2.5.29.17` | No       | At least one IP address or DNS name to which CSRs can be sent. See Section 2.1.9.3.

### 2.3 Gate Profile

A _Gate_ certificate is associated with a message broker or bus that exists to guarantee delivery of messages between the local clouds of distinct organizations.
Its existence means that such messages can sent over a secure transport.

#### 2.3.1 `issuer`

_Must_ be issued by a Master certificate.

#### 2.3.2 `subject`

The subject field DN _must_ contain the following attributes exactly once, either in the same or different RDNs:

| Attribute Type    | OID        | Value |
|:------------------|:-----------|:------|
| DN Qualifier (DN) | `2.5.4.46` | `gate`.
| Common Name (CN)  | `2.5.4.3`  | A valid DNS name label, such as `mqtt-04`. See Section 2.1.6.

#### 2.3.3 `extensions`

The following extensions _must_ be used and configured as described:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Authority Key Identifier | `2.5.29.35` | No       | Hash of issuer public key. See Section 2.1.9.1.
| Basic Constraints        | `2.5.29.19` | Yes      | `cA: false`. See Section 2.1.9.6.
| Key Usage                | `2.5.29.15` | Yes      | Bits `digitalSignature(0)` and `keyEncipherment(2)` _must_ be set. See Section 2.1.9.1.
| Extended Key Usage       | `2.5.29.37` | No       | Purposes `serverAuth` and `clientAuth` _must_ be specified. See Section 2.1.9.1.
| Subject Alternative Name | `2.5.29.17` | No       | At least one IP address or DNS name through which the system can be reached. See Section 2.1.9.3.

### 2.4 Organization Profile

An _Organization_ certificate is maintained by a single organization, allowing it to manage the certificates of its own local clouds.

#### 2.4.1 `issuer`

_Must_ be issued by a Master certificate.

#### 2.4.2 `subject`

The subject field DN _must_ contain the following attributes exactly once, either in the same or different RDNs:

| Attribute Type    | OID        | Value |
|:------------------|:-----------|:------|
| DN Qualifier (DN) | `2.5.4.46` | `organization`.
| Common Name (CN)  | `2.5.4.3`  |  A valid DNS name label, such as `company-rsa-2021`.

#### 2.4.3 `extensions`

The following extensions _must_ be used and configured as described:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Authority Key Identifier | `2.5.29.35` | No       | Hash of issuer public key. See Section 2.1.9.1.
| Subject Key Identifier   | `2.5.29.14` | No       | Hash of subject public key. See Section 2.1.9.1.
| Basic Constraints        | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 1`. See Section 2.1.9.6.
| Key Usage                | `2.5.29.15` | Yes      | Bits `keyCertSign(5)` and `cRLSign(6)` _must_ be set. See Section 2.1.9.1.

If the certificate will be used to automatically respond to CSRs via a network application interface, the following must also be present:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Key Usage                | `2.5.29.15` | Yes      | Bits `digitalSignature(0)` and `keyEncipherment(2)` _must_ be set in addition to `5` and `6`. See Section 2.1.9.1.
| Extended Key Usage       | `2.5.29.37` | No       | Purposes`serverAuth` and `clientAuth` _must_ be specified. See Section 2.1.9.1.
| Subject Alternative Name | `2.5.29.17` | No       | At least one IP address or DNS name to which CSRs can be sent. See Section 2.1.9.3.

### 2.5 Local Cloud Profile

An _Local Cloud_ certificate is maintained by a single local cloud, enabling it to issue end entity certificates for on-boarding and on-boarded devices, as well as for systems and operators.

#### 2.4.1 `issuer`

_Must_ be issued by an Organization certificate.

#### 2.4.2 `subject`

The subject field DN _must_ contain the following attributes exactly once, either in the same or different RDNs:

| Attribute Type    | OID        | Value |
|:------------------|:-----------|:------|
| DN Qualifier (DN) | `2.5.4.46` | `localcloud`.
| Common Name (CN)  | `2.5.4.3`  |  A valid DNS name label, such as `pumping-station-04`.

#### 2.4.3 `extensions`

The following extensions _must_ be used and configured as described:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Authority Key Identifier | `2.5.29.35` | No       | Hash of issuer public key. See Section 2.1.9.1.
| Subject Key Identifier   | `2.5.29.14` | No       | Hash of subject public key. See Section 2.1.9.1.
| Basic Constraints        | `2.5.29.19` | Yes      | `cA: true, pathLenConstraint: 0`. See Section 2.1.9.6.
| Key Usage                | `2.5.29.15` | Yes      | Bits `keyCertSign(5)` and `cRLSign(6)` _must_ be set. See Section 2.1.9.1.

If the certificate will be used to automatically respond to CSRs via a network application interface, the following must also be present:

| Extension                | OID         | Critical | Value |
|:-------------------------|:------------|:---------|:------|
| Key Usage                | `2.5.29.15` | Yes      | Bits `digitalSignature(0)` and `keyEncipherment(2)` _must_ be set in addition to `5` and `6`. See Section 2.1.9.1.
| Extended Key Usage       | `2.5.29.37` | No       | Purposes`serverAuth` and `clientAuth` _must_ be specified. See Section 2.1.9.1.
| Subject Alternative Name | `2.5.29.17` | No       | At least one IP address or DNS name to which CSRs can be sent. See Section 2.1.9.3.

### 2.6 On-Boarding Profile

### 2.7 Device Profile

### 2.8 System Profile

### 2.9 Operator Profile

## 3. Algorithms, Key Lengths and Other Security Details

Choosing a signature scheme for a certificate poorly can lead to severe security vulnerabilities.
We _recommend_ that relevant recommendations of credible information security institutes, such as NIST, ENSIA or IETF, be consulted for making choices about algorithms, key lengths and other relevant security details.

Given that no relevant breakthroughs are made or expected in quantum computing, you _may_ chose to follow RFC 7525, which recommends the following four TLS _cipher suites_:

| Key Exchange | Authentication     | Encryption  | Hash   |
|:-------------|:-------------------|:------------|:-------|
| DHE          | RSA (2048 or 3072) | AES 128 GCM | SHA256 |
| ECDHE        | RSA (2048 or 3072) | AES 128 GCM | SHA256 |
| DHE          | RSA (2048 or 3072) | AES 256 GCM | SHA384 |
| ECDHE        | RSA (2048 or 3072) | AES 256 GCM | SHA384 |

Note that all TLS versions prior to 1.2 are deprecated, as per RFC 8996.
A cipher suite is a collection of cryptographic algorithm used to establish secure connections over untrusted transports.
The _Authentication_ and _Hash_ algorithms of a cipher suite constitute what is referred to as a _signature scheme_, which must be specified in and used with every created X.509 certificate, as mentioned in Section 2.1.3.

The above recommendations are _general_, in the sense that no particular assumptions are made about the setting in which the device employing the signature or cipher suite is located.
We understand that many Arrowhead installations will involve hardware with limited computational capabilities, which may not be able to handle primitives of the cryptographic strengths we have mentioned.
The Eclipse Arrowhead project will publish summaries of recommendations for such and other settings in the future.

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
