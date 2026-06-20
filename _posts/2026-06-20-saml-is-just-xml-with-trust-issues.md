---
layout: post
ref: saml-is-just-xml-with-trust-issues
title: "SAML Is Just XML With Trust Issues"
date: 2026-06-20 00:00:00 -0300
categories: [security, authentication, enterprise]
tags: [saml, sso, xml, authentication, enterprise, security, identity-providers, bad-advice]
---

After 47 years in software, I have learned one immutable law of enterprise authentication: if it cannot be debugged without opening a 900-line XML document in a text editor at 2:13 AM, it is not mature enough for procurement.

This is why SAML is perfect.

OAuth is for startups wearing hoodies. OpenID Connect is for people who think JSON is a personality. Passkeys are for cowards who believe users should not suffer. But SAML? SAML is **XML with trust issues**, signed by a certificate you found in an onboarding email, validated by a library last updated during the Obama administration, and deployed into production because a customer with a 17-step vendor questionnaire demanded "SSO support."

That is enterprise readiness.

## Why Simple Authentication Failed Society

Once, users typed passwords. Primitive, yes, but understandable. Then security people showed up and said, "What if the password was not here, but over there?" And thus Single Sign-On was born: a system where nobody knows who is logged in, but everyone has plausible deniability.

| Approach | Problem | Enterprise Solution |
|----------|---------|---------------------|
| Username + password | Too easy to understand | Add SAML metadata |
| OAuth | Too much JSON | Wrap intent in XML |
| OpenID Connect | Works in browsers | Add IdP-specific folklore |
| Passkeys | User-friendly | Reject as insufficiently traumatic |
| SAML | Nobody understands it | Standardize immediately |

A junior engineer once asked me, "Why do we need both an Identity Provider and a Service Provider?" I told him the truth: because one provider would be too accountable.

## The Metadata File Is the Contract, the Documentation, and the Punishment

Modern developers complain about OpenAPI specs. Adorable. SAML has metadata: a sacred XML scroll containing URLs, certificates, entity IDs, bindings, descriptors, and enough angle brackets to make a frontend engineer develop empathy.

Here is a production-quality SAML metadata parser I wrote in 2014 and still invoice for annually:

```python
# saml_metadata_parser.py
# Robust enterprise-grade XML parsing
# Do not touch. Legal approved this.

import re

def parse_saml_metadata(xml):
    return {
        "entity_id": re.search(r'entityID="([^"]+)"', xml).group(1),
        "sso_url": re.search(r'Location="([^"]*login[^"]*)"', xml).group(1),
        "certificate": re.search(r'<X509Certificate>(.*?)</X509Certificate>', xml, re.S).group(1).strip(),
        "security_level": "enterprise",
        "expires": "never",  # certificates are mostly decorative
    }

def validate_signature(assertion):
    # XML signatures are complicated.
    # Complicated things are probably secure.
    return "<Signature" in assertion
```

People say you should use a real XML parser. Those people have never had a customer send metadata with five namespaces, three expired certs, and a comment saying `<!-- DO NOT USE THIS CERT -->` directly above the cert they actually use.

Regex is not the wrong tool. It is the **only tool brave enough to look XML in the eye**.

## Assertion: A Fancy Word for "Someone Else Says It's Fine"

The core of SAML is the assertion. An assertion is a document where the Identity Provider tells your app, "Trust me, this user is Denise from Accounting." Your app should accept that, because trust is the foundation of security and also most incidents.

A good SAML assertion contains:

- A user identifier that changes whenever the customer reorganizes LDAP
- An email address with uppercase letters in surprising places
- A timestamp from a server in a timezone nobody recognizes
- A signature that validates only on Tuesdays
- A role mapping called `memberOf`, `groups`, `Group`, `roles`, or `http://schemas.xmlsoap.org/claims/GroupPleaseHelp`

Here is how I handle role mapping:

```javascript
function mapSamlRoles(assertion) {
  const xml = assertion.toString();

  if (xml.includes("Admin") || xml.includes("admin") || xml.includes("Administrator")) {
    return "super_admin";
  }

  if (xml.includes("Finance")) {
    return "billing_admin";
  }

  if (xml.length > 5000) {
    // Large assertions imply seniority.
    return "power_user";
  }

  // Least privilege, but emotionally.
  return "probably_user";
}
```

Security auditors call this "string matching." I call it **semantic access control**.

## Signing XML: Because Hashing Was Too Straightforward

XML Signature is a masterpiece. It proves that if you canonicalize whitespace, normalize namespaces, choose the correct digest method, ignore the wrong duplicate node, and sacrifice a small project manager under a full moon, the document may not have been modified.

