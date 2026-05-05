# BU1 KYS MVP Workflow

MVP Restrictions:
## Supplier Perspective (Holder)
- The person who initiated the customer onboarding is the contact person.
- The Mutual authentication is set to default true (no TLOL or device-binding checks are applied).
- The supplier wallet  is authorized to present attestations and receive attestations (no configuration support)
- The MVP process is executed sequentially in one step.

## Customer Perspective (RelyingParty) 
- The supplier will be classified as low/medium risk supplier. It will be no high-risk supplier  (therefore, e.g.: no sanction screening is required)

MVP+ Extension:
## Supplier perspective
- Additionally support for the KYS sanction validation 
- Additionally support for the ESG Certificates 

## Pre-requisites
This are the pre-requisites for the company in order to run the MVP 

```mermaid
sequenceDiagram
    participant Auth.Source 
    note right of Auth.Source : Bundesanzeiger, KVK, ...
    participant TAX_Administration 
    participant Supplier
    participant Customer 
    participant Bank
    
    Auth.Source ->> Supplier : issue EBWOID, EUCC 
    Auth.Source ->> Customer : issue EBWOID
    
    alt PubEAA Issuer available
        TAX_Administration ->> Supplier : issue TAX
    else EAA attestation issuing
        Supplier ->> Supplier: issue TAX
    end
    Supplier ->> Supplier: issue VAT, CompanyInfo,  ContactPerson
    Supplier ->> Supplier: issue OwnershipList,ControlList    
    
    alt Supplier has LEI Number 
        LEI ->> Supplier: issue LEI
    else Supplier has GLN Number
        GS1 ->> Supplier: issue GLN
    else Supplier has DUNS Number        
        Supplier ->> Supplier: issue DUNS
    end 
    
    alt Supplier has a Site 
        Supplier ->> Supplier : issue SiteAttestation
    end
    
    Supplier ->> Supplier: issue PaymentTerms  
    Bank ->> Supplier: issue IBAN 
    
```

This are the additionally pre-requisites for the company in order to run the MVP+
```mermaid
sequenceDiagram
    Auth.Source ->> Supplier: issue TFS
    Supplier ->> Supplier: issue ESG (as EAA, QEAA - in clarification)    
```
### 1. Scenario KYC 

### 1.1. Legal Entity Selection
```mermaid
sequenceDiagram
    actor Initiator
    activate Initiator
    Initiator->>+RP_Portal: Select "start supplier onboarding" Service
    alt Wallet_Support_EndPoint (ex. Resolvable eAddress or public endpoint URI)
        RP_Portal->>+RP_Portal : Provide an input field
        Initiator->>+RP_Portal: fill the address or end-point of the business wallet
        RP_Portal->>+RP_Portal: resolve eAddress
    else Support directly into EUBW  
        Note over Supplier_Wallet: the company wallet already integrate the business process of specific relying party
        Initiator->>+Supplier_Wallet: Select relying party in the EUBW (configured in wallet)
    else Other: manual process (EUBW or EUDI Wallet)
        Note over Supplier_Wallet: manuall proces by the Initiator
    end
    Initiator->>+RP_Portal: trigger process
```

### 1.2. LegalEntity-Base Identification

```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal->>RP_Wallet: generate proof-request
    RP_Portal->>RP_Wallet: for EBWOID, EUCC,TAX, VAT, CompanyInfo, ContactPerson
    alt Automatically (EUBW support end-points)
        RP_Wallet->>+Supplier_Wallet: request presentations 
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI for the request
        Initiator->>+Supplier_Wallet: copy/paste openid4vp-URI into the company wallet or scan the QRCode
    end
    Supplier_Wallet->>RP_Wallet: request mutual authentification ( x509 certificate or eubwoid rulebook)
    RP_Wallet->>Supplier_Wallet: present mutual authentification ( x509 certificate or eubwoid rulebook)
    Supplier_Wallet<<->>Supplier_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Supplier_Wallet->>RP_Wallet: present the attestations
    RP_Wallet->>RP_Portal: sends sucess of verification of conformity regarding issuers, rulebooks and vadility 
 
```
RP_Portal represents in this case companies web portal and internal ERP System. 

