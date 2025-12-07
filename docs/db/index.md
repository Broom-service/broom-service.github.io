# Databáze

Pro správu DB je použit EF Core.

DB provider je PostgreSQL.

## DB Schéma

(schéma entit, EF Core má ve správě celou DB)

DB schémata:

* `broomsvc`:
      * Users
      * UserRoles
      * UsersAudit
* `broomsvc_apartments`
      * Apartments
      * Reservations
      * ApartmentReservationLinks
      * Cleanings
      * ApartmentsAudits
* `broomsvc_invoices`
      * Suppliers
      * BankAccounts
      * Currency
      * SuppliersBankAccounts
      * Invoices
      * Recepients
      * Vats
      * InvoiceVatSummaries
      * InvoiceItems
      * AmountTypes
      * InvoiceNumberSeries
      
```puml
@startuml

entity "Users" as Users {
  *Id : int
  --
  *Email : string <<UK>>
  *FirstName : string
  *LastName : string
  *State : UserAccountState
}

enum UserAccountState {
    New
    Registered
    Active
    Inactive
}

entity "UserRoles" as UserRoles {
  *Id : int
  --
  *UserId : int <<FK>>
  *Role : string
  --
  UK(UserId, Role)
}

entity "UsersAudit" as UsersAudit {
  *Id : int
  --
  *UserId : int
  *ChangedBy : int
  *Action : string
  *ChangeDate : DateTime
}

entity "Reservations" as Reservations {
  *Id : int
  --
  *ApartmentId : int <<FK>>
  *StartDate : DateOnly
  *EndDate : DateOnly
  *ICalEventId : string
}

entity "Cleanings" as Cleanings {
  *Id : int
  --
  *ReservationId : int <<FK>>
  *ApartmentId : int <<FK>>
  *UserId : int <<FK>>
  ConfirmedAt : DateTimeOffset
}

entity "Apartments" as Apartments {
  *Id : int
  --
  *Name : string
}

entity "ApartmentReservationLinks" as ApartmentReservationLinks {
  *Id : int
  --
  *ApartmentId : int <<FK>>
  *ReservationsICalUri : string
}

entity "ApartmentsAudits" as ApartmentsAudits {
  *Id : int
  --
  *ApartmentId : int
  *ChangedBy : int
  *Action : string
  *ChangeDate : DateTime
}

Users -- UserAccountState
Users ||--o{ UserRoles
Users ||--o{ UsersAudit : Subject (UserId)
Users ||--o{ UsersAudit : Editor (ChangedBy)

Apartments ||--o{ Reservations
Apartments ||--o{ ApartmentsAudits
Apartments ||--o{ ApartmentReservationLinks

Reservations ||--|| Cleanings
Users ||--|| Cleanings

entity "Suppliers" as Suppliers {
    *Id : int
    --
    *Name : string
    *Address : string
    *IdNumber : string (IČO)
    VatNumber : string
}

entity "BankAccounts" as BankAccounts {
    *Id : int
    --
    *BankName: string
    *AccountNumber: string
    *FriendlyName: string
    *Currency: Currency
    Iban: string
    Swift: string
}

enum Currency {
    CZK
    EUR
}

entity "SuppliersBankAccounts" as SBA {
    *SupplierId: int <<FK>>
    *BankAccountId: int <<FK>>
    --
}

BankAccounts -- Currency
Suppliers ||--o{ SBA
SBA }o--|| BankAccounts

entity Invoices {
    *Id : int
    --
    *CreatedByUserId: int <<FK>>
    PaidByUserId: int <<FK>>
    
    Notes: string
    *Paid: bool
    PaidAt: DateTimeOffset
    *InvoiceNumber: string
    *IssuedAt: DateTimeOffset
    *Currency: Currency
    Total: decimal
    TotalVat: decimal
    TotalWithVat: decimal
    QrCode: Array<byte>
    
    *SupplierId: int <<FK>>
    *SupplierName: string
    *SupplierAddress: string
    *SupplierIdNumber: string
    SupplierVat: string
    *SupplierBankName: string
    *SupplierBankAccountNumber: string
    *SupplierBankAccountCurrency: Currency
    SupplierBankIban: string
    SupplierBankSwift: string
    
    *RecepientId: int <<FK>>
    *RecepientName: string
    RecepientNote: string
    RecepientAddress: string
    RecepientIdNumber: string
    RecepientVatNumber: string
    --
    UK(SupplierId, RecepientId, InvoiceNumber)
}

entity Recepients {
    *Id: int
    --
    *Name: string
    Note: string
    Address: string
    IdNumber: string
    VatNumber: string
}

entity Vats {
    *Rate: decimal
    *Currency
}

entity InvoiceVatSummaries {
    *Id: int
    *InvoiceId: int <<FK>>
    *VatRate: decimal
    --
    *Total: decimal
    *TotalWithVat: decimal
    --
}

entity InvoiceItems {
    *Id: int
    --
    *InvoiceId: int <<FK>>
    *ItemNumber: int
    *Name: string
    *VatRate: decimal
    *Price: decimal
    *PriceWithVat: decimal
    *Amount: decimal
    *AmountType: string
    --
    UK(InvoiceId, ItemNumber)
}

entity AmountTypes {
    *Id: int
    --
    *Type: string
    --
    UK(Type)
}

entity InvoiceNumberSeries {
    *Id: int
    --
    *Name: string
    *Template: string
    *LastNumber: int
    *LastUsageDate: DateOnly
    --
    UK(Name)
}

Invoices -- Currency
Vats -- Currency
Invoices ||--|{ InvoiceVatSummaries

Invoices ||--o{ InvoiceItems
InvoiceItems }o--|| AmountTypes
Invoices ||--|| Suppliers
Invoices ||--|| Recepients
Invoices ||--|| Users : CreatedBy
Invoices ||--o| Users : PaidBy

@enduml
```