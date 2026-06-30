
# WE BUILD - Conformance Specification: <TITLE>

Version 0.1

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

This Conformance Specification focuses exclusively on identity matching and session continuity for users operating within the WE BUILD persistent and pseudonymous identification frameworks.

### In Scope
* Initial Onboarding: Identity matching and verification workflows for users connecting to a Relying Party (RP) for the first time using the conformance profiles defined in this document.
* Session Continuity (Returning Pseudonymous Users) The ongoing ability for a user to return to a previously visited RP and resume an official matter securely, leveraging the persistence of a derived directed pseudonym or unique identifier.

### Out of Scope
* Legacy/Historic User Migration: Database reconciliation, identity matching, or account-linking for historic users who established local official records or accessed digital e-services prior to the deployment of these wallet-based pseudonym profiles.

Although there are many potential structural solutions for managing identity matching across borders, these three specific use-cases have been chosen for their coverage of varying legal mandates and progressive degrees of technical implementation complexity.

# 3. Normative Language

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These keywords indicate the strength of requirements for conforming implementations.

# 4. Roles and Components

This section identifies the roles and components relevant to this specification.

Only roles that matter for this specification should be included.

Examples may include:

- **PID-Issuer:** entity issuing the PID and supplying a seed to enable reidentification
- **Wallet-provider:** entity providing the EUDIW 
- **Verifier / Relying Party:** entity requesting and validating presentations
- **Issuer of Photo-ID:** entity issuing an official Photo-ID
- **Trust Provider:** component or service publishing trust-related information
- **Pseudonym Service:** service that generates the directed pseudonym. This service could be stand-alone, integrated in the ITB, or managed directly by the Wallet-provider

# 5. Protocol Overview

This section gives a short explanation of how the protocol or function works at a high level.

## Use-case 1: Official unique persistent identifier from authentic source in PID 
The PID Provider **SHALL** include an official unique persistent identifier as an attribute within the Person Identification Data (PID) credential. To protect user privacy, the PID implementation **SHALL** support Selective Disclosure (SD) for this identifier attribute.

