
# WE BUILD - Conformance Specification: Identity Matching

Version 0.5 Date: 30-June-2026

Authors / Contributors: 
- Michelle Ludovici
- Malin Norlander
- Laurent Loup

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Scope](#2-scope)
- [3. Normative Language](#3-normative-language)
- [4. Roles and Components](#4-roles-and-components)
- [5. Protocol Overview](#5-protocol-overview)
- [6. High-level Flows](#6-high-level-flows)
- [7. Normative Requirements](#7-normative-requirements)
- [8. Interface Definitions](#8-interface-definitions)
- [9. Conformance](#9-conformance)
- [References](#references)

# 1. Introduction

This document defines the **WE BUILD Conformance Specification for Identity Matching**.

Its purpose is to describe how Identity matching can be tested within WeBuild based on three use-cases.
- The first type of use-case mandates by law that an official identifier from an authentic source is to be registered. This is the case for some public bodies and defined by national law. For this use-case a unique persistent identifier will be used in the PID.
- In the second use-case, the RP only wants to uniquely and persistently be able to identify a natural person within the EU, so that the person can return and continue an official matter that they started in an online flow. This can be solved with directed pseudonyms in combination with a PID.
- in the third use-case, enrichment of the PID-data with a Photo ID is tested for uniquenes and persistency, while recognising that overuse of Photo ID could lead to oversharing of personal information.

This specification should:

- identify the relevant protocol or functional area
- clarify which actors are involved
- define the main requirements needed for interoperability
- support implementation and conformance testing

This specification is based on [PR #223](https://github.com/webuild-consortium/wp4-architecture/pull/223/changes) and should be read together with other applicable WE BUILD specifications where relevant. 
> [!WARNING]
> REFERENCE TO BE CHANGED ONCE THE ADR IS ACCEPTED

# 2. Scope

This Conformance Specification focuses exclusively on identity matching for users operating within WE BUILD.

### In Scope
* Initial Onboarding: Identity matching and verification workflows for users connecting to a Relying Party (RP) for the first time using the conformance profiles defined in this document.
* Returning Pseudonymous Users: The ongoing ability for a user to return to a previously visited RP and resume an official matter securely, leveraging the persistence of a derived directed pseudonym or unique identifier.

### Out of Scope
* Legacy/Historic User Migration: Database reconciliation, identity matching, or account-linking for historic users who established local official records or accessed digital e-services prior to the deployment of these wallet-based pseudonym profiles.

Although there are many potential structural solutions for managing identity matching across borders, these three specific use-cases have been chosen for their coverage of varying legal mandates and progressive degrees of technical implementation complexity.

# 3. Normative Language

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **MUST**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These keywords indicate the strength of requirements for conforming implementations.

# 4. Roles and Components

This section identifies the roles and components relevant to this specification.

Only roles that matter for this specification should be included.

Examples may include:

- **PID-Issuer:** entity issuing the PID and supplying a seed to enable reidentification. The PID Provider key-binding behaviour follows [CS-04](https://github.com/webuild-consortium/wp4-architecture/blob/main/conformance-specs/cs-04-wua-lifecycle.md)
- **Wallet-provider:** entity providing the EUDIW 
- **Verifier / Relying Party:** entity requesting and validating presentations
- **Issuer of Photo-ID:** entity issuing an official Photo-ID
- **Trust Provider:** component or service publishing trust-related information
- **Pseudonym Service:** service that generates the directed pseudonym. This service could be stand-alone, integrated in the ITB, or managed directly by the Wallet-provider

# 5. Protocol Overview

This section gives a short explanation of how the protocol or function works at a high level.

## Use-case 1: Official unique persistent identifier from authentic source in PID 
The PID Provider **MUST** include an official unique persistent identifier as an attribute within the Person Identification Data (PID) credential. To protect user privacy, the PID implementation **MUST** support Selective Disclosure (SD) for this identifier attribute.

#### Identifier Requirements and Constraints
1. Uniqueness and Persistency: The identifier **MUST** remain unique and persistent across the scope of all WE BUILD conformance testing environments.
2. National Identifiers: Where legally permissible and technically available, implementations **MAY** utilize official national unique natural person identifiers (e.g., the Swedish *personnummer*).
3. Fallback Identifier (UUIDv4): In jurisdictions where a national persistent identifier is unavailable, or where privacy constraints restrict its transmission, a Version 4 Universally Unique Identifier (UUIDv4) **MUST** be generated as a fallback. 
   * Generated UUIDv4 values **MUST** strictly conform to [RFC 9562](https://www.rfc-editor.org/info/rfc9562/#name-uuid-version-4), utilizing lowercase hexadecimal strings partitioned by hyphens into the standard `8-4-4-4-12` character pattern.
     
#### Schema Implementation
To satisfy conformance verification for this use-case, the unique identifier **MUST** be integrated into the `properties` block of the official PID schema definition ([ds002-pid-sd-jwt.json](https://github.com/webuild-consortium/webuild-attestation-rulebooks-catalog/blob/main/data-schemas/sd-jwt/ds002-pid-sd-jwt.json)) as specified in the following structural definition:

```json
"properties": {
  "family_name": {
    "type": "string",
    "description": "Current last name(s), surname(s), or primary identifier of the user.",
    "examples": [
      "Smith"
    ]
  },
  "given_name": {
    "type": "string",
    "description": "Current first name(s) of the user.",
    "examples": [
      "Alice"
    ]
  },
  "unique_id": {
    "type": "string",
    "format": "uuid",
    "description": "A unique, persistent identifier such as a UUIDv4.",
    "examples": [
      "7b3e9a1c-fd84-4c6e-92b1-5a63f82b410d"
    ]
  },
  "birth_date": {
    "type": "string",
    "format": "date",
    "description": "The date of birth of the user.",
    "examples": [
      "1970-01-01"
    ]
  }
}
```

This use-case eliminates the need for identity matching and therefore does not need any high-level flows or other details in the specification. 

## Use-case 2: Directed Pseudonyms

This use-case is based on the architectural description of directed pseudonyms defined in [Pseudonyms for the EUDI Wallet](https://github.com/AltmannPeter/webuild-architecture/blob/bfa775c22a0f111f2412032da9cfc9bd26fba810/webuild-drafts/pseudonyms.md) by Peter Altmann.

Within the WeBuild implementation framework, the pseudonym service **MAY** be deployed as a stand-alone service, integrated into the ITB, or managed directly by the Wallet Provider. The support for relying party-specific pseudonyms by wallet providers is explicitly mandated by [Regulation (EU) 2024/1183](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1183), Article 5a(4)(b): 

> *"European Digital Identity Wallets MUST enable the user, in a manner that is user-friendly, transparent, and traceable by the user, to: [...] generate pseudonyms and store them encrypted and locally within the European Digital Identity Wallet;"*

Furthermore, Article 5b(9) establishes the corresponding acceptance obligations for external services:

> *"Relying parties MUST be responsible for carrying out the procedure for authenticating and validating person identification data and electronic attestation of attributes requested from European Digital Identity Wallets. Relying parties MUST not refuse the use of pseudonyms, where the identification of the user is not required by Union or national law."*



### Pseudonym Generation Workflow

#### Step 1: Seed Generation by the PID Provider
The lifecycle begins at the PID Provider (Issuer), who is responsible for establishing the cryptographic root of the user's pseudonyms.

1. **Generation:** The PID Provider **MUST** generate a cryptographically secure, high-entropy secret known as the pseudonym seed (`nym_seed`).
2. **Requirements:** The `nym_seed` **MUST** be generated using a cryptographically secure pseudorandom number generator (CSPRNG) and **MUST** maintain a minimum key length of 256 bits (32 bytes) to resist brute-force vectors.
3. **Transmission:** The seed **MUST** be securely bound to the user's PID. It **MAY** be supplied directly by the issuer inside the credential payload or communicated securely during issuance via authorized protocols (e.g., OID4VCI flow H.5).

#### Step 2: Site and Context-Specific Derivation
Once the `nym_seed` is established, the pseudonym service **MUST** compute the specific directed pseudonym dynamically whenever the user interacts with a Relying Party (RP). 

The pseudonym value **MUST** be calculated using $HMAC-SHA256$, combining the secret seed with the target domain and an optional application context:

$$\text{Pseudonym} = \text{HMAC-SHA256}(\text{nym}_{\text{seed}}, \text{"directed:"} \mathbin{\Vert} \text{rp}_{\text{identifier}} \mathbin{\Vert} \text{ps}_{\text{context}})$$

Where:
* `rp_identifier`: The unique domain name or identifier of the Relying Party (e.g., `google.com`). This parameter is REQUIRED and MUST be used to isolate the resulting pseudonym to that specific Relying Party.
* ps_context: An application, service, or session sub-context string used to isolate separate profiles under the same domain (e.g., colab.research). If no sub-context isolation is required for the transaction, this parameter MUST be passed as an empty string ("") to ensure a deterministic cryptographic concatenation block.

*Deterministic Result Example:* For an issuer-supplied seed of `FrvCFWys...`, an `rp_identifier` of `google.com`, and a `ps_context` of `colab.research`, the derived pseudonym output string evaluates deterministically to: `7OMvywPJlFjbblVFkjUJb6gR-AgGnxRf5j7XEBn3CFk`.

For an executable reference implementation of this derivation formula and test vectors, see the [WE BUILD Pseudonym Jupyter Notebook](https://github.com/AltmannPeter/webuild-architecture/blob/bfa775c22a0f111f2412032da9cfc9bd26fba810/webuild-drafts/pseudonyms.ipynb).

#### Step 3: SD-JWT Payload Representation and Disclosure

To support selective disclosure, the derived pseudonym is not exposed in plaintext inside the core credential structure. Instead, it is obfuscated using salted hashes within the SD-JWT framework.
1. Disclosure Creation: The wallet or provider MUST package the calculated pseudonym along with a unique random salt and the claim key into a standardized JSON disclosure array (e.g., ["_2BB69p5Yxl9Z_g3Q", "unique_id", "7OMvywPJlFjbblVFkjUJb6gR-AgGnxRf5j7XEBn3CFk"]).
2. Base64URL Encoding: This disclosure array string MUST be encoded into a Base64URL string without padding.
3. Hashing: The Base64URL string MUST be transformed via a $SHA-256$ hash function, and the resulting binary digest MUST be encoded into a Base64URL string.
4. Token Ingestion: Only the final Base64URL-encoded hash string MUST be appended to the public _sd array of the PID token stream. This ensures that third-party observers cannot track, decode, or link the user across sessions without the wallet explicitly presenting the corresponding disclosure snippet.

```json
{
  "issuer_signed_jwt": "eyJhbGciOi...",
  "disclosures": [
    "_2BB69p5Yxl9Z_g3QInVuaXF1ZV9pZCIsIjdPTXZ5d1BKbEZqYmJsVkZraiJd"
  ],
  "kb_jwt": {
    "protected": { "alg": "ES256", "typ": "kb+jwt" },
    "payload": {
      "iat": 1719734400,
      "aud": "https://google.com",
      "nonce": "n-0S6_WzA2Mj",
      "sd_hash": "9pNH0o0VTYjvqJqg3wYbKJtFq0R8X9q6kZYzGfO5pjA"
    }
  }
}
```
where sd_hash = SHA-256(issuer_signed_jwt || "~" || disclosure_1 || "~" || ... || "~"), computed over the final string that the wallet transmits, including the freshly-appended pseudonym disclosure.

## Use-case 3: Enrichment of PID with Photo-ID

> [!TBD]
> BY LAURENT LOUP


# 6. High-level Flows for Use-case 2 (Directed pseudonyms)

This section describes the main interaction flows between actors.

## 6.1 Directed Pseudonym Provisioning (Issuance)

This flow describes how the initial cryptographic material is generated and securely bound to the user's credential.

### Participating Actors
* **PID Provider (Issuer):** The authoritative body issuing the Person Identification Data.
* **User / Wallet:** The holder requesting the credential and storing the cryptographic keys.

### How the Interaction Begins
The interaction begins when the User initiates a request for a new PID credential inside their Wallet application (e.g., via scanning a QR code or clicking an issuance link).

### Main Sequence of Actions
1. The Wallet establishes a secure session with the PID Provider using the OID4VCI protocol.
2. The PID Provider generates a cryptographically secure, high-entropy master seed (`nym_seed`).
3. The PID Provider binds this `nym_seed` directly to the user's core profile context.
4. The PID Provider packages the seed securely (either directly within the encrypted credential metadata or injected via OID4VCI flow H.5).
5. The PID Provider signs the credential object and delivers it to the Wallet.

### Expected Outcome
The Wallet successfully stores the issued PID credential along with the hidden master `nym_seed`, ready to be used for future site-specific derivations.

---

## 6.2 Directed Pseudonym Derivation and Presentation

This flow describes how the wallet dynamically derives a site-specific pseudonym and presents it to a relying party using selective disclosure.

### Participating Actors
* **User / Wallet:** The holder presenting the pseudonym.
* **Relying Party (Verifier):** The service provider requesting user authentication (e.g., `google.com`).

### How the Interaction Begins
The interaction begins when the User attempts to access a service on the Relying Party's platform that accepts pseudonymous eIDAS authentication, prompting the RP to present an OID4VP request.

### Main Sequence of Actions
1. The Relying Party transmits an OID4VP authorization request containing a `presentation_definition` that queries for a "unique_id". This request includes the mandatory `rp_identifier` (domain) and an optional `ps_context`.
2. The Wallet parses the request and extracts the `rp_identifier` and `ps_context`.
3. The Wallet executes an $HMAC-SHA256$ computation using the stored master nym_seed as the cryptographic key, and the combined domain/context data string as the message data. The wallet MUST inject the extracted rp_identifier into this calculation to ensure that a unique, site-specific pseudonym is generated for each unique Relying Party, alongside the optional ps_context string.
4. The Wallet packages this derived pseudonym string into an SD-JWT disclosure array along with a random salt.
5. The Wallet retrieves the previously issued SD-JWT credential (containing family_name, given_name, birth_date, and the cnf confirmation key, but no pseudonym-related digest), and packages the freshly computed pseudonym disclosure for inclusion at presentation time.
6. The Wallet appends the disclosure to the credential, computes sd_hash over the resulting Combined Format string, signs a Key-Binding JWT (kb+jwt) over iat, aud, nonce, and sd_hash using its device key, and transmits the issuer-signed JWT, the disclosure, and the KB-JWT together to the Relying Party
7. The Relying Party validates the Issuer's signature on issuer_signed_jwt and confirms the cnf key matches the device key used below; validates the Wallet's KB-JWT signature against that cnf key; recomputes sd_hash over the received Combined Format string and confirms it matches the KB-JWT's sd_hash claim; and extracts the disclosed pseudonym from the disclosure array.

   
### Expected Outcome
The Relying Party securely authenticates the user via a persistent, site-specific identifier without learning the user's real-world identity or master seed, preventing tracking across other relying parties.

# 7. Normative Requirements for use-case 2 (Directed Pseudonyms)



## 7.1 Issuance and Provisioning Component

The Pseudonym Service and PID Provider **MUST**:

1. Use the [OpenID for Verifiable Credential Issuance 1.0](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) (`OID4VCI`) protocol for secure credential provisioning and token exchange.
2. Structure all verifiable credential metadata in compliance with the [SD-JWT-based Verifiable Digital Credentials](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/) (`draft-ietf-oauth-sd-jwt-vc`) profiling specification.

The Pseudonym Service and PID Provider **SHOULD**:

1. Utilize the OID4VCI flow H.5 protocol extension if the user-supplied seed method is selected for transmission.

The Pseudonym Service and PID Provider **MUST NOT**:

1. Transmit the raw, unhashed master pseudonym seed (`nym_seed`) within the public unencrypted token payload structures.

The PID Provider **MUST**:
1. Per the key-binding procedure defined in CS-04 [Conformance Specification CS-04: Individual Wallet Unit Attestation (WUA) Lifecycle], Annex B.1: verify the Key Attestation (KA) presented in the Credential Request, confirm proof of possession against attested_keys[0], and embed the corresponding public key as the cnf claim in the issued PID's issuer_signed_jwt payload. This cnf claim is the credential-level confirmation key used by the Wallet to compute sd_hash over Key-Binding JWTs at presentation (Section 5, Use-case 2, Step 3), and is distinct from the WIA-level cnf (DPoP key) used during the issuance session itself (CS-04, Annex B, closing note).



## 7.2 Presentation and Verification Component 

The Wallet and Relying Party (RP) **MUST**:

1. Utilize the [OpenID for Verifiable Presentations 1.0](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html) (`OID4VP`) transaction protocol to transmit identity tokens from the wallet container to the verifier.
2. Formulate and parse queries for selective attribute disclosure using the syntax defined in the [DIF Presentation Exchange 2.0](https://identity.foundation/presentation-exchange/spec/v2.0.0/) framework.
3. Validate the plaintext salts, disclosure arrays, and corresponding $SHA-256$ digests according to the validation rules of [RFC 9901: Selective Disclosure for JSON Web Tokens (SD-JWT)](https://datatracker.ietf.org/doc/html/rfc9901).

The Wallet **MUST**:
1.  Generate a Key-Binding JWT (kb+jwt) for each presentation containing the pseudonym disclosure, computing sd_hash over the full Combined Format string (issuer-signed JWT plus all disclosures, including the dynamically generated pseudonym disclosure) per the Key Binding requirements of draft-ietf-oauth-sd-jwt-vc.

The Relying Party **MUST**:
1. Verify the Key-Binding JWT signature against the cnf key bound in the issuer-signed JWT, and verify that sd_hash matches a recomputed digest of the received Combined Format string, before trusting any dynamically disclosed claim such as the pseudonym.

The Relying Party (RP) **MUST NOT**:

1. Refuse or block the processing of a validly presented directed pseudonym, unless explicit real-world identification of the natural person is legally mandated by Union or national law.

## 7.3 Trust Model Note
The Issuer's signature in issuer_signed_jwt attests to the user's core PID attributes and to a confirmation key (cnf) — the public key corresponding to the wallet's device key.
The RP's trust that this pseudonym value is legitimate now rests entirely on (a) verifying the KB-JWT signature against the cnf key from the issuer-signed JWT, and (b) trusting the wallet software to have correctly run the HMAC-SHA256 derivation against the real nym_seed. The Issuer no longer cryptographically attests to the pseudonym value itself — only to the fact that this device is the legitimate holder.
That is a weaker guarantee than the issuance-time approach: a compromised or non-conformant wallet could present an arbitrary unique_id value per RP and the RP has no way to detect that cryptographically, since nothing the Issuer signed constrains the value, only the holder. This residual risk is partially mitigated by the verification obligations in 7.2; see also CS-04 Annex B.2 for the underlying holder-binding mechanism.



# 8. Interface Definitions

This section describes the technical interfaces used in the Directed Pseudonym flow (Use-case 2). Use-case 1 (static unique_id in PID) requires no additional interfaces beyond standard PID issuance and is not repeated here. Use-case 3 interfaces are TBD pending Section 5.3.

## 8.1 Credential Issuance Request (Wallet → PID Provider)

*Direction:* Wallet → PID Provider (Credential Issuer)
*Method:* HTTP POST (OID4VCI Credential Endpoint)

**Request**
- `credential_configuration_id`: identifies the PID credential type being requested
- `proofs.jwt`: proof-of-possession JWT, with `key_attestation` header carrying the KA (per CS-04, Annex B.1)

**Response**
- `credential`: the issued SD-JWT-VC, containing the signed `issuer_signed_jwt` with `cnf` bound to the attested key (Section 7.1, item 5)
- `nym_seed` transmission (if using the in-band method): delivered via OID4VCI flow H.5 extension rather than as a top-level response field (Section 7.1, SHOULD item)

**Example (illustrative only)**
```text
POST /credential HTTP/1.1
Host: pid-provider.example.se
Authorization: DPoP 

{
  "credential_configuration_id": "se.pid.sd_jwt_vc",
  "proofs": { "jwt": ["eyJ..."] }
}
```

## 8.2 OID4VP Authorization Request (RP → Wallet)

*Direction:* Relying Party → Wallet
*Method:* HTTP redirect / QR-initiated OID4VP request (per OID4VP 1.0)

**Request**
- `presentation_definition`: DIF Presentation Exchange 2.0 query targeting the `unique_id` claim
- `rp_identifier`: REQUIRED, the RP's domain (Section 5, Use-case 2, Step 2)
- `ps_context`: OPTIONAL, application/session sub-context string; if absent, treated as empty string per Section 5
- `nonce`, `aud`: standard OID4VP anti-replay and audience-binding parameters

**Response**
- Not applicable (this is a request-only interface; the response is defined in 8.3)

**Example (illustrative only)**
```text
GET /authorize?
  presentation_definition=...&
  rp_identifier=google.com&
  ps_context=colab.research&
  nonce=n-0S6_WzA2Mj
```

## 8.3 OID4VP Authorization Response (Wallet → RP)

*Direction:* Wallet → Relying Party
*Method:* HTTP POST (OID4VP direct_post, or equivalent response mode)

**Request**
- `vp_token`: Combined Format for Presentation, comprising:
  - `issuer_signed_jwt`: the signed PID credential, including `cnf`
  - `disclosures`: array including the dynamically generated pseudonym disclosure (Section 6.2, step 4)
  - `kb_jwt`: Key-Binding JWT with `iat`, `aud`, `nonce`, `sd_hash` (Section 5, Use-case 2, Step 3)

**Response**
- HTTP 200 / redirect on successful validation (Section 6.2, step 7)
- HTTP 4xx with an OID4VP-conformant error object on validation failure (e.g., `sd_hash` mismatch, KB-JWT signature failure, revoked WUA per CS-04 Section 7.2)

**Example (illustrative only)**
```text
POST /callback HTTP/1.1
Host: google.com

vp_token={"issuer_signed_jwt":"eyJ...","disclosures":["..."],"kb_jwt":{...}}
```

## 8.4 WUA Status / Revocation Check (RP → Status List Endpoint)

*Direction:* Relying Party → Wallet Provider status list endpoint
*Method:* HTTP GET

Per CS-04, Section 8.2: used by the RP to verify that the Wallet Unit's KA (and, transitively, the `cnf` key embedded in the PID) has not been revoked. This specification defers entirely to CS-04 for the interface shape and does not redefine it.

**Request**
- `idx`: status list index referenced in the KA's `key_storage_status.status`
- `uri`: status list endpoint, as published in the KA

**Response**
- Token Status List response per draft-ietf-oauth-status-list-20, as pinned by CS-04 Section 8.2
```


# 9. Conformance

An implementation **conforms to this specification** if it implements the requirements defined in this document for its role and supports the relevant interfaces and flows. Conformance is stated per use-case and per role, since not all roles participate in all three use-cases.

## 9.1 Conformance for Use-case 1 (Static Unique Identifier)

An implementation **conforms as a PID Provider (Use-case 1)** if it:

1. issues a PID credential containing a `unique_id` attribute meeting the requirements of Section 5, Use-case 1 (uniqueness, persistence, and either a national identifier or UUIDv4 fallback per RFC 9562);
2. supports Selective Disclosure for the `unique_id` attribute per the schema definition in Section 5, Use-case 1;
3. uses the schema structure defined in `ds002-pid-sd-jwt.json`.

## 9.2 Conformance for Use-case 2 (Directed Pseudonyms)

An implementation **conforms as a PID Provider (Use-case 2)** if it:

1. implements the applicable MUST and MUST NOT requirements in Section 7.1;
2. generates and binds `nym_seed` as specified in Section 5, Use-case 2, Step 1, and Section 6.1;
3. embeds the `cnf` claim per the CS-04 Annex B.1 key-binding procedure referenced in Section 7.1, item 5.

An implementation **conforms as a Wallet (Use-case 2)** if it:

1. implements the applicable MUST requirements in Section 7.2;
2. correctly performs the HMAC-SHA256 pseudonym derivation specified in Section 5, Use-case 2, Step 2, including correct injection of `rp_identifier` and `ps_context`;
3. constructs the SD-JWT disclosure and Key-Binding JWT as specified in Section 5, Use-case 2, Step 3, and Section 6.2;
4. supports the flows in Section 6.1 and 6.2 and the interfaces in Section 8.1 to 8.3.

An implementation **conforms as a Relying Party (Use-case 2)** if it:

1. implements the applicable MUST and MUST NOT requirements in Section 7.2;
2. correctly validates the Issuer signature, KB-JWT signature, and `sd_hash` as specified in Section 6.2, step 7;
3. does not refuse a validly presented directed pseudonym except where real-world identification is legally mandated (Section 7.2);
4. supports the interfaces in Section 8.2 to 8.4, including WUA revocation status checks per CS-04.

## 9.3 Conformance for Use-case 3 (Photo-ID Enrichment)

Conformance criteria for Use-case 3 are TBD pending completion of Section 5, Use-case 3.

## 9.4 General

Conformance to this specification does not by itself imply conformance to CS-04; implementations acting as PID Provider, Wallet, or Relying Party under Use-case 2 must separately conform to the applicable CS-04 roles (Wallet Provider, Wallet Unit, Issuer/Verifier) for WUA issuance, key binding, and revocation handling.

Additional WE BUILD profiles may define stricter requirements for specific use-cases. Such profiles **MUST NOT** weaken the mandatory requirements in this specification.

# References

[1] IETF (1997) RFC 2119: Key words for use in RFCs to Indicate Requirement Levels. Available at: https://datatracker.ietf.org/doc/html/rfc2119 (Accessed: 30 June 2026).

[2] IETF (2024) RFC 9562: Universally Unique IDentifiers (UUIDs). Available at: https://www.rfc-editor.org/info/rfc9562 (Accessed: 30 June 2026).

[3] webuild-consortium (2025) ds002-pid-sd-jwt.json: PID SD-JWT VC Schema. Available at: https://github.com/webuild-consortium/webuild-attestation-rulebooks-catalog/blob/main/data-schemas/sd-jwt/ds002-pid-sd-jwt.json (Accessed: 30 June 2026).

[4] Altmann, P. (2025) Pseudonyms for the EUDI Wallet. Available at: https://github.com/AltmannPeter/webuild-architecture/blob/bfa775c22a0f111f2412032da9cfc9bd26fba810/webuild-drafts/pseudonyms.md (Accessed: 30 June 2026).

[5] Altmann, P. (2025) WE BUILD Pseudonym Jupyter Notebook (derivation formula and test vectors). Available at: https://github.com/AltmannPeter/webuild-architecture/blob/bfa775c22a0f111f2412032da9cfc9bd26fba810/webuild-drafts/pseudonyms.ipynb (Accessed: 30 June 2026).

[6] European Union (2024) Regulation (EU) 2024/1183 of the European Parliament and of the Council amending Regulation (EU) No 910/2014 as regards establishing the European Digital Identity Framework. Available at: https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32024R1183 (Accessed: 30 June 2026).

[7] OpenID Foundation (2025) OpenID for Verifiable Credential Issuance 1.0. Available at: https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html (Accessed: 30 June 2026).

[8] OpenID Foundation (2025) OpenID for Verifiable Presentations 1.0. Available at: https://openid.net/specs/openid-4-verifiable-presentations-1_0.html (Accessed: 30 June 2026).

[9] IETF (2025) SD-JWT-based Verifiable Credentials (SD-JWT-VC), draft-ietf-oauth-sd-jwt-vc. Available at: https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/ (Accessed: 30 June 2026).

[10] IETF (2025) RFC 9901: Selective Disclosure for JWTs (SD-JWT). Available at: https://datatracker.ietf.org/doc/html/rfc9901 (Accessed: 30 June 2026).

[11] Decentralized Identity Foundation (2021) Presentation Exchange 2.0.0. Available at: https://identity.foundation/presentation-exchange/spec/v2.0.0/ (Accessed: 30 June 2026).

[12] WE BUILD (2026) Conformance Specification CS-04: Individual Wallet Unit Attestation (WUA) Lifecycle, version 0.5. Available at: https://github.com/webuild-consortium/wp4-architecture/blob/main/conformance-specs/cs-04-wua-lifecycle.md (Accessed: 30 June 2026).

[13] webuild-consortium (2025) wp4-architecture, Pull Request #223. Available at: https://github.com/webuild-consortium/wp4-architecture/pull/223/changes (Accessed: 30 June 2026).
