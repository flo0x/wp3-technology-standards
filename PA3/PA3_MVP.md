# PA3 MVP Workflow
MVP Restrictions
- 
## Pre-requisites

```mermaid
sequenceDiagram
    participant Auth.Source
    participant TransparencyRegister
    participant TAX.Administration 
    participant Company
    participant Bank 
    Auth.Source ->> Company : issue EBWOID  
    Auth.Source ->> Company : issue EUCC
    critical
        option PubEAA 
            TAX.Administration ->> Company : issue TAX,VAT
        option EAA
             Company ->> Company: issue TAX,VAT
    end 
    critical
        option Initator POA required  
            Company ->> Company: issue PoA for Initiator
        option SR POA required
            Company ->> Company: issue PoA for SignatoryRights Persons
    end                
    Company ->> Company: issue CompanyInfo
    Company ->> Company: issue ContactPerson 
    Company ->> Company: issue SignatoryRights 
    Company ->> Company: issue OwnerList, CntrolList, UBO
    Company ->> Company: issue UBO
    Company ->> TransparencyRegister: submit the UBO 
    Auth.Source ->> Bank : issue EBWOID
```

### 1. Scenario 1 

### 1.2. Legal Entity Selection
```mermaid
sequenceDiagram
    actor Person
    activate Person
    Person->>+Bank_Portal: Select "open business account" Service
    critical
    option Option1.Wallet_Support_EndPoint (ex. EUBW DirectoryList)
        Bank_Portal->>+Bank_Portal : Provide the list of available legal entities
        Person->>+Bank_Portal: select the legal entity from the list & the respective wallet address
        Bank_Portal->>+Bank_Portal: resolve the endpoint of selected legal entity
    option Option2.Wallet_Support_EndPoint (ex. Resolvable eAddress or endpoint URI)
        Bank_Portal->>+Bank_Portal : Provide an input field
        Person->>+Bank_Portal: fill the address or end-point of the business wallet
        Bank_Portal->>+Bank_Portal: resolve eAddress
    option Option3.Support directly into EUBW
        Note over Company_Wallet: the company wallet already integrate the business process of specific banks
        Person->>+Company_Wallet: Select bank in the EUBW (configured in wallet)
    option Option4. Other: manual process (EUBW or EUDI Wallet)
        Note over Company_Wallet: manuall proces by the Person
    end
    Person->>+Bank_Portal: trigger process
```

### 1.2. Initiator Identification 
```mermaid
sequenceDiagram
    actor Person
    participant EUDIWallet
    critical
    option Option1.PID
        Bank_Portal<<->>Bank_Wallet: generate request to identity Initiator (PID) and embed into QRCode
        Person->>+EUDIWallet : scans QR Code with personal wallet
        EUDIWallet->>+EUDIWallet : mutual authentification ( auth. certificate )
        Person->>+EUDIWallet : check the authorization for presentation (visual check)
        EUDIWallet->Bank_Portal : send the pid information
        Bank_Portal<<->>Bank_Wallet: verification of PID  (rulebook)
    option Option2.Other identification
        Note over EUDIWallet: identification of the person with other identfication means (ex. eID)
    end
```

### 1.3. LegalEntity Identification

```mermaid
sequenceDiagram
    actor Person
    Bank_Portal<<->>Bank_Wallet: generate proof-request
    Bank_Portal<<->>Bank_Wallet: EBWOID, EUCC,TAX, VAT,CompanyInfo, ContactPerson
    critical
    option Automatically (EUBW support end-points)
        Bank_Portal->>+Company_Wallet: request presentations
        Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
        Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
        Company_Wallet->>Bank_Portal: present the attestations
    option Manually ( EUBW or EUDI Wallet)
        Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
        Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
        Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
        Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration or visual check)
        Company_Wallet->>Bank_Portal: present the attestations
    end
    Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks
```

### 1.4. Initiator Authorization (MVP+)

### 1.5. Additionally KYC information

```mermaid
sequenceDiagram
    actor Person
    Bank_Portal<<->>Bank_Wallet: generate proof-request
    Bank_Portal<<->>Bank_Wallet: OwnershipList,Controllist,UBOList,SignatoryRights
    critical
        option Automatically (EUBW support end-points)
            Bank_Portal->>+Company_Wallet: request presentations
            Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
            Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
            Company_Wallet->>Bank_Portal: present the attestations
        option Manually ( EUBW or EUDI Wallet)
            Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
            Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
            Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
            Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration or visual check)
            Company_Wallet->>Bank_Portal: present the attestations
        option is the company a branch
            Note over Person,Bank_Wallet: This case will be handled in the MVP+
        end
    Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)

    Note right of Bank_Portal: UBO Calculation is not part of the MVP. This is an internal process
    Bank_Portal->>Bank_Portal: UBO List will be automatically accepted.
```

