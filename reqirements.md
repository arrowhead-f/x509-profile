# X.509 Profile Requirements

This file lists the current set of requirements on the Arrowhead X.509 profiles.

|   # | Description |
|----:|:------------|
|   1 | _Official Root_. An official Arrowhead root will be managed by the Eclipse Arrowhead project. Its use will not be mandatory, even though some root certificate must be used.
|   2 | _Roots can issue (A) organization, (B) gatekeeper and (C) gateway certificates_. Organization certificates enable organizations to setup their own local clouds, while gatekeeper and gateways help manage communication between organizations that trust the same root certificates.
|   3 | _Gatekeeper and Gateway certificates cannot issue further certificates_. They are user certificates.
|   4 | _Organization certificates can issue local cloud certificates_. An organization certificate must exist even if a given organization would only have a single local cloud.
|   5 | _Every local cloud certificate must be issued by an organization certificate_. Local clouds do not compose (i.e. local clouds cannot contain other local clouds), which should be reflected in the certificate structure.
|   6 | _Local cloud certificates can issue (A) on-boarding, (B) device, (C) system and (D) operator certificates_. On-boarding certificates are used to request device certificates, which can be used to request system certificates. System certificates enable a given device to securely host systems and provide services. Operator certificates are associated with human or non-human operators, which administer local clouds by accessing their services.
|   7 | _On-boarding, Device, System and Operator certificates cannot issue further certificates_. They are user certificates.
|   8 | _The type of each certificate is directly identifiable_. Some field of each certificate explicitly denotes whether it is a root, organization, gatekeeper, gateway, local cloud, on-boarding, device, system or operator certificate. This is key for access control operations needing to take the type of accessing entity into account.
|   9 | _The identity of the certificate owner is described in each certificate, as relevant to the certificate type_. Owning organization name, department, person name and other similar details must be recorded in every certificate, as briefly as possible. This is key both for access control and accountability purposes.
|  10 | _Every on-boarding, device and system certificate must state the network interfaces through which they can be reached_. IP addresses, DNS names, or whatever else is relevant must be stated in the certificate. As operators never provide services, they must not have network information associated with them. This information must be provided via the `SubjectAlternativeNames` extension.
|  11 | _Every certificate able to provide services must be IETF RFC 5280 compliant_. This enables using a single certificate to establish connections both between Arrowhead and non-Arrowhead systems, given that the latter kind is RFC 5280 compliant. RFC 5280 compliance also means compatibility with DNSSEC and other relevant DNS protocols.
|  12 | _The extensions required to enforce distinction between CAs and users, as well as to limit the length of issuance hierarchies, must be used_. This means that the `BasicConstraints` extension must be used, as per RFC 5280.
|  13 | _A manufacturer certificate may be self-signed or issued by an arbitrary authority._ A manufacturer can issue transfer certificates, which can be used by devices produced by the manufacturer to acquire on-boarding certificates in local clouds where it, or its issuer, is trusted. No other requirements exist on the certificate, other than that it has acceptable security parameters, is not expired, and so on.
|  14 | _Transfer certificates cannot issue further certificates_. No other requirements exist on the certificate, other than that it has acceptable security parameters, is not expired, and so on.

As the requirements currently stand, the following hierarchy of profiles exist:

```
                          +---------------+
                          |     Root      |
                          +---------------+
                                  A
                                  |
                                  +--------------------+--------------------+
                                  |                    |                    |
     +---------------+    +-------+-------+    +-------+-------+    +-------+-------+
     | Manufacturer  |    | Organization  |    |  Gatekeeper   |    |    Gateway    |
     +-------+-------+    +---------------+    +---------------+    +-------+-------+
             A                    A
             |                    |
     +-------+-------+    +-------+-------+
     |   Transfer    |    |  Local Cloud  |
     +---------------+    +---------------+
                                  A
                                  |
       +-----------------+--------+--------+-----------------+
       |                 |                 |                 | 
+------+------+   +------+------+   +------+------+   +------+------+
| On-Boarding |   |   Device    |   |   System    |   |  Operator   |
+-------------+   +-------------+   +-------------+   +-------------+
```
