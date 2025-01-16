
# Recommendations {#recommendations}

## Validation Record Owner Name {#name}

The RECOMMENDED format for a Validation Record's owner name is application-specific underscore prefix labels. Domain Control Validation Records are constructed by the Application Service Provider by prepending the label "`_<PROVIDER_RELEVANT_NAME>-challenge`" to the domain name being validated (e.g. "\_service-challenge.example.com"). The prefix "_" is used to avoid collisions with existing hostnames and to prevent the owner name from being a valid hostname.

If an Application Service Provider has an application-specific need to have multiple validations for the same label, multiple prefixes can be used, such as "`_<FEATURE>._<PROVIDER_RELEVANT_NAME>-challenge`".

An Application Service Provider may also specify prepending a random token to the owner name of a validation record, such as "`_<RANDOM_TOKEN>._<PROVIDER_RELEVANT_NAME>-challenge`". This can be done either as part of the challenge itself ({{cname-dcv}}), to support multiple Intermediaries ({{multiple}}), or to make it harder for a third party to scan what Application Service Providers are being used by a given domain name.

## Time-bound checking


One exception is if the record is being used as part of a delegated domain control validation setup ({{delegated}}); in that case, the CNAME record that points to the actual validation TXT record cannot be removed as long as the User is still relying on the Intermediary.

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


### CNAME based {#cname-examples}

#### CNAME for Domain Control Validation {#cname-dcv-examples}

##### DocuSign

{{DOCUSIGN-CNAME}} asks the User to add a CNAME record with the "Host Name" set to be a 32-digit random value pointing to `verifydomain.docusign.net.`.

##### Google Workspace

{{GOOGLE-WORKSPACE-CNAME}} lets you specify a CNAME record for verifying domain ownership. The User gets a unique 12-character string that is added as "Host", with TTL 3600 (or default) and Destination an 86-character string beginning with "gv-" and ending with ".domainverify.googlehosted.com.".

#### Delegated Domain Control Validation {#delegated-examples}

##### Content Delivery Networks (CDNs): Akamai and Cloudflare

In order to be issued a TLS cert from a Certification Authority like Let's Encrypt, the requester needs to prove that they control the domain. Often this is done via the {{DNS-01}} challenge. Let's Encrypt only issues certs with a 90 day validity period for security reasons {{LETSENCRYPT-90-DAYS-RENEWAL}}. This means that after 90 days, the DNS-01 challenge has to be re-done and the random token has to be replaced with a new one. Doing this manually is error-prone. Content Delivery Networks like Akamai and Cloudflare offer to automate this process using a CNAME record in the User's DNS that points to the Validation Record in the CDN's zone ({{AKAMAI-DELEGATED}} and {{CLOUDFLARE-DELEGATED}}).

##### AWS Certificate Manager (ACM)

AWS Certificate Manager {{ACM-CNAME}} allows delegated domain control validation {{delegated}}. The record name for the CNAME looks like:

     _<random-token1>.example.com.  IN   CNAME _<random-token2>.acm-validations.aws.

The CNAME points to:

     _<random-token2>.acm-validations.aws.  IN   TXT "<random-token3>"

Here, the random tokens are used for the following:

* `<random-token1>`: Unique sub-domain, so there's no clashes when looking up the Validation Record.
* `<random-token2>`: Proves to ACM that the requester controls the DNS for the requested domain at the time the CNAME is created.
* `<random-token3>`: The actual token being verified.

Note that if there are more than 5 CNAMEs being chained, then this method does not work.
