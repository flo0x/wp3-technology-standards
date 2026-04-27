# PA3 MVP Workflow

MVP Restrictions:
## Company Perspective
- The person who initiated the account opening process is also: legal representative, UBO and contact person (to make it simple: 4 positions in 1; one-person GmbH in Germany, covering SME-cases)”
- The initiator is the sole legal representative, who is also the UBO and contact person for the bank relationship (one-person GmbH in Germany, covering SME-cases).
- Means: No additional persons with signatory rights are required
- The company is not a branch.

## Bank perspective
- The company who wants to open a bank account will be classified as low/medium risk client. It will be no high-risk client (therefore, e.g.: no sanction screening, PEP-screening etc. is required)
- The bank supports cross-border account opening and no registration in the national register for the initiator/account holder is required.
Info: If other banks participate in the MVP, this can be extended to include the POR attestation.

## Following assumptions for the company business wallet
- The company is authorized to present attestations and receive attestations (no configuration support)
- Mutual authentication is set to default true (no TLOL or device-binding checks are applied).

## MVP Flow
- The MVP process (Scenario 1-3) is executed sequentially by one person
- The process ending with the triggering and issuance of the IBAN-OV issuance/attestation as EAA (means no additional onboarding is required)
- UBO calculation is an internal process and is not part of the MVP or MVP+. 
- Discrepancy reporting to the transparency register is a separate process and is not part of the MVP or MVP+.

## Pre-requisites
This are the pre-requisites for the company and bank in order to run the MVP.

```mermaid
sequenceDiagram
    actor Person
    participant Issuing.Authority 
    participant EUDI_Wallet 
    participant Auth.Source 
    note right of Auth.Source : Bundesanzeiger, KVK, ...
    participant TransparencyRegister 
    participant TAX_Administration 
    participant Company
    participant Bank
    activate Person    

    Issuing.Authority ->> EUDI_Wallet : issuer PID
    
    Auth.Source ->> Company : issue EBWOID  
    Auth.Source ->> Company : issue EUCC
    Auth.Source ->> Bank : issue EBWOID
    alt PubEAA Issuer available
        TAX_Administration ->> Company : issue TAX
    else EAA attestation issuing
        Company ->> Company: issue TAX
    end
    Company ->> Company: issue VAT, CompanyInfo, ContactPerson, SignatoryRights
    Company ->> Company: issue OwnerList, CntrolList, UBOList
    Company ->> TransparencyRegister: submit the UBOList
    Auth.Source ->> Bank : issue EBWOID
```

### 1. Scenario 1 

### 1.1. Legal Entity Selection
```mermaid
sequenceDiagram
    actor Initiator
    activate Initiator
    Person->>+Bank_Portal: Select "open business account" Service
    alt Wallet_Support_EndPoint (ex. Resolvable eAddress or public endpoint URI)
        Bank_Portal->>+Bank_Portal : Provide an input field
        Initiator->>+Bank_Portal: fill the address or end-point of the business wallet
        Bank_Portal->>+Bank_Portal: resolve eAddress
    else Support directly into EUBW  
        Note over Company_Wallet: the company wallet already integrate the business process of specific banks
        Initiator->>+Company_Wallet: Select bank in the EUBW (configured in wallet)
    else Other: manual process (EUBW or EUDI Wallet)
        Note over Company_Wallet: manuall proces by the Initiator
    end
    Person->>+Bank_Portal: trigger process
```

### 1.2. Initiator Identification 
```mermaid
sequenceDiagram
    actor Initiator
    participant EUDIWallet
    alt PID - Identification
        Bank_Portal<<->>Bank_Wallet: generate request to identity Initiator (PID) and embed into QRCode
        Initiator->>+EUDIWallet : scans QR Code with personal wallet
        EUDIWallet->>+EUDIWallet : mutual authentification ( auth. certificate )
        Initiator->>+EUDIWallet : check the authorization for presentation (visual check)
        EUDIWallet->Bank_Portal : send the pid information
        Bank_Portal<<->>Bank_Wallet: verification of PID  (rulebook)
    else Other identification means
        Note over EUDIWallet: identification of the person with other identfication means (ex. eID)
    end
```

### 1.3. LegalEntity-Base Identification

```mermaid
sequenceDiagram
    actor Initiator
    Bank_Portal<<->>Bank_Wallet: generate proof-request
    Bank_Portal<<->>Bank_Wallet: for EBWOID, EUCC,TAX, VAT,CompanyInfo, ContactPerson
    alt Automatically (EUBW support end-points)
        Bank_Portal->>+Company_Wallet: request presentations 
    else Manually ( EUBW or EUDI Wallet)
        Bank_Portal->>Bank_Portal: embed request into QRCode and provide an openid4vp-URI for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI into the company wallet or scan the QRCode
    end
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Company_Wallet->>Bank_Portal: present the attestations
    Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks
```

### 1.4. Initiator Authorization

```mermaid
sequenceDiagram
    actor Initiator
    participant CompanyWallet 
    alt initiator is legal representative and no additional authorization required  
        Bank_Portal->>+Bank_Portal: check the initiator against the EUCC
    else all other cases (no legal representative and national & cross-boarder cases)
        Note over Initiator,Bank_Wallet: This cases will be handled in the MVP+
    end
```

### 1.5. KYC - Customer Due Diligence Information