### 1.3.1 KYS - Supplier Due Diligence  Information
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal->>RP_Wallet: generate proof-request
    RP_Portal->>RP_Wallet: for OwnershipList,ControlList -> only the relevant KYS attributes ( GDPR conform)  
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Supplier_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Supplier_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Supplier_Wallet->>RP_Wallet: request mutual authentification ( x509 certificate or eubwoid rulebook)
    RP_Wallet->>Supplier_Wallet: present mutual authentification ( x509 certificate or eubwoid rulebook)
    Supplier_Wallet<<->>Supplier_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Supplier_Wallet->>RP_Wallet: present the attestations
    RP_Wallet->>RP_Portal: sends sucess of verification of conformity regarding issuers, rulebooks and vadility 
```
RP_Portal represents in this case companies web portal and internal ERP System. 

### 1.3.2. KYS - Additionally identifier Information
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>RP_Wallet: generate proof-request
    RP_Portal<<->>RP_Wallet: for DUNS or  GS1 or  LEI 
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Supplier_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Supplier_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Supplier_Wallet->>RP_Wallet: request mutual authentification ( x509 certificate or eubwoid rulebook)
    RP_Wallet->>Supplier_Wallet: present mutual authentification ( x509 certificate or eubwoid rulebook)
    Supplier_Wallet<<->>Supplier_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Supplier_Wallet->>RP_Wallet: present the attestations
    RP_Wallet->>RP_Portal: sends sucess of verification of conformity regarding issuers, rulebooks and vadility 
```
RP_Portal represents in this case companies web portal and internal ERP System. 

### 1.3.3. KYS - Payment and other additionally Information
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>RP_Wallet: generate proof-request
    RP_Portal<<->>RP_Wallet: for PaymentTerms, IBAN, (SiteAttestation)  
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Supplier_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Supplier_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
     Supplier_Wallet->>RP_Wallet: request mutual authentification ( x509 certificate or eubwoid rulebook)
    RP_Wallet->>Supplier_Wallet: present mutual authentification ( x509 certificate or eubwoid rulebook)
    Supplier_Wallet<<->>Supplier_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Supplier_Wallet->>RP_Wallet: present the attestations
    RP_Wallet->>RP_Portal: sends sucess of verification of conformity regarding issuers, rulebooks and vadility 
```
RP_Portal represents in this case companies web portal and internal ERP System. 

### 1.3.4. KYS-Screening and additionally Information (this will be handled in the MVP+)
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>RP_Wallet: generate proof-request
    RP_Portal<<->>RP_Wallet: for TFS, ESG 
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Supplier_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Supplier_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Supplier_Wallet->>RP_Wallet: request mutual authentification ( x509 certificate or eubwoid rulebook)
    RP_Wallet->>Supplier_Wallet: present mutual authentification ( x509 certificate or eubwoid rulebook)
    Supplier_Wallet<<->>Supplier_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Supplier_Wallet->>RP_Wallet: present the attestations
    RP_Wallet->>RP_Portal: sends sucess of verification of conformity regarding issuers, rulebooks and vadility 
```
RP_Portal represents in this case companies web portal and internal ERP System. 

### 1.4. Cross-Check  
```mermaid
sequenceDiagram
    actor ContactPerson
    RP_Portal<<->>RP_Portal: cross check over all attestations (to be defined exactly what will be checked)
    RP_Portal->>+RP_InternalSystem: transfer data to internal system

    RP_Portal->>ContactPerson: Send notification to the contact person that onboarding was successful.
    RP_Portal<<->>RP_Portal: Display success notification for initiator
```