#### Identifier Requirements and Constraints
1. Uniqueness and Persistency: The identifier **SHALL** remain unique and persistent across the scope of all WE BUILD conformance testing environments.
2. National Identifiers: Where legally permissible and technically available, implementations **MAY** utilize official national unique natural person identifiers (e.g., the Swedish *personnummer*).
3. Fallback Identifier (UUIDv4): In jurisdictions where a national persistent identifier is unavailable, or where privacy constraints restrict its transmission, a Version 4 Universally Unique Identifier (UUIDv4) **SHALL** be generated as a fallback. 
   * Generated UUIDv4 values **SHALL** strictly conform to [RFC 9562](https://www.rfc-editor.org/info/rfc9562/#name-uuid-version-4), utilizing lowercase hexadecimal strings partitioned by hyphens into the standard `8-4-4-4-12` character pattern.
     
#### Schema Implementation
To satisfy conformance verification for this use-case, the unique identifier **SHALL** be integrated into the `properties` block of the official PID schema definition ([ds002-pid-sd-jwt.json](https://github.com/webuild-consortium/webuild-attestation-rulebooks-catalog/blob/main/data-schemas/sd-jwt/ds002-pid-sd-jwt.json)) as specified in the following structural definition:

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

> *"European Digital Identity Wallets shall enable the user, in a manner that is user-friendly, transparent, and traceable by the user, to: [...] generate pseudonyms and store them encrypted and locally within the European Digital Identity Wallet;"*

Furthermore, Article 5b(9) establishes the corresponding acceptance obligations for external services:

> *"Relying parties shall be responsible for carrying out the procedure for authenticating and validating person identification data and electronic attestation of attributes requested from European Digital Identity Wallets. Relying parties shall not refuse the use of pseudonyms, where the identification of the user is not required by Union or national law."*



### Pseudonym Generation Workflow

#### Step 1: Seed Generation by the PID Provider
The lifecycle begins at the PID Provider (Issuer), who is responsible for establishing the cryptographic root of the user's pseudonyms.

1. **Generation:** The PID Provider **SHALL** generate a cryptographically secure, high-entropy secret known as the pseudonym seed (`nym_seed`).
2. **Requirements:** The `nym_seed` **SHALL** be generated using a cryptographically secure pseudorandom number generator (CSPRNG) and **SHALL** maintain a minimum key length of 256 bits (32 bytes) to resist brute-force vectors.
3. **Transmission:** The seed **SHALL** be securely bound to the user's PID. It **MAY** be supplied directly by the issuer inside the credential payload or communicated securely during issuance via authorized protocols (e.g., OID4VCI flow H.5).

#### Step 2: Site and Context-Specific Derivation
Once the `nym_seed` is established, the pseudonym service **SHALL** compute the specific directed pseudonym dynamically whenever the user interacts with a Relying Party (RP). 

The pseudonym value **SHALL** be calculated using $HMAC-SHA256$, combining the secret seed with the target domain and an optional application context:

$$\text{Pseudonym} = \text{HMAC-SHA256}(\text{nym}_{\text{seed}}, \text{"directed:"} \mathbin{\Vert} \text{rp}_{\text{identifier}} \mathbin{\Vert} \text{ps}_{\text{context}})$$

Where:
* `rp_identifier`: The unique domain name or identifier of the Relying Party (e.g., `google.com`). This parameter is REQUIRED and SHALL be used to isolate the resulting pseudonym to that specific Relying Party.
* ps_context: An application, service, or session sub-context string used to isolate separate profiles under the same domain (e.g., colab.research). If no sub-context isolation is required for the transaction, this parameter SHALL be passed as an empty string ("") to ensure a deterministic cryptographic concatenation block.

*Deterministic Result Example:* For an issuer-supplied seed of `FrvCFWys...`, an `rp_identifier` of `google.com`, and a `ps_context` of `colab.research`, the derived pseudonym output string evaluates deterministically to: `7OMvywPJlFjbblVFkjUJb6gR-AgGnxRf5j7XEBn3CFk`.

For an executable reference implementation of this derivation formula and test vectors, see the [WE BUILD Pseudonym Jupyter Notebook](https://github.com/AltmannPeter/webuild-architecture/blob/bfa775c22a0f111f2412032da9cfc9bd26fba810/webuild-drafts/pseudonyms.ipynb).

#### Step 3: SD-JWT Payload Representation and Disclosure

To support selective disclosure, the derived pseudonym is not exposed in plaintext inside the core credential structure. Instead, it is obfuscated using salted hashes within the SD-JWT framework.
1. Disclosure Creation: The wallet or provider SHALL package the calculated pseudonym along with a unique random salt and the claim key into a standardized JSON disclosure array (e.g., ["_2BB69p5Yxl9Z_g3Q", "unique_id", "7OMvywPJlFjbblVFkjUJb6gR-AgGnxRf5j7XEBn3CFk"]).
2. Base64URL Encoding: This disclosure array string SHALL be encoded into a Base64URL string without padding.
3. Hashing: The Base64URL string SHALL be transformed via a $SHA-256$ hash function, and the resulting binary digest SHALL be encoded into a Base64URL string.
4. Token Ingestion: Only the final Base64URL-encoded hash string SHALL be appended to the public _sd array of the PID token stream. This ensures that third-party observers cannot track, decode, or link the user across sessions without the wallet explicitly presenting the corresponding disclosure snippet.

```json
{
  "iss": "https://authentic-source.pid.se",
  "sub": "7b3e9a1c-fd84-4c6e-92b1-5a63f82b410d",
  "family_name": "Smith",
  "given_name": "Alice",
  "birth_date": "1970-01-01",
  "_sd": [
    "jsu9Knu7F_83Gv_Dsz89KlA73mN_oq1WxzP6Klm3bX0"
  ],
  "_sd_alg": "sha-256"
}
```

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
3. The Wallet executes an $HMAC-SHA256$ computation using the stored master nym_seed as the cryptographic key, and the combined domain/context data string as the message data. The wallet SHALL inject the extracted rp_identifier into this calculation to ensure that a unique, site-specific pseudonym is generated for each unique Relying Party, alongside the optional ps_context string.
4. The Wallet packages this derived pseudonym string into an SD-JWT disclosure array along with a random salt.
5. The Wallet retrieves the pre-computed, signed SD-JWT credential containing the obfuscated pseudonym hash within its _sd array, alongside the corresponding cleartext disclosure snippet array (containing the salt, key, and raw pseudonym value) that was generated during the issuance phase.
6. The Wallet transmits the unchanged, signed token along with the plaintext disclosure snippet back to the Relying Party via the OID4VP response interface.
7. The Relying Party validates the Issuer's cryptographic signature on the token envelope, runs the plaintext disclosure snippet through a $SHA-256$ hash function, verifies that the resulting digest matches one of the values inside the token's signed _sd array, and securely extracts the unique directed pseudonym.

### Expected Outcome
The Relying Party securely authenticates the user via a persistent, site-specific identifier without learning the user's real-world identity or master seed, preventing tracking across other relying parties.

# 7. Normative Requirements for use-case 2 (Directed Pseudonyms)

This section defines the normative requirements for implementations.

## 7.1 Issuance and Provisioning Component

The Pseudonym Service and PID Provider **MUST**:

1. Use the [OpenID for Verifiable Credential Issuance 1.0](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) (`OID4VCI`) protocol for secure credential provisioning and token exchange.
2. Structure all verifiable credential metadata in compliance with the [SD-JWT-based Verifiable Digital Credentials](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/) (`draft-ietf-oauth-sd-jwt-vc`) profiling specification.
3. Obfuscate the derived pseudonyms using salted digests as mandated by [RFC 9901: Selective Disclosure for JSON Web Tokens (SD-JWT)](https://datatracker.ietf.org/doc/html/rfc9901).

The Pseudonym Service and PID Provider **SHOULD**:

1. Utilize the OID4VCI flow H.5 protocol extension if the user-supplied seed method is selected for transmission.

The Pseudonym Service and PID Provider **MUST NOT**:

1. Transmit the raw, unhashed master pseudonym seed (`nym_seed`) within the public unencrypted token payload structures.



## 7.2 Presentation and Verification Component 

The Wallet and Relying Party (RP) **MUST**:

1. Utilize the [OpenID for Verifiable Presentations 1.0](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html) (`OID4VP`) transaction protocol to transmit identity tokens from the wallet container to the verifier.
2. Formulate and parse queries for selective attribute disclosure using the syntax defined in the [DIF Presentation Exchange 2.0](https://identity.foundation/presentation-exchange/spec/v2.0.0/) framework.
3. Validate the plaintext salts, disclosure arrays, and corresponding $SHA-256$ digests according to the validation rules of [RFC 9901: Selective Disclosure for JSON Web Tokens (SD-JWT)](https://datatracker.ietf.org/doc/html/rfc9901).

The Relying Party (RP) **MUST NOT**:

1. Refuse or block the processing of a validly presented directed pseudonym, unless explicit real-world identification of the natural person is legally mandated by Union or national law.

# 8. Interface Definitions

This section describes the technical interfaces used by the protocol.

Examples may include:

- HTTP endpoints
- wallet invocation URLs
- metadata endpoints
- credential request structures
- presentation responses
- trust registry queries

For each interface, describe:

- direction of communication
- transport method
- request parameters
- response structure

Example subsection:

## 8.1 <Interface Name>

*Direction:* <SENDER> → <RECEIVER>  
*Method:* <HTTP METHOD>

**Request**

- <FIELD 1>
- <FIELD 2>

**Response**

- <FIELD 1>
- <FIELD 2>

Example (illustrative only):

```text
<EXAMPLE REQUEST OR URL>
```
# 9. Conformance

An implementation **conforms to this specification** if it implements the requirements defined in this document for its role and supports the relevant interfaces and flows.

Where relevant, conformance may be stated separately for each role.

**Example**

An implementation conforms as a **<ROLE 1 CONFORMANCE CLASS>** if it:

1. implements the applicable requirements in Section 7  
2. supports the relevant interfaces and flows in Sections 6 and 8  
3. supports any required standards or formats referenced by this specification

Additional WE BUILD profiles may define stricter requirements for specific use cases. Such profiles **MUST NOT** weaken the mandatory requirements in this specification.

# References

[1] <ORGANISATION> (<YEAR>) <TITLE>. Available at: <URL> (Accessed: <DATE>).

[2] <ORGANISATION> (<YEAR>) <TITLE>. Available at: <URL> (Accessed: <DATE>).

[3] <ORGANISATION> (<YEAR>) <TITLE>. Available at: <URL> (Accessed: <DATE>).
