
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

This CS only includes identity matching for new users. Identity matching of persons in existing records is out of scope.
Althoug there are many more potential solutions for identity matching, these three have been chosen for their coverage of use-cases and degrees of implementation complexity.

# 3. Normative Language

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These keywords indicate the strength of requirements for conforming implementations.

# 4. Roles and Components

This section identifies the roles and components relevant to this specification.

Only roles that matter for this specification should be included.

Examples may include:

- **PID-Issuer:** entity issuing the PID and supplying a seed to enable reidentification
- **Wallet-provider:** entity providing the EUDIW and pseudonym service
- **Verifier / Relying Party:** entity requesting and validating presentations
- **Issuer of Photo-ID:** entity issuing an official Photo-ID
- **Trust Provider:** component or service publishing trust-related information
- **Pseudonym Service:** service that generates the directed pseudonym

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



It should help the reader understand:

- which standards or mechanisms are used
- how the main actors interact
- which security or trust features are important
- what the overall outcome of the interaction is

This section should stay concise. Detailed behaviour belongs in later sections.

# 6. High-level Flows

This section describes the main interaction flows between actors.

Flows should be written as step-by-step sequences that help implementers understand how the protocol operates.

Example subsections:

## 6.1 <Flow Name>

Describe:

- participating actors
- how the interaction begins
- the main sequence of actions
- the expected outcome

Example:

1. <STEP 1>
2. <STEP 2>
3. <STEP 3>

# 7. Normative Requirements

This section defines the normative requirements for implementations.

Requirements may be grouped in the way that best fits the specification, for example by:

- role
- component
- capability
- protocol step

Example structure:

## 7.1 <Role or Component>

<ROLE OR COMPONENT> **MUST**:

1. <REQUIREMENT 1>
2. <REQUIREMENT 2>

<ROLE OR COMPONENT> **SHOULD**:

1. <RECOMMENDATION 1>

<ROLE OR COMPONENT> **MUST NOT**:

1. <PROHIBITED BEHAVIOUR>

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