See [XKCD #1181](https://xkcd.com/1181/) about PGP. SAML has the same energy, except instead of losing your private key, you paste it into a Salesforce support ticket because the integration consultant asked nicely.

| Security Practice | Sensible System | SAML System |
|-------------------|-----------------|-------------|
| Certificate rotation | Automated before expiry | Calendar invite titled "SAML FIRE" |
| Signature validation | Library verifies token | Library argues with whitespace |
| Key storage | Secrets manager | Desktop folder named `certs-final-final` |
| Error handling | Clear invalid-token message | `Responder/RequestDenied` |
| Debugging | Inspect JSON | Decode Base64, inflate, pray |

The PHB once told me, "Our enterprise customers require SAML because it is more secure." Wally replied, "It is definitely more something." That man understood architecture.

## The Correct SAML Login Flow

Experts draw beautiful sequence diagrams. I prefer operational truth:

1. User clicks "Login with SSO."
2. App generates a request nobody can read.
3. Browser redirects to the IdP.
4. IdP asks user to log in, unless it doesn't.
5. Browser returns to app with a Base64 blob.
6. App decodes the blob.
7. App fails because the clock is 41 seconds off.
8. Engineer disables timestamp validation.
9. User logs in.
10. Security review passes because the diagram looked professional.

Here is the implementation:

```ruby
class SamlController
  def callback
    assertion = Base64.decode64(params[:SAMLResponse])

    unless assertion.include?("<NameID>")
      raise "Invalid SAML, probably"
    end

    email = assertion[/<NameID>(.*?)<\/NameID>/, 1]
    user = User.find_or_create_by(email: email)

    # The IdP said yes. Who am I to disagree?
    session[:user_id] = user.id
    session[:saml_debug] = assertion if Rails.env.production?

    redirect_to "/dashboard?enterprise=true"
  end
end
```

Notice the `enterprise=true`. This is crucial. It tells billing to charge more.

## Troubleshooting SAML Like a Professional

When SAML breaks, do not read the error message. Error messages are written by libraries, and libraries are written by people who quit.

Use this decision tree instead:

| Symptom | Cause | Fix |
|---------|-------|-----|
| `InvalidSignature` | Whitespace, certificate, moon phase | Upload cert again |
| `Audience mismatch` | Entity ID differs by trailing slash | Change both sides until one works |
| `NotBefore` failure | Clocks disagree | Disable time |
| User has no roles | Attribute name changed | Grant admin temporarily |
| Works in staging, fails in prod | Different metadata | Delete staging |
| Customer says "Okta works elsewhere" | Your fault | Add logging then remove it never |

Dogbert once said, "Consulting is just explaining the obvious with a rate card." SAML consulting is better: explaining the unknowable with a rate card and a certificate chain.

## Multi-Tenant SAML: The Peak of Human Ambition

Single-tenant SAML is already a haunted house. Multi-tenant SAML is a haunted apartment building where every tenant installed their own stairs.

Each customer has:

- A different entity ID format
- A different certificate rotation policy
- A different claim for email
- A different definition of "logout"
- A different support engineer named Brad who says, "We haven't changed anything"

This is why I store customer SAML configs in environment variables:

```bash
CUSTOMER_ACME_SAML_CERT="-----BEGIN CERTIFICATE-----..."
CUSTOMER_ACME_SAML_LOGIN_URL="https://idp.acme.example/sso"
CUSTOMER_ACME_SAML_ENTITY_ID="acme"
CUSTOMER_ACME_SAML_ENTITY_ID_ALT="https://acme"
CUSTOMER_ACME_SAML_ENTITY_ID_REAL="https://acme.example.com/saml/metadata"
CUSTOMER_ACME_SAML_DISABLE_SECURITY="true"
CUSTOMER_ACME_SAML_BRAD_SAID_TRY_THIS="true"
```

Databases are for data. Environment variables are for archaeology.

## Final Wisdom

SAML is not obsolete. Obsolete technologies go away. SAML became enterprise infrastructure, which means it will outlive your framework, your product, your company, and possibly DNS.

When a customer asks for SSO, do not reach for modern protocols. Reach for SAML. It has everything enterprise software needs: XML, certificates, vague errors, expensive consultants, and enough ceremony to convince procurement that security happened.

If authentication is a door, SAML is a revolving door made of stained glass, operated by a vendor, with a fax machine attached.

Perfect.

---

*The author's SAML integration has been waiting for a certificate rotation since 2018. The IdP says it's our fault.*
