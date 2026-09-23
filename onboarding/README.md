# Onboarding for Testing

> [!NOTE]
> **Draft.** Open questions and actions are marked inline and should be resolved before this guide is finalised.
>
> **General open question:** Is the EDD being used within WE BUILD?

## Purpose

This guide describes what a participant needs to do to be onboarded for testing once their use case is ready. The starting point is always the participant's role in the ecosystem, since the role determines which registrations, certificates, conformance specifications and test cases apply.

As a general rule, it is important for all roles in the Consortium that you apply the CS (Conformance Specifications) and pass the ITB tests that are relevant to your role. 
The ITB tests protocol behaviour and profile conformance only, not business rules.

## Out-of-scope
CS-07 (DC API), CS-10 (Revocation Mechanism) and CS-017 Directory & Discovery, are out of scope for this on-boarding guide.

This guidance distinguishes between support aimed at roles for onboarding and support aimed at use cases. For use-case specific onboarding requirements, use-cases will provide you with further information. 

## ITB onboarding
## Testing in the ITB

Testing takes place in the WE BUILD Interoperability Test Bed (ITB): [ITB test bed](https://dev-i4mlab.aegean.gr/itb)

### Onboarding to the ITB

Please read the [ITB user guide](https://github.com/webuild-consortium/wp4-interop-test-bed/blob/main/docs/user-guide-interoperability-test-bed.md) for instructions on onboarding.

Provide the necessary information through the Slack channel [*#itb-support*](https://we-build-consortium.slack.com/archives/C09K65GLKT2), and the ITB team will set you up with an account.

### What you need to provide

- **Endpoints:** base URLs for dev, acc and prod.
- **Security mode:** bearer, mTLS or x-api-key.

### REST contract

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/request` | Returns an `openid4vp://` URI and a QR code |
| `GET` | `/poll` | Returns the verification result |
| `GET` | `/logs/{sessionId}` | *Optional.* Includes your logs in the test report |

- Every call carries the ITB `sessionId`.
- The ITB polls every 10 seconds and times out after 60 seconds.

### Environments

Use one ITB *System* per environment, so that your evidence stays reproducible.

### Evidence and conformance listing

1. Download the conformance statement report (PDF) from the ITB.
2. Upload it to the [Testing group files](https://portal.webuildconsortium.eu/group/testing/files).
3. Ask in *#itb-support* to be listed on the [Conformance Overview](https://webuild-consortium.github.io/wp4-interop-test-bed/docs/conformance-overview.html).

> **Note:** If you skip a test (for example mdoc), explain why in writing to the Test Group when you submit your report.



## Step 1: Identify your role

When your use case is ready for testing, first determine which role (or roles) you act in:

- Relying Party (RP) / verifier
- PID Provider (issuer)
- QEAA Provider (issuer)
- PuB-EAA Provider (issuer)
- Qualified Trust Service Provider (QTSP)
- Authentic Source
- Wallet Provider
- Intermediary
- Holder (economic operator or natural person)

For role descriptions, see:

- ARF 3.0.0, Chapter 3 – Roles within the EUDI Wallet ecosystem: <https://eudi.dev/3.0.0/main/03-roles-within-the-eudi-wallet-ecosystem>
- WE BUILD Architecture & Integration Blueprint (D4.1), Section 3.2 – role descriptions as applied in WE BUILD: <https://webuild-consortium.github.io/wp4-architecture/blueprint>

## Step 2: Use the role template

Each role will have an onboarding template that specifies:

- the conformance specifications (CS) the participant must conform to, and
- the ITB test cases the participant must pass (for wallets).



## Onboarding for Relying Parties

### 1. Register in the RP registry

The RP registers as a participant in the RP registry (for example, Raidiam). See the WP4 onboarding description: [relying-party-registry-onboarding.md](relying-party-registry-onboarding.md)

As part of registration, the RP:
- requests an RP access certificate, and
- requests an RP registration certificate.

> [!NOTE]
> The registration is done through filling out a form

### 2. Ensure functionality to verify presentations, i.e. conform to conformance specifications (CS)

The RP must be able to verify presentations. There are two routes:

- **Do it yourself** – the RP implements verification functionality itself.
- **Use an intermediary** – the RP relies on an intermediary and should contact that intermediary about onboarding. The verification component may be provided by:
  - a QTSP offering a verification component, or
  - a wallet solution that also offers verification functionality.
 
### 3. Relying Party (Verifier) Tests in the Interoperability Test Bed (ITB)

**Test 002 – Verifier VP 1.0**

In the Relying Parties suite, the ITB acts as the wallet. the ITB acts as the wallet. A headless reference wallet inside the ITB+ responds to your verifier's presentation request, so nobody needs to scan a QR code. The headless wallet must already hold the requested credential before you run this test (see Test 001 in the same suite).

**Good to know**
- In the ITB repo, Test 002 appears as `tc02` with the title "Issuer VP 1.0". It is the verifier test.
- Create a live VP request URI (openid4vp://) with your verifier and paste it into the ITB at step 0.
- The ITB sends the URI to the headless wallet (`POST /session`), which completes the presentation. The ITB then collects the wallet logs for the report.
-  A PASS shows that the headless wallet could complete a presentation to your verifier. It does not show that your verifier validates the response correctly (CS-02 §7.2).
-  If you only support SD-JWT VC, you can skip mdoc (deferred in CS-02, NOTE_CS02_01). State the reason when you submit your report.


> [!NOTE]
> There are currently no trust framework tests where an RP is the system under test. The trust framework test suite has to be updated for RPs. 
> The RP test is positive-path only.



**Legend:** 🟢 Obligatory · 🟡 Optional
SUT = System under test

| Applicable CS | Comment | ITB test suite / test case (RP as SUT) |
|---|---|---|
| 🟢 CS-02 Credential Presentation (v1.1, approved) | Applies to every RP: it defines the Verifier role in OpenID4VP presentation (§4, §7.2), including when the verifier function sits inside a multi-role product such as a Business Wallet (§7). | Test 002 – Verifier VP 1.0|
| 🟢 UPCOMING: CS-006 RP Access & Registration Certificates (in review, not published) | Will apply to every RP: it covers the access and registration certificates an RP needs in order to identify itself to wallets. | No dedicated test yet. TBD. |
| 🟡 CS-03 Remote QES – Wallet-Centric model only (v1.2, draft) | Applies to RPs that request a qualified electronic signature on a document from the Signer's wallet in the Wallet-Centric model (§4.1, §7.2). | **None for RP.** The QES suite ("CS03 – 1: Mock Signature") tests the wallet only. |
| 🟡 CS-12 SCA for Payments / TS12 (v1.0, pre-flight) | Applies to RPs that request SCA presentations to authenticate payment transactions: the ASPSP itself, or a third party such as a merchant, payee, PISP or TPP (§4, §7.4). | **None for RP.** The TS12 suite (9 tests) tests the wallet only. |



---

## Onboarding for EBWOID Issuers

### Step 1: Register on the EBWOID provider trust list

- **Precondition:** It is checked whether you are on the GA – just a form.
- You need to create keys to sign the trust list. *(Link to info – TBD)*

### Step 2: Ensure functionality to issue EBWOID

Relevant conformance specifications: CS-01, CS-02, CS-04, CS-05, CS-12 (payment)

You can issue yourself, or use a QTSP to issue for you. If a QTSP issues the EBWOID, the EBWOID issuer becomes an Authentic Source.

What is being verified: the EBWOID issuer is on the access certificate.

### Step 3: Conform to test cases in the ITB

- Trust framework integration test suite (only for wallets at the moment)
  - Issuers and verifiers will be added soon *(check with George for timeframe)*
- RP conformance test suite (for EBWOID issuers)
  - Test cases 1 and 3
- TS12 test suite (payment)

### Out of scope for this onboarding guide

- **Attestation catalogue:** Registering your attestations in the attestation catalogue (structure, whether rulebooks are followed, version). No CS and no test cases.
- **Revocation**

---

## Onboarding for Wallet Providers

### Step 1: Register on the trust list

### Step 2: Issue WIA/KA

Relevant conformance specifications:

- CS-05 (BW)
- CS-04 (EUDIW)

Corresponding ITB test cases:

- CS-05: no test suite at the moment
- CS-04: trust framework integration test suite (issuance from wallet perspective), test cases 1–8

### Step 3: Ensure functionality

Relevant conformance specifications: CS-01, CS-02, CS-04/CS-05, (CS-07/CS-10), (CS-03 Sealing/Signing – not a must)

Under development:

- CS-11 (remote QSeal creation)
- CS-13
- CS-07 (DC-API)
- CS-14 (proximity cases)
- CS-15 (QERDS)
- CS-12 SCA (for payment use cases)

> [!IMPORTANT]
> **Action – George:** Share slides mapping conformance specifications to ITB test cases.

---

## Onboarding for QTSPs

### Step 1: What kind of QTSP service do you offer?

#### QEAA Issuer

- **Registration:** Trust list for QEAA providers – simple form.
- **Functionality:** CS-01, (CS-08 IdM – under development), CS-04–CS-05 (for verification). For verification before issuance (Article 45e or 45f): CS-?? *(TBD)*
- **ITB test cases:** Same as for issuers.
