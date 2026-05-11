# PA4 MVP Workflow

MVP Restrictions:

## Seller Perspective (Holder)
- The seller has an active European Business Wallet (EUBW) containing only the minimum set of attestations required to execute PA4.
- The seller is not required to complete onboarding with the Home Bank (PA3) or onboarding into the KYS process (BU1 – KYS).

## Buyer Perspective (Relying Party)
- The buyer holds an active European Business Wallet (EUBW) containing only the minimum set of attestations required to execute PA4.
- The buyer is not required to complete onboarding with the Home Bank (PA3) or undergo onboarding into the KYC process (BU1 – KYC).
- The onboarding of the acting person representing the buyer company within the Home Bank is performed as part of the end-customer onboarding process, rather than as a separate business representative onboarding process (covered under PA2).
- Consequently, the buyer’s Business Wallet is assumed to contain valid Authorised Signatory Rights (SR) attestations, together with the corresponding Power of Attorney (PoA) for the acting person.
- In the case of IBAN-based transactions, the IBAN information provided in the eInvoice is assumed to be trusted. 
- The IBAN validation is considered out of scope for the MVP, as the KYS/KYC process is excluded.
- The invoice delivery channel is out of scope for the MVP and covered under SC5.
- Sellers are assumed to be classified exclusively as low- or medium-risk sellers.

## Person Perspective
- The natural person acting on behalf of the legal entity holds a valid EUDI Wallet containing a PID credential.
- It is assumed that the person holds the SCA attestation in the wallet ( PA2 use case). 
- Validation of the company’s authorised representative lists (SR) is out of scope for the MVP.
- The person performs Strong Customer Authentication (SCA) using possession of the wallet device combined with either a PIN or biometric authentication.

## Assumptions – Holder Business Wallet
- Automatic Flow
  - Mutual authentication is enabled by default; no TLOL or device-binding checks are applied.
  - The buyer wallet is pre-authorized to both present and receive attestations; no additional configuration is supported in the MVP.
  - The Business Wallet accepts credential offers automatically when an active corporate authorisation session is already established.

- Manual Flow
  - Mutual authentication is reduced to a visual verification step.
  - Authorization is reduced to a visual check and manual approval process.

## Assumptions – Relying Party Business Wallet
- Attestation verification for each attestation is limited to:
  - Cryptographic validation (Section 4.2.1)
  - Issuer identification against an internal trusted list (Sections 4.2.2-4.3.3)

## General UseCase assumption  
- the payment process is executed sequentially in a single-user flow.
- It is assumed that the eInvoice has already been received in the Business Wallet.
- Cross-border cases is not part of the MVP.
- The process concludes with the issuance of the eReceipt Attestation into the Business Wallet.
- No attestation is issued to the acting person’s EUDI Wallet.


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

### 1. Scenario 

The following diagram provides an overview of the end-to-end process

```mermaid
sequenceDiagram
participant Seller as Seller
actor Initiator
participant EUDI_Wallet  as EUDI_Wallet
participant Buyer  as Buyer
participant Bank as Bank
participant PSP  as PSP

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
    note over Seller, PSP  : Step 2  – Payment Initiation
    alt Choice of "payment by card"
        Initiator <<->> EUDI_Wallet : scan the code 
        EUDI_Wallet <<->> PSP: initiate the payment 
    else Choice of "payment via bank transfer(IBAN)"
        Initiator <<->> Buyer : initiate the payment IBAN transfer.
        Buyer <<->> Bank : trigger payment filling with token 
    end
    
    %% ──────────────────────────────────────────────
    %% STEP 3 – Payment Authorization  
    %% ──────────────────────────────────────────────
    note over Seller, Bank  : Step 3 – Payment Authorization
    alt Choice of "card payment"
        PSP <<->> EUDI_Wallet: request the SCA
        Initiator <<->> EUDI_Wallet : present the SCA
    else Choice of "IBAN payment - standard"
        Bank <<->> Bank  : token validity check    
    end

    %% ──────────────────────────────────────────────
    %% STEP 4 –  Legal Entity - Payment Authorization 
    %% ──────────────────────────────────────────────
    note over Seller,  PSP  : Step 4 – Legal Entity Payment Authorization
    alt Choice of "card payment"
        PSP <<->> Buyer : check initiator authorization  (PoA (scope: card, limit:  x) ) 
        PSP <<->> Bank : initiate the settlement  
    else Choice of "IBAN payment - high tx"
        Bank <<->> Buyer : check initiator authorization (SignatoryRights, PoA (scope : tx, limit : x  )
    else Choice of "IBAN payment - standard"
        Bank <<->> Buyer : check the token authorization and initiate the settlement
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

