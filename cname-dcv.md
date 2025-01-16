
# CNAME Records for Domain Control Validation {#cname-dcv}

CNAME records MAY be used instead of TXT records where specified by Application Service Providers to support Users who are unable to create TXT records. Two forms of this are common: including the random token in the owner name of a validation record, or including the random token as a part of the CNAME target. This approach has a number of limitations relative to using TXT records.

## Random Token in Owner Names

Application Service Providers MAY specify that a random token be included in the owner name of a validation record.  In this case an underscore-prefixed label MUST be used (e.g., `_<token>._service` or `_service-<token>`). The resource record is then a CNAME to a domain name specified by the Application Service Provider. The Application Service Provider uses the presence of a resource record at the CNAME target to perform the validation, validating the both presence of the record as well as the CNAME target. The CNAME target of the Validation Record MUST exist in order to verify the domain. For example:

    _<random-token>._service-challenge.example.com.  IN   CNAME dcv.provider.example.

In practice, many Application Service Providers that employ CNAMEs for domain control validation today use an entirely random subdomain label which works to avoid accidential collisions, but which could allow for a malicious Application Service Provider to smuggle instructions from some other Application Service Provider. Adding an provider-specific component in addition (such as `_<token>._service-challenge` or `_service-<token>-challenge`) make it easier for the domain owner to keep track of why and for what service a Validation Record has been deployed.

Since the random token exists entirely in the challenge, it is not possible to delegate Domain Control Validation challenges of this form to Intermediaries in a way that allows the Intermediary to refresh the challenge over time.

## Random Token in CNAME Targets

An Application Service Provider MAY specify using CNAME records instead of TXT records for Domain Control Validation. In this case, the target of the CNAME would contain the base16-encoded (or base32-encoded) random token followed by a suffix specified by the Application Service Provider. For example:

    _service-challenge.example.com.  IN   CNAME <random-token>.dcv.provider.example.

The Application Service Provider then validates that the target of the CNAME matches the token provided. This approach has similar properties to TXT records ({{txt-record}}) but does not allow for additional attributes such as expiry to be added.

As mentioned in {{cname-considerations}}, the owner name of the Validation Record MUST be distinct from the domain name being validated.

## Delegated Domain Control Validation {#delegated}

Delegated domain control validation lets a User delegate the domain control validation process for their domain to an Intermediary without granting the Intermediary the ability to make changes to their domain or zone configuration.  It is a variation of TXT record validation ({{txt-record}}) that indirectly inserts a CNAME record prior to the TXT record.

The Intermediary gives the User a CNAME record to add for the domain and Application Service Provider being validated that points to the Intermediary's domain, where the actual validation TXT record is placed. The record name and base16-encoded (or base32-encoded) random tokens are generated as in {{random-token}}. For example:

    _service-challenge.example.com.  IN   CNAME  <intermediary-random-token>.dcv.intermediary.example.

The Intermediary then adds the actual Validation Record in a domain they control:

    <intermediary-random-token>.dcv.intermediary.example.  IN   TXT "<provider-random-token>"

Such a setup is especially useful when the Application Service Provider wants to periodically re-issue the challenge with a new provider random token. CNAMEs allow automating the renewal process by letting the Intermediary place the random token in their DNS zone instead of needing continuous write access to the User's DNS.

Importantly, the CNAME record target also contains a random token issued by the Intermediary to the User (preferably over a secure channel) which proves to the Intermediary that example.com is controlled by the User. The Intermediary must keep an association of Users and domain names to the associated Intermediary-random-tokens. Without a linkage validated by the Intermediary during provisioning and renewal there is the risk that an attacker could leverage a "dangling CNAME" to perform a "subdomain takeover" attack ({{SUBDOMAIN-TAKEOVER}}).

When a User stops using the Intermediary they should remove the domain control validation CNAME in addition to any other records they have associated with the Intermediary.

See {{delegated-examples}} for examples.

## Domain Control Validation Supporting Multiple Intermediaries {#multiple}

There are use-cases where a User may wish to simultaneously use multiple intermediaries or multiple independent accounts with an Application Service Provider. For example, a hostname may be using a "multi-CDN" where the hostname simultaneously uses multiple Content Delivery Network (CDN) providers.

To support this, Application Service Providers may support prefixing the challenge with a label containing an unique account identifier of the form `_<identifier-token>` and following the requirements of {{random-token}}, specified as either base32 or base16 encoded. This identifier token should be stable over time and would be provided to the User by the Application Service Provider, or by an Intermediary in the case where domain validation is delegated ({{delegated}}).

The resulting record could either directly contain a TXT record or a CNAME (as in {{delegated}}).  For example:

    _<identifier-token>._service-challenge.example.com.  IN   TXT  "3419...3d206c4"

or

    _<identifier-token>._service-challenge.example.com.  IN   CNAME  <intermediary-random-token>.dcv.intermediary.example.

When performing validation, the Application Service Provider would resolve the DNS name containing the appropriate identifier token.

Application Service Providers may wish to always prepend the `_<identifier-token>` to make it harder for third parties to scan, even absent supporting multiple intermediaries.  The `_<identifier-token>` MUST start with an underscore so as to not be a valid hostname.
