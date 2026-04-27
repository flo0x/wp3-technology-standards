# BU1 KYC_KYS MVP Workflow

1) MVP Restrictions:
## Seller/Holder Perspective
- Any person can trigger the process of customer onboarding
- The company is authorized to present attestations and receive attestations (no configuration support)
- Mutual authentication is set to default true (no TLOL or device-binding checks are applied).
- The MVP process is executed sequentially in one step.

## Buyer/RelyingParty Perspective 
- The seller will be classified as low/medium risk customer. It will be no high-risk customer  (therefore, e.g.: no sanction screening is required)

2) MVP+ Extension:
## Seller Perspective
- Additionally support for the KYC sanction validation

## Pre-requisites
1) This are the pre-requisites for the company in order to run the MVP 

```mermaid
sequenceDiagram
    participant Auth.Source 
    note right of Auth.Source : Bundesanzeiger, KVK, ...
    participant TAX_Administration 
    participant Seller
    participant Buyer

    Auth.Source ->> Seller : issue EBWOID  
    Auth.Source ->> Seller : issue EUCC
    Auth.Source ->> Buyer : issue EBWOID
    
    alt EAA Issuer available
        TAX_Administration ->> Seller : issue TAX
    else EAA attestation issuing
        Seller ->> Seller: issue TAX
    end
    
    Seller ->> Seller: issue VAT, CompanyInfo, ContactPerson
    Seller ->> Seller: issue OwnershipList,ControlList
```

2) This are the additionally pre-requisites for the company in order to run the MVP+
```mermaid
sequenceDiagram
    participant SocialSecurityIssuer
    SocialSecurityIssuer->> Seller: issue SocialSecurityAttestation 
    Auth.Source ->> Seller: issue TFS
```

### 1. Scenario KYC 

### 1.1. Legal Entity Selection
```mermaid
sequenceDiagram
    actor Initiator
    activate Initiator
    Initiator->>+Buyer_Portal: Select "start customer onboarding" Service
    alt Wallet_Support_EndPoint (ex. Resolvable eAddress or public endpoint URI)
        Buyer_Portal->>+Buyer_Portal : Provide an input field
        Initiator->>+Buyer_Portal: fill the address or end-point of the business wallet
        Buyer_Portal->>+Buyer_Portal: resolve eAddress
    else Support directly into EUBW  
        Note over Seller_Wallet: the supplier wallet already integrate the business process for specific buyer
        Initiator->>+Seller_Wallet: Select Buyer in the EUBW (configured in wallet)
    else Other: manual process (EUBW or EUDI Wallet)
        Note over Seller_Wallet: manuall proces by the Initiator
    end
    Person->>+Buyer_Portal: trigger process
```


### 1.2. LegalEntity Identification

```mermaid
sequenceDiagram
    actor Initiator
    Buyer_Portal<<->>Buyer_Wallet: generate proof-request
    Buyer_Portal<<->>Buyer_Wallet: for EBWOID, EUCC, TAX, VAT
    alt Automatically (EUBW support end-points)
        Buyer_Portal->>+Seller_Wallet: request presentations 
    else Manually ( EUBW or EUDI Wallet)
        Buyer_Portal->>Buyer_Portal: embed request into QRCode and provide an openid4vp-URI for the request
        Initiator->>+Seller_Wallet: copy/paste openid4vp-URI into the company wallet or scan the QRCode
    end
    Seller_Wallet<<->>Seller_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Seller_Wallet<<->>Seller_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Seller_Wallet->>Buyer_Portal: present the attestations
    Buyer_Portal<<->>Buyer_Wallet: verification of attestations rulebooks
```

### 1.3. KYC - Base Information
```mermaid
sequenceDiagram
    actor Initiator
    Buyer_Portal<<->>Buyer_Wallet: generate proof-request
    Buyer_Portal<<->>Buyer_Wallet: for CompanyInfo, ContactPerson
    alt Automatically (EUBW support end-points)
        Buyer_Portal->>+Seller_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        Buyer_Portal->>Buyer_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Seller_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Seller_Wallet<<->>Seller_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Seller_Wallet<<->>Seller_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Seller_Wallet->>Buyer_Portal: present the attestations
    Buyer_Portal<<->>Buyer_Wallet: verification of attestations (rulebook)
```

### 1.4. KYC - Customer Due Diligence  Information  
```mermaid
sequenceDiagram
    actor Initiator
    Buyer_Portal<<->>Buyer_Wallet: generate proof-request
    Buyer_Portal<<->>Buyer_Wallet: for OwnershipList,ControlList   
    alt Automatically (EUBW support end-points)
        Buyer_Portal->>+Seller_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        Buyer_Portal->>Buyer_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Seller_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Seller_Wallet<<->>Seller_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Seller_Wallet<<->>Seller_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Seller_Wallet->>Buyer_Portal: present the attestations
    Buyer_Portal<<->>Buyer_Wallet: verification of attestations (rulebook)
```

### 1.5. Additionally KYC Screening Information (this will be handled in the MVP+)
```mermaid
sequenceDiagram
    actor Initiator
    Buyer_Portal<<->>Buyer_Wallet: generate proof-request
    Buyer_Portal<<->>Buyer_Wallet: for TFS - Attestation and SocialSecurityAttestation 
    alt Automatically (EUBW support end-points)
        Buyer_Portal->>+Seller_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        Buyer_Portal->>Buyer_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Seller_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Seller_Wallet<<->>Seller_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Seller_Wallet<<->>Seller_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Seller_Wallet->>Buyer_Portal: present the attestations
    Buyer_Portal<<->>Buyer_Wallet: verification of attestations (rulebook)
```

### 1.6. Cross-Check
```mermaid
sequenceDiagram
    actor ContactPerson
    Buyer_Portal<<->>Buyer_Portal: cross check over all attestations (to be defined exactly what will be checked)
    Buyer_Portal->>+RP_InternalSystem: transfer data to internal system
    
    Buyer_Portal->>ContactPerson: Send notification to the contact person that onboarding was successful.
    Buyer_Portal<<->>Buyer_Portal: Display success notification for initiator
```
