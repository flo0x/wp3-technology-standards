# PA4 MVP Workflow

MVP Restrictions:

## Seller Perspective (Holder)
- The seller has an active European Business Wallet (EUBW) containing only the minimum set of attestations required to execute PA4.
- The seller is not required to complete onboarding with the Home Bank (PA3) or onboarding into the KYS process (BU1 – KYS).

## Buyer Perspective (Relying Party)
- The buyer has an active European Business Wallet (EUBW) containing only the minimum set of attestations required to execute PA4.
- The company is not required to complete onboarding with the Home Bank (PA3) or onboarding into the KYC process (BU1 – KYC).
- the validation of the IBAN  is out of scope (since the KYS process was escaped) 
- It is assumed that the eInvoice has already been received in the Business Wallet. 
- The delivery channel is out of scope for the MVP (part of the SC5)
- Sellers are assumed to be classified only as low- or medium-risk sellers.
- High-risk seller scenarios are out of scope for the MVP.
- The company’s Business Wallet contains a valid Mandate Verifiable Presentation (VP) listing the authorised signatories (SR), as well as the corresponding Power of Attorney (PoA) for the acting person.
- The direct exchange scenario is covered by SC5.
- For the MVP, the onboarding process is executed sequentially as a single-user flow.
- The process ends with the issuance of the eReceipt VC into the Business Wallet, bound to the legal entity.

## Person Perspective
- The natural person acting on behalf of the legal entity holds a valid personal EUDI Wallet containing a PID credential.
- The person is listed as an authorised representative of the company for interactions with the financial sector (SR).
- The person also holds a valid Power of Attorney (PoA) specifying the permitted scope of authority (e.g. transactions without limitation).
- It is assumed that the person holds the SCA attestation in the wallet ( PA1 use case).
- Validation of the company’s authorised representative lists (SR) is out of scope for the MVP.
- The person performs Strong Customer Authentication (SCA) using possession of the wallet device combined with either a PIN or biometric authentication.
- In the case of IBAN-based transactions, the IBAN information provided in the eInvoice is assumed to be trusted. 
- High-risk transaction scenarios are excluded from the MVP scope and may be addressed in a future MVP+ phase.

## Assumptions – Holder Business Wallet
a.Automatic Flow
- Mutual authentication is enabled by default; no TLOL or device-binding checks are applied.
- The buyer wallet is pre-authorized to present and receive attestations; no additional configuration is supported in the MVP.
- The Business Wallet accepts credential offers automatically when an active corporate authorisation session is already established.

b. Manual Flow
- Mutual authentication is reduced to a visual verification step.
- Authorization is reduced to a visual check and manual approval process.

## Assumptions – Relying Party Business Wallet
- Attestation verification for each attestation is limited to:
  a. Cryptographic validation (Section 4.2.1)
  b. Basic issuer identification against an internal trusted list (Sections 4.2.2–4.2.3)

## General 
- It is assumed that the eInvoice has already been received in the Business Wallet.
- Cross-border cases is not part of the MVP.
- The process ends with the issuance of the eReceipt VC into the Business Wallet, bound to the legal entity.


## Pre-requisites
Pre-requisites : These are the pre-requisites for the Seller and  Buyer in order to run the MVP.

1.1 Seller Pre-requisites

```mermaid
sequenceDiagram
participant Auth_Source
participant Seller
participant Invoice_Processor

    Auth_Source ->> Seller : issue EUBWOID
    Seller <<->> Invoice_Network : connnect to invoice processor ( internal system or or equivalent e-invoicing infrastructure)
```

1.2 Buyer Pre-requisites

```mermaid
sequenceDiagram
participant Issuing_Authority
participant EUDI_Wallet
participant Auth_Source
participant Buyer
participant Invoice_Processor
participant Payment_Processor

    Issuing_Authority ->> EUDI_Wallet : issue Personal Identification Data (PID)
    Auth_Source ->> Buyer : issue EUBWOID
    Buyer <<->> Bank : the Bank Wallet is defined for the buyer      
    Buyer <<->> Invoice_Processor : Connnect to invoice processoer ( internal system or equivalent e-invoicing infrastructure)
    Buyer <<->> Payment_Processor : Connnect to payment processoer ( internal system or PSP - Provider)
```

### 1. Scenario 1 

The following diagram provides an overview of the end-to-end process

```mermaid
sequenceDiagram
participant Seller as Seller
actor Initiator
participant EUDI_Wallet  as EUDI_Wallet
participant Buyer  as Buyer
participant Bank as Bank

    Seller ->> Buyer : The buyer company receives an electronic invoice (eInvoice) in its Business Wallet (direct or indirect)
    %% ──────────────────────────────────────────────
    %% STEP 1 – Legal Entity Selection
    %% ──────────────────────────────────────────────
    note over Seller, Bank  : Step 1  – Legal Entity Selection
    Initiator ->>+ Bank : Select "Start Payment process" service
    Initiator ->>+ Bank : Select the "legal entity" discovery mechanism 
 
    %% ──────────────────────────────────────────────
    %% STEP 2 – Payment Initiation 
    %% ──────────────────────────────────────────────
    note over Seller, Bank  : Step 2  – Payment Initiation
    alt Choice of "payment by card"
        Initiator <<->> Bank : initiate the payment 
    else Choice of "payment via bank transfer(IBAN)"
        Initiator <<->> Buyer : initiate the payment IBAN transfer.
        Buyer <<->> Bank : trigger payment filling with token 
    end
    
    %% ──────────────────────────────────────────────
    %% STEP 3 – Payment Authorization  
    %% ──────────────────────────────────────────────
    note over Seller, Bank  : Step 3 – Payment Authorization
    alt Choice of "card payment"
        Bank <<->> Bank: request the SCA
        Initiator <<->> EUDI_Wallet : present the SCA
    else Choice of "IBAN payment - standard"
        Bank <<->> Bank  : token validity check    
    end

    %% ──────────────────────────────────────────────
    %% STEP 4 –  Legal Entity - Payment Authorization 
    %% ──────────────────────────────────────────────
    note over Seller, Bank  : Step 4 – Legal Entity Payment Authorization
    alt Choice of "card payment"
        Bank <<->> Buyer : check initiator authorization (SignatoryRights, PoA (scope: card, limit: 0 ) )   
    else Choice of "IBAN payment - high tx"
        Bank <<->> Buyer : check initiator authorization (SignatoryRights, PoA (scope : tx, limit : 1000K  )
    else Choice of "IBAN payment - standard"
        Bank <<->> Buyer : check the token authorization
    end

    %% ──────────────────────────────────────────────
    %% STEP 5 – Payment Settlement  
    %% ──────────────────────────────────────────────
    note over Seller, Bank  : Step 5 – Payment 
    Bank <<->> Bank : Payment settlement  
    Bank <<->> Buyer: payment or settlement confirmation  

    %% ──────────────────────────────────────────────
    %% STEP 6 – eReceipt
    %% ──────────────────────────────────────────────
    note over Seller, Buyer  : Step 6 – eReceipt issuing 
    Buyer <<->> Seller : eReceipt Issuing (direct or indirect)   
```

