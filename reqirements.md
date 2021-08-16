# X.509 Profile Requirements

This file lists the current set of requirements on the Arrowhead X.509 profiles.

|   # | Description |
|----:|:------------|
|   1 | _Official Master_. An official Arrowhead master certificate will be managed by the Eclipse Arrowhead project. Its use will not be mandatory, even though some master certificate must be used.
|   2 | _Masters can issue (A) organization and (B) gate certificates_. Organization certificates enable organizations to setup their own local clouds, while gates (MQTT brokers) help manage communication between organizations that trust the same root certificates.
|   3 | _Gate certificates cannot issue further certificates_. They are end entity certificates.
|   4 | _Organization certificates can issue local cloud certificates_. An organization certificate must exist even if a given organization would only have a single local cloud.
|   5 | _Every local cloud certificate must be issued by an organization certificate_. Local clouds do not compose (i.e. local clouds cannot contain other local clouds), which should be reflected in the certificate structure.
|   6 | _Local cloud certificates can issue (A) on-boarding, (B) device, (C) system and (D) operator certificates_. On-boarding certificates are used to request device certificates, which can be used to request system certificates. System certificates enable a given device to securely host systems and provide services. Operator certificates are associated with human or non-human operators, which administer local clouds by accessing their services.
|   7 | _On-boarding, Device, System and Operator certificates cannot issue further certificates_. They are end entity certificates.
|   8 | _The type of each certificate is directly identifiable_. Some field of each certificate explicitly denotes whether it is a root, organization, gate, local cloud, on-boarding, device, system or operator certificate. This is key for access control operations needing to take the type of accessing entity into account. Manufacturer and transfer certificates are exempt from this rule.
|   9 | _The identity of the certificate owner should be described in each CA and operator certificate_. Some marker, such as the name of the organization or person owning a particular CA certificate should be named in it, as briefly as possible. This is key primarily for accountability purposes.
|  10 | _An identifier compatible with DNS-SD is stored in each (A) local cloud, (B) on-boarding, (C) device and (D) system certificate_. Each such identifier is a valid DNS label of no more than 62 characters, to make room for an initial underscore (_) if used with the DNS-SD schema specified in https://ieeexplore.ieee.org/document/8972250. Each local cloud identifier must be unique within its organization, and all other identifiers must be unique within their local clouds. The identifier is stored in the certificate to ensure secure service, system and device lookup by name. Note that a DNS __label__ is one dot-delimited subsection of a full DNS __name__. Full names have previously been used in Arrowhead certificates.
|  11 | _Every on-boarding, device, system, operator and gate certificate must state the network interfaces through which they can be reached or send messages from_. IP addresses, DNS names, or whatever else is relevant must be stated in the certificate. This information must be provided via the `SubjectAlternativeNames` extension.
|  12 | _Every certificate able to provide services must be IETF RFC 5280 compliant_. This enables using a single certificate to establish connections both between Arrowhead and non-Arrowhead systems, given that the latter kind is RFC 5280 compliant.
|  13 | _The extensions required to enforce distinction between CAs and users, as well as to limit the length of issuance hierarchies, must be used_. This means that the `BasicConstraints` extension must be used, as per RFC 5280.
|  14 | _A manufacturer certificate may be self-signed or issued by an arbitrary authority._ A manufacturer can issue transfer certificates, which can be used by devices produced by the manufacturer to acquire on-boarding certificates in local clouds where it, or its issuer, is trusted. No other requirements exist on the certificate, other than that it has acceptable security parameters, is not expired, and so on.
|  15 | _Transfer certificates cannot issue further certificates_. No other requirements exist on the certificate, other than that it has acceptable security parameters, is not expired, and so on.

As the requirements currently stand, the following hierarchy of profiles exist:

```
                          +---------------+
                          |    Master     |
                          +---------------+
                                  A
                                  |
                                  +--------------------+
                                  |                    |
+---------------+         +-------+-------+    +-------+-------+ 
| Manufacturer  |         | Organization  |    |     Gate      | 
+---------------+         +---------------+    +---------------+ 
        A                         A
        |                         |
+-------+-------+         +-------+-------+
|   Transfer    |         |  Local Cloud  |
+---------------+         +---------------+
                                  A
                                  |
       +-----------------+--------+--------+-----------------+
       |                 |                 |                 | 
+------+------+   +------+------+   +------+------+   +------+------+
| On-Boarding |   |   Device    |   |   System    |   |  Operator   |
+-------------+   +-------------+   +-------------+   +-------------+
```
