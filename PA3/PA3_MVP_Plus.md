# PA3 MVP+ Workflow

MVP Restrictions:

- no restriction in regard to initiator, legal repr and ubo list  
- no restriction in regard to signatory rights
- no restriction in regard to IBAN request (wih delay)
- Mutual authentication is default true (no TLOL or device binding checks applied).
- The company is authorized to present the credential and receive attestation
- The company is a branch
- The company is risk-related company (sanction list check is required)

## Pre-requisites
This are the Pre-requisites for the company and bank in order to run the MVP.

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
    Company ->> Company: issue CompanyInfo, ContactPerson
    Company ->> Company: issue OwnerList, CntrolList, UBOList
    Company ->> Company: issue SignatoryRights 
    Company ->> TransparencyRegister: submit the UBO 
    TransparencyRegister ->> Company : issue UBOList(TR)
    Auth.Source ->> Bank : issue EBWOID
    Auth.Source ->> Company : issue TFS 
```

### 1. Scenario 1

### 1.1. Legal Entity Selection
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
    option Manually ( EUBW or EUDI Wallet)
        Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
        Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
    end
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Company_Wallet->>Bank_Portal: present the attestations
    Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks
```

### 1.4. Initiator Authorization

```mermaid
sequenceDiagram
    actor Person
    critical 
        option cross-boarder case
            Bank_Portal<<->>Bank_Wallet: generate proof-request PoR
            critical
                option Automatically (EUBW support end-points)
                    Bank_Portal->>+Company_Wallet: request presentations
                option Manually ( EUBW or EUDI Wallet)
                    Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
                    Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
            end
            Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
            Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
            Company_Wallet->>Bank_Portal: present the attestations
            Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks
        option natonal case and initiator is not legal representative
            Bank_Portal<<->>Bank_Wallet: generate proof-request EU PoA
            critical
                option Automatically (EUBW support end-points)
                    Bank_Portal->>+Company_Wallet: request presentations
                option Manually ( EUBW or EUDI Wallet)
                    Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
                    Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
            end
            Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
            Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
            Company_Wallet->>Bank_Portal: present the attestations
            Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks
    end   
```

### 1.5. Additionally KYC information

```mermaid
sequenceDiagram
    actor Person
    Bank_Portal<<->>Bank_Wallet: generate proof-request
    Bank_Portal<<->>Bank_Wallet: OwnershipList,Controllist,UBOList,SignatoryRights
    critical
        option Automatically (EUBW support end-points)
            Bank_Portal->>+Company_Wallet: request presentations
        option Manually ( EUBW or EUDI Wallet)
            Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
            Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
    end
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>Bank_Portal: present the attestations
    Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
    
    Bank_Portal->>Bank_Portal: provide input field to specify the endpoint of the corporate wallet 
    critical
        option Automatically (EUBW support end-points)
            Bank_Portal->>+Company_Wallet: request presentations
        option Manually ( EUBW or EUDI Wallet)
            Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
            Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
    end
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>Bank_Portal: present the attestations
    Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
    
    Note right of Bank_Portal: UBO Calculation is not part of the MVP. This is an internal process
    Bank_Portal->>Bank_Portal: UBO List will be automatically accepted.
```

### 1.6. UBOList from Transparency Register

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
        Bank_Portal->>+Company_Wallet: request presentations
        Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
        Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration or visual check)
        Company_Wallet->>Bank_Portal: present the attestations
    end
    Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
    Bank_Portal->>Bank_Portal: cross-check the identification data of the UBOs from both UBO Lists
    Note right of Bank_Portal: Reporting is not part of the MVP. This is an internal process