### 1.5. UBOList from Transparency Register

```mermaid
sequenceDiagram
    Bank_Portal<<->>Bank_Wallet: generate request for UBOList (TR)
    critical
    option Option1. From Transparency Register
        Bank_Portal->>+Trans.Register: request presentations
        Trans.Register<<->>Trans.Register: mutual authentification ( x509 certificate or eubwoid rulebook)
        Trans.Register<<->>Trans.Register: check the authorization of requester to present requested attestations (own business configuration or visual check)
        Trans.Register->>Bank_Portal: present the attestations
    option Option2. From Company Wallet - Automatically
        Note over Company Wallet: part of MVP+
    end
    Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
    Bank_Portal->>Bank_Portal: cross-check the identification data of the UBOs from both UBO Lists
    Note right of Bank_Portal: Reporting is not part of the MVP. This is an internal process
```

### 1.6. UBOs Verification 
```mermaid
sequenceDiagram
    critical
        option UBO = initiator
            Bank_Portal->>Bank_Portal: cross-check the identification data of the Person from PID and UBO List
            Bank_Portal->>Bank_Portal: are the information ok, go further, else inform the company
        option UBO <> Initiator
            Note right of Bank_Portal: This case will be handled in the MVP+ (ex. UBO is not the initiator)
    end

    Bank_Portal<<->>Bank_Portal: cross check over all attestations
    Bank_Portal->>+Bank_InternalSystem: transfer data to internal system
```

### 1.6. Success  
```mermaid
sequenceDiagram
    Bank_Portal<<->>Bank_Portal: Display success notification ( KYC Data are completly).
```

### 2. Scenario 2

### 2.1. Contract signing 

```mermaid
sequenceDiagram
    actor Person
    participant EUDIWallet
    Bank_Portal->>+Bank_InternalSystem: Create the contract
    Bank_Portal->>+Bank_Portal: presents document to be signed and QR Code to authenticate for Signature
    critical
        option Person <> Initiator
            Person ->>+EUDIWallet: scans QR Code with personal wallet to authenticate for Signature
            EUDIWallet->>+EUDIWallet:  mutual authentification ( over auth. certificate)
            Person->>+EUDIWallet : check the authorization to present requested attestations (visual check)
            EUDIWallet->Bank_Portal : send the pid information
            Bank_Portal<<->>Bank_Wallet: verification of PID  (rulebook)
            Bank_Portal<<->>Bank_Portal: Cross check against the EUCC
            Person ->>+EUDIWallet: Register for QES Certificate
            Person ->>+EUDIWallet: activates Signature with personal wallet
            Person ->>+EUDIWallet: sign the contract
        option other
            Note right of Bank_Portal: This cases will be handled in the MVP+ (legal repr or signatory rights with POA)
    end
    Bank_Wallet->>+Bank_Wallet: verifies collected attestations
    Bank_Wallet->>+Bank_Wallet: verifies QES
    Bank_Portal<<->>+Bank_InternalSystem : create the account
    Bank_Portal<<->>Bank_Portal: Display success notification ( account created)
    Note right of Bank_Portal: in case that SignatoryPerson is not the initiator or legal represantative this will be handled in MVP+
```

### 3. Scenario 3

### 3.1. IBAN Issuing

```mermaid
sequenceDiagram
    actor Person
    Person ->>+Bank_Portal : Selects "IBAN-OV Attestation" service

    Bank_Portal ->>+Bank_InternalSystem: Retrieves authoritative IBAN-OV data
    Bank_Portal ->>+Bank_Wallet : Creates structured IBAN-OV EAA
    Bank_Portal ->>+Company_Wallet : Delivers signed IBAN-OV EAA
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to issue the attestations (own bussiness configuration)
    Company_Wallet->>+Bank_Portal: confirm the acceptance

    Bank_Portal->>+Company_Wallet: issue the attestation
    Bank_Portal ->>+Bank_Portal : Displays success notification (IBAN issued)
```