```mermaid
sequenceDiagram
    actor Initiator
    Bank_Portal<<->>Bank_Wallet: generate proof-request
    Bank_Portal<<->>Bank_Wallet: OwnershipList,Controllist,UBOList
    alt Company is not a branch
        alt Automatically (EUBW support end-points)
            Bank_Portal->>+Company_Wallet: request presentations
        else Manually ( EUBW or EUDI Wallet)
            Bank_Portal->>Bank_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
            Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
        end                 
        Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
        Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
        Company_Wallet->>Bank_Portal: present the attestations
        Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
    else Company is a branch
        Note over Initiator,Bank_Wallet: This case will be handled in the MVP+
    end

    Note right of Bank_Portal: UBO Calculation is not part of the MVP. This is an internal process
    Bank_Portal->>Bank_Portal: UBO List will be automatically accepted.
```

### 1.6. Signatory Rights and UBOList from Transparency Register

```mermaid
sequenceDiagram
    Bank_Portal<<->>Bank_Wallet: generate proof-request
    Bank_Portal<<->>Bank_Wallet: SignatoryRights
    alt Automatically (EUBW support end-points)
        Bank_Portal->>+Company_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        Bank_Portal->>Bank_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>Bank_Portal: present the attestations
    Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
    
    alt From Transparency Register
        Bank_Portal<<->>Bank_Wallet: generate request for UBOList (TR)
        Bank_Portal->>+Trans.Register: request presentations
        Trans.Register<<->>Trans.Register: mutual authentification ( x509 certificate or eubwoid rulebook)
        Trans.Register<<->>Trans.Register: check the authorization of requester to present requested attestations (own business configuration or visual check)
        Trans.Register->>Bank_Portal: present the attestations
        Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
        Bank_Portal->>Bank_Portal: cross-check the identification data of the UBOs from both UBO Lists
    else From Company Wallet - Automatically
        Note over Company Wallet: This case will be handled in the MVP+
    end
    Note right of Bank_Portal: Reporting is not part of the MVP. This is an internal process
```

### 1.7. UBOs Verification 

```mermaid
sequenceDiagram
    actor UBO
    Bank_Portal->>Bank_Portal: generate link for UBO Verification for each UBO in the UBO List
    alt UBO online (PID)
        UBO->>Bank_Portal: open the link
        Bank_Portal<<->>EUDIWallet: generate request for PID and embed into QRCode
        UBO->>EUDIWallet: scans QR Code with personal wallet to authenticate for Verification process
        EUDIWallet->>+EUDIWallet : mutual authentification ( auth. certificate )
        Initiator->>+EUDIWallet : check the authorization to present requested attestations (visual check)
        EUDIWallet->>Bank_Wallet: present the attestations
        Bank_Portal<<->>Bank_Wallet: verification of attestations (PID rulebook)
        Bank_Portal<<->>Bank_Portal: cross_check attestation data against UBO submitted data (or part of request: Temporal Validity- UBO Rulebook) 
    else other cases online (Pass),offline or videoident  
        Note right of Bank_Portal: This cases will be handled in the MVP+.
    end
```

### 1.8. Sanction check (this will be handled in the MVP+)
* This will be handled in the MVP+

### 1.9 Cross-Check  
```mermaid
sequenceDiagram
    actor ContactPerson 
    Bank_Portal<<->>Bank_Portal: cross check over all attestations 
    Bank_Portal->>+Bank_InternalSystem: transfer data to internal system

    Bank_Portal->>ConstactPerson: Send notification to the contact person that onboarding was successful.
    Bank_Portal<<->>Bank_Portal: Display success notification for initiator
```

### 2. Scenario 2

### 2.1. Contract signing 

```mermaid
sequenceDiagram
    actor Person
    participant EUDIWallet
    Bank_Portal->>+Bank_InternalSystem: Create the contract
    Bank_Portal->>+Bank_Portal: presents document to be signed and QR Code to authenticate for Signature
    alt Person is Initiator=Legal Representative
        Person ->>+EUDIWallet: scans QR Code with personal wallet to authenticate for Signature
        EUDIWallet->>+EUDIWallet:  mutual authentification ( over auth. certificate)
        Person->>+EUDIWallet : check the authorization to present requested attestations (visual check)
        EUDIWallet->Bank_Portal : send the pid information
        Bank_Portal<<->>Bank_Wallet: verification of PID  (rulebook)
        Bank_Portal<<->>Bank_Portal: Cross check against the EUCC
        Person ->>+EUDIWallet: Register for QES Certificate
        Person ->>+EUDIWallet: activates Signature with personal wallet
        Person ->>+EUDIWallet: sign the contract
        Bank_Wallet->>+Bank_Wallet: verifies collected attestations
        Bank_Wallet->>+Bank_Wallet: verifies QES
        Bank_Portal<<->>+Bank_InternalSystem : create the account
        Bank_Portal<<->>Bank_Portal: Display success notification ( account created)
    else other cases (in case that SignatoryPerson is not the initiator or legal represantative)
        Note right of Bank_Portal: This cases will be handled in the MVP+. 
    end
```

### 3. Scenario 3

### 3.1. Onboarding process (for future access of bank services)
This will be handled in the MVP+

### 3.2. IBAN Issuing

```mermaid
sequenceDiagram
    actor Initiator
    participant Company_Wallet
    Initiator ->>+Bank_Portal : Selects "IBAN-OV Attestation" service

    Bank_Portal ->>+Bank_InternalSystem: Retrieves authoritative IBAN-OV data
    Bank_Portal ->>+Bank_Wallet : Creates structured IBAN-OV EAA
    Bank_Portal ->>+Company_Wallet : Delivers signed IBAN-OV EAA
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to issue the attestations (own bussiness configuration)
    Company_Wallet<<->>Company_Wallet: verification of attestations (IBAN-OV rulebook issuance)
    Company_Wallet->>+Bank_Portal: confirm the acceptance
    Bank_Portal->>+Company_Wallet: issue the attestation
    Company_Wallet<<->>Company_Wallet: store the attestation
    
    Bank_Portal ->>+Bank_Portal : Displays success notification (IBAN issued)
```