```

### 1.7. UBOs Verification
```mermaid
sequenceDiagram
    UBOs->>Bank_Portal: open the link
    critical
        option UBO online (PID or PASS
            Bank_Portal<<->>EUDIWallet: generate request for PID/PASS and embed into QRCode
            UBOs->>EUDIWallet: scans QR Code with personal wallet to authenticate for Verification process
            EUDIWallet->>+EUDIWallet : mutual authentification ( auth. certificate )
            Initiator->>+EUDIWallet : check the authorization to present requested attestations (visual check)
            EUDIWallet->>Bank_Wallet: present the attestations
            Bank_Portal<<->>Bank_Wallet: verification of attestations (PID rulebook)
            Bank_Portal<<->>Bank_Portal: cross_check attestation data against UBO submitted data (or part of request: Temporal Validity)
        option UBO Offline 
            Bank_Portal<<->>Bank_Wallet: generate request for EU POA 
            critical 
                option Automatically (EUBW support end-points)
                    Bank_Portal->>+Company_Wallet: request presentations
                option Manually ( EUBW or EUDI Wallet)
                    Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
                    Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
                option other
                    Note right of Bank_Portal: Use the video ident
            end 
            Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
            Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
            Company_Wallet->>Bank_Portal: present the attestations
            Bank_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
        option Other 
             Bank_Portal<<->>Bank_Wallet: usew the video ident
    end
```

### 1.8. Sanction check 

```mermaid
sequenceDiagram
    actor Person
    Bank_Portal<<->>Bank_Wallet: generate proof-request TFS 
    critical
    option Automatically (EUBW support end-points)
        Bank_Portal->>+Company_Wallet: request presentations
    option Manually ( EUBW or EUDI Wallet)
        Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
        Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
    end
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Company_Wallet->>Bank_Portal: present the attestations
    Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks
```

### 1.9. Cross-Check
```mermaid
sequenceDiagram
    Bank_Portal<<->>Bank_Portal: cross check over all attestations
    Bank_Portal->>+Bank_InternalSystem: transfer data to internal system

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
    Person ->>+EUDIWallet: scans QR Code with personal wallet to authenticate for Signature
    EUDIWallet->>+EUDIWallet:  mutual authentification ( over auth. certificate)
    Person->>+EUDIWallet : check the authorization to present requested attestations (visual check)
    EUDIWallet->Bank_Portal : send the pid information
    Bank_Portal<<->>Bank_Wallet: verification of PID  (rulebook)

    critical 
        option Person is Initiator
            Bank_Portal<<->>Bank_Portal: Cross check against the EUCC
        option other
            Bank_Portal<<->>Bank_Portal: Cross check against the EUCC and SR 
            critical 
                option person is not a legal representative 
                    critical 
                    Bank_Portal<<->>Bank_Wallet: generate proof-request EU POA  
                    option Automatically (EUBW support end-points)
                        Bank_Portal->>+Company_Wallet: request presentation POA 
                    option Manually ( EUBW or EUDI Wallet)
                        Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
                        Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
                    end
                    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
                    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
                    Company_Wallet->>Bank_Portal: present the attestations
                    Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks                    
                option rest     
            end
    end

    Person ->>+EUDIWallet: Register for QES Certificate
    Person ->>+EUDIWallet: activates Signature with personal wallet
    Person ->>+EUDIWallet: sign the contract

    Bank_Wallet->>+Bank_Wallet: verifies collected attestations
    Bank_Wallet->>+Bank_Wallet: verifies QES
    Bank_Portal<<->>+Bank_InternalSystem : create the account
    Bank_Portal<<->>Bank_Portal: Display success notification ( account created)
    Note right of Bank_Portal: in case that SignatoryPerson is not the initiator or legal represantative this will be handled in MVP+
```

### 3. Scenario 3

### 3.1. Onboarding process (this will be handled in the MVP+)

```mermaid
sequenceDiagram
    actor Person
    participant EUDIWallet

    Bank_Portal<<->>Bank_Wallet: generate request to identity Initiator (PID) and embed into QRCode
    Person->>+EUDIWallet : scans QR Code with personal wallet
    EUDIWallet->>+EUDIWallet : mutual authentification ( auth. certificate )
    Person->>+EUDIWallet : check the authorization for presentation (visual check)
    EUDIWallet->Bank_Portal : send the pid information
    Bank_Portal<<->>Bank_Wallet: verification of PID  (rulebook)
    Bank_Portal<<->>Bank_Portal: Cross check against the EUCC and SR
    critical 
        option person not onboarded and in EUCC
            Bank_Portal<<->>SR Person : issue token 
        option person not onboarded and in SR 
            Bank_Portal<<->>Bank_Wallet: generate proof-request EU POA  
            critical 
                option Automatically (EUBW support end-points)
                    Bank_Portal->>+Company_Wallet: request presentation POA 
                option Manually ( EUBW or EUDI Wallet)
                    Bank_Portal->>Bank_Portal: embed request into QRCode and provide a DeepLink for the request
                    Person->>+Company_Wallet: copy/paste deep-link presentation into the company wallet or scan the QRCode
            end
            Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
            Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
            Company_Wallet->>Bank_Portal: present the attestations
            Bank_Portal<<->>Bank_Wallet: verification of attestations rulebooks                    
            Bank_Portal<<->>SR Person : issue token 
       option other 
            Note right of Bank_Portal: no access to the portal  
    end
```

### 3.2. IBAN Issuing

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
