# BU1 KYC_KYS MVP Workflow

MVP Restrictions:
## Buyer Perspective (Holder)
- The person who initiated the customer onboarding is the contact person.
- Mutual authentication is set to default true (no TLOL or device-binding checks are applied).
- The buyer wallet is authorized to present attestations and receive attestations (no configuration support)
- The MVP process is executed sequentially in one step.

## Seller Perspective (RelyingParty) 
- The buyer will be classified as low/medium risk customer. It will be no high-risk customer  (therefore, e.g.: no sanction screening is required)

MVP+ Extension:
## Seller Perspective
- Additionally support for the KYC sanction validation

## Pre-requisites
1) This are the pre-requisites for the company in order to run the MVP 

```mermaid
sequenceDiagram
    participant Auth.Source 
    note right of Auth.Source : Bundesanzeiger, KVK, ...
    participant TAX_Administration 
    participant Buyer
    participant Seller

    Auth.Source ->> Buyer : issue EBWOID,EUCC
    Auth.Source ->> Seller : issue EBWOID
    
    alt EAA Issuer available
        TAX_Administration ->> Buyer : issue TAX
    else EAA attestation issuing
        Buyer ->> Buyer: issue TAX
    end
  
    Buyer ->> Buyer: issue VAT, CompanyInfo, ContactPerson
    Buyer ->> Buyer: issue OwnershipList,ControlList
```

2) This are the additionally pre-requisites for the company in order to run the MVP+
```mermaid
sequenceDiagram
    participant SocialSecurityIssuer
    SocialSecurityIssuer->> Buyer: issue SocialSecurityAttestation 
    Auth.Source ->> Buyer: issue TFS
```

### 1. Scenario KYC 

### 1.1. Legal Entity Selection
```mermaid
sequenceDiagram
    actor Initiator
    activate Initiator
    Initiator->>+Seller_Portal: Select "start customer onboarding" Service
    alt Wallet_Support_EndPoint (ex. Resolvable eAddress or public endpoint URI)
        Seller_Portal->>+Seller_Portal : Provide an input field
        Initiator->>+Seller_Portal: fill the address or end-point of the business wallet
        Seller_Portal->>+Seller_Portal: resolve eAddress
    else Support directly into EUBW  
        Note over Buyer_Wallet: the buyer wallet already integrate the business process for specific seller 
        Initiator->>+Buyer_Wallet: Select seller in the EUBW (configured in wallet)
    else Other: manual process (EUBW or EUDI Wallet)
        Note over Buyer_Wallet: manuall proces by the Initiator
    end
    Initiator->>+Seller_Portal: trigger process
```


### 1.2. LegalEntity Base Identification

```mermaid
sequenceDiagram
    actor Initiator
    Seller_Portal<<->>Seller_Wallet: generate proof-request
    Seller_Portal<<->>Seller_Wallet: for EBWOID, EUCC, TAX, VAT, CompanyInfo, ContactPerson
    alt Automatically (EUBW support end-points)
        Seller_Portal->>+Buyer_Wallet: request presentations 
    else Manually ( EUBW or EUDI Wallet)
        Seller_Portal->>Seller_Portal: embed request into QRCode and provide an openid4vp-URI for the request
        Initiator->>+Buyer_Wallet: copy/paste openid4vp-URI into the company wallet or scan the QRCode
    end
    Buyer_Wallet<<->>Buyer_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Buyer_Wallet<<->>Buyer_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Buyer_Wallet->>Seller_Portal: present the attestations
    Seller_Portal<<->>Seller_Wallet: verification of attestations rulebooks
```

### 1.3. KYC - Customer Due Diligence  Information
```mermaid
sequenceDiagram
    actor Initiator
    Seller_Portal<<->>Seller_Wallet: generate proof-request
    Seller_Portal<<->>Seller_Wallet: for OwnershipList,ControlList -> only the relevant KYS attributes ( GDPR conform)  
    alt Automatically (EUBW support end-points)
        Seller_Portal->>+Buyer_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        Seller_Portal->>Seller_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Buyer_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Buyer_Wallet<<->>Supplier_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Buyer_Wallet<<->>Supplier_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Buyer_Wallet->>Seller_Portal: present the attestations
    Seller_Portal<<->>Seller_Wallet: verification of attestations (rulebook)
```

### 1.4. KYC-Screening and additionally Information (this will be handled in the MVP+)
```mermaid
sequenceDiagram
    actor Initiator
    Seller_Portal<<->>Seller_Wallet: generate proof-request
    Seller_Portal<<->>Seller_Wallet: for TFS - Attestation, SocialSecurityAttestation 
    alt Automatically (EUBW support end-points)
        Seller_Portal->>+Buyer1_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        Seller_Portal->>Seller_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Buyer1_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Buyer1_Wallet<<->>Buyer1_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Buyer1_Wallet<<->>Buyer1_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Buyer1_Wallet->>Seller_Portal: present the attestations
    Seller_Portal<<->>Seller_Wallet: verification of attestations (rulebook)
```

### 1.5. Cross-Check
```mermaid
sequenceDiagram
    actor ContactPerson
    Seller_Portal<<->>Seller_Portal: cross check over all attestations (to be defined exactly what will be checked)
    Seller_Portal->>+RP_InternalSystem: transfer data to internal system
    
    Seller_Portal->>ContactPerson: Send notification to the contact person that onboarding was successful.
    Seller_Portal<<->>Seller_Portal: Display success notification for initiator
```
