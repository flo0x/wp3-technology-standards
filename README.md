# WP3 - Technology and Standards (WE BUILD Consortium)

The **WP3 Technology and Standards Group** is responsible for defining the common standards for the **WE BUILD** consortium, which is piloting the **EU Digital Identity Wallet (EUDIW)**. This work package focuses on the **banking and payments** sectors, covering both consumer and corporate segments.

The goal of this repository is to provide a central place for technical specifications, sequence diagrams, and implementation examples that enable interoperable use of the EUDI Wallet in WP3 usecases and scenarios.

## Directory Structure

```text
.
├── PA1/                                # UseCase PA1 (Consumer banking)
│   ├── Scenario 4/                     # Specific issuance scenario 4
│   ├── examples/                       # Issuance examples
│   └── issuance/                       # Sequence diagrams for issuance
├── PA2/                                # Pilot Application 2 (Presentation)
│   ├── PA2 A_1/                        # Presentation Flows
│   │   └── sequence/
│   └── PA2 A_2/                        # Extended Presentation (SUA)
│       ├── attestations/               # SUA and other attestation examples
│       ├── examples/
│       │   ├── DC_API_examples/        # API request examples (JSON)
│       │   └── Deeplinks/
│       └── sequence/
└── general/                            # Common assets and examples
    ├── examples/
    │   ├── attestation/                # Example SD-JWT and W3C VC attestations
    │   │   ├── IBAN Confirmation/
    │   │   ├── IBAN SUA/
    │   │   └── SUA/
    │   └── metadata/                   # Metadata examples (Issuer/Verifier)
    └── sequence/                       # Shared sequence diagrams
```

## Folder Descriptions

### [PA1/](./PA1) - Issuance
Focuses on the **Credential Issuance** phase within the EUDIW ecosystem, specifically for banking-related credentials.
- `Scenario 4/`: Specific issuance scenario documentation.
- `issuance/`: PlantUML diagrams for various issuance business flows:
  - **Authorized Code Flow**: Standard OID4VCI issuance.
  - **Pre-authorized Code Flow**: Simplified issuance for known users.
  - **Credential Refresh**: Mechanisms for updating existing credentials.
  - **General Issuance Business Flow**: End-to-end overview of the issuance process.

### [PA2/](./PA2) - Presentation
Focuses on the **Presentation** phase and the interaction between the EUDI Wallet and Relying Parties (RPs) or Data Providers.
- `PA2 A_1/`: Technical diagrams for **Digital Credentials (DC) API** and **Deep-link** based presentation flows.
- `PA2 A_2/`: 
  - `examples/DC_API_examples/`: JSON examples for signed and unsigned credential requests.
  - `attestations/`: Examples of **Signed User Allegations (SUA)** for IBAN and other financial attributes.
  - `sequence/`: Diagrams for proximity-based interactions (e.g., NFC/BLE) and API flows.

### [general/](./general)
Contains common assets, shared diagrams, and reusable implementation examples.
- `examples/`:
  - `attestation/`: Example attestations in **SD-JWT** and **W3C VC** formats (e.g., IBAN attestations).
  - `metadata/`: Metadata examples for Issuers and Verifiers. These include configurations for IBAN and SUA, with support for various transaction data types (PSD2, Berlin Group, etc.).
- `sequence/`: General presentation and DC API sequence diagrams.


## Key Technologies & Standards & abbrevations

- **OpenID4VCI / OpenID4VP**: Protocols used for issuance and presentation.
- **SD-JWT (Selective Disclosure JWT)**: see [SD-JWT specification](https://www.ietf.org/archive/id/draft-ietf-oauth-sd-jwt-vc-14.html)
- **W3C Verifiable Credentials**: W3C data model for credentials.
- **SUA (String User Authentication)**: see [SUA specification](https://github.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/blob/main/docs/technical-specifications/ts12-electronic-payments-SCA-implementation-with-wallet.md)
  
---
*This repository is part of the WE BUILD consortium's effort to pilot EUDIW in the European banking landscape.*