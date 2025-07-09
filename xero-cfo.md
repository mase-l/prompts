# Xero CFO Custom GPT
* Virtual CFO for small businesses, integrated directly with the Xero Accounting API. Get instant insights from your Xero data.
* Live Xero Data: Real-time financials pulled from any Xero endpoint (e.g., Profit & Loss, Invoices, Cash Summary).
* Visual Dashboards: Full-width, interactive dashboards using React, shadcn/ui, and Recharts.

## Example Questions
* “Compare revenue month-on-month for the last year.”
* “Show my top 5 customers by sales.”
* “Where did most of my expenses go last quarter?”
* “Can you give advice on reducing overheads?”

## Prompt
---

```## 1. Role

You are **Custom-CFO**, a friendly, casually-professional virtual CFO who helps small-business owners understand and act on the financial data in their Xero account.

---

## 2. Super-powers

* **Data access** – Query any endpoint in the **Xero Accounting & Assets APIs** (OAuth already handled).
  *Examples:* `Invoices`, `Bills`, `BankTransactions`, `Reports/ProfitAndLoss`, `Reports/CashSummary`, `FixedAssets`, etc.
* **Analysis & advice** – Turn raw numbers into plain-language insights and, when explicitly requested, strategic recommendations.
* **Industry awareness** – Adapt explanations and examples to the user’s sector (retail, SaaS, agency, manufacturing, …) once known.
* **Visual dashboards** – Whenever you illustrate numbers or trends, **generate a full-width dashboard** via **canvas** using **shadcn/ui components** and Recharts

---

## 3. Behaviour Rules

1. **User-led workflow** – Fetch data, build dashboards, or give advice.
2. **Evidence first** – Base every numeric claim on fresh Xero data; cite the report name and period inline.
3. **Visual by default** – When showing financials, comparisons, trends, or summaries (e.g., “compare revenue month-on-month”), create a **canvas dashboard web app** with appropriate charts instead of listing raw numbers alone.
4. **Clarity over jargon** – Explain finance terms in everyday language; define acronyms on first use.
5. **No hallucinations** – If Xero lacks the required info, ask clarifying questions rather than guessing.
6. **Privacy & security** – Never expose tokens, raw credentials, or internal system details.

---

## 4. Tool-Calling Contracts

### 4.1 Xero API

Emit **exactly one** JSON function call to retrieve data:

`json
{
  "name": "xero.query",
  "arguments": {
    "endpoint": "Reports/ProfitAndLoss",
    "params": {
      "fromDate": "2025-06-01",
      "toDate":   "2025-06-30"
    }
  }
}
`

* **`endpoint`** (string, required) – any valid Xero path (e.g., `"Invoices"`, `"Reports/CashSummary"`).
* **`params`** (object, optional) – key-value query-string parameters per Xero docs.

### 4.2 Canvas Dashboard App

When I ask for metrics, generate a **canvas** call that builds an interactive dashboard **app** (not just a static page) using **shadcn/ui** and **Recharts**.  

- The dashboard must span the full viewport width.  
- Import **only** from `@/components/ui/*` (Card, Tabs, etc.) and from `recharts`.  
- We are limited to 230 code lines in Canvas. Make code as efficient as possible with refactors of repetitive code, merging if statements etc. If the code is over 230 lines it'll fail to display in the Canvas so this is crucial.
- Scaffold a top (or side) **navigation menu / tab bar** that persists across requests.
  - Each menu item or tab corresponds to a report view (e.g. “Profit”, “Expenses”, “Sales”).  
  - When I request a new report, append a new menu item and render its content—do not overwrite existing views. 
  - Revisit a view by clicking its tab; only that view’s contents should re-render when updated.
- Inside each view, include KPI cards (`<CardHeader>`, `<CardContent>`), charts, tables, etc., based on what I’ve asked.
- Always rename the canvas to reflect the current set of tabs (e.g. “Dashboard App: Profit • Expenses • Sales”).

---

## 5. Response Workflow

| Step              | What you do                                            | How it looks in chat                |
| ----------------- | ------------------------------------------------------ | ----------------------------------- |
| **🔍 Plan**       | Briefly state which data you will pull.                | “Let me compare June to May P\&L …” |
| **🛠️ Call tool** | Dispatch the `xero.query` call (see §4.1).             | *function call*                     |
| **📊 Analyse**    | Convert the API JSON into insights & trends.           | *private reasoning*                 |
| **🎨 Visualise**  | Build a full-width shadcn/ui dashboard via **canvas**. | *canvas call*                       |
| **💬 Respond**    | Summarise insights in clear, friendly prose.           | See §6 format                       |

---

## 6. Output Format

1. **Headline (one sentence)** – the direct answer.
2. **Embedded dashboard** (canvas rendering) for any comparisons or trend analysis.
3. **Bulleted insights** under short **bold** headings (e.g., **Cash Inflows**, **Key Expenses**).
4. **CFO Advice** (optional) – include only if the user asked for recommendations.
5. Finish with an **invitation or prompt** (e.g., “Would you like a breakdown by customer?”) to keep the conversation user-driven.

---

## 7. Style Cheatsheet

* Tone: friendly, patient, casually professional.
* Show numbers clearly (e.g., “$ 83 200” instead of “83200”).
* Cite data inline: “(Cash Summary, 8 Jul 2025)”.
* Prefer short sub-headings to large blocks of text.

---

## 8. Illustrative Example

**User:** “Compare my revenue month-on-month for the last six months.”

**Assistant (plan):**

> I’ll fetch the monthly Profit & Loss for the past six months and visualise the revenue trend.

*🛠️* `xero.query` *(function call, omitted for brevity)*

**Assistant (visualise):**
*🎨* *canvas call rendering a full-width bar chart using shadcn/ui components*

**Assistant (final):**
**Headline:** Revenue has grown **15 % overall** in the past six months, with the sharpest jump (+8 %) in April.

* **Steady growth** from February to April; slight dip in May.
* **June rebound** (+6 %) driven by new subscription tier.

Would you like to see the same dashboard for gross profit or cash flow?```

---

## GPT Actions Schema
```openapi: 3.1.0
info:
  title: Xero Accounting API (Read-Only)
  version: "9.0.0"
  description: "Read-Only OpenAPI specification for Xero's Accounting API, for retrieving core financial data like invoices, accounts, contacts, and reports."
  termsOfService: "https://developer.xero.com/xero-developer-platform-terms-conditions/"
  contact:
    name: "Xero Platform Team"
    url: "https://developer.xero.com"
    email: "api@xero.com"
servers:
  - url: https://api.xero.com
    description: "Main Xero API server"
security:
  - OAuth2:
      - openid
      - profile
      - email
      - offline_access
      - accounting.settings.read
      - accounting.transactions.read
      - accounting.reports.read
      - accounting.journals.read
      - accounting.contacts.read
      - accounting.attachments.read
      - accounting.budgets.read
paths:
  /connections:
    get:
      summary: "List your Xero connections (tenants)"
      operationId: getConnections
      security:
        - OAuth2: []
      responses:
        '200':
          description: "A list of your Xero connections"
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: string
                      format: uuid
                      description: "Tenant ID (GUID)"
                    tenantName:
                      type: string
                      description: "Name of the tenant"
  /api.xro/2.0/Accounts:
    get:
      summary: "Retrieves the full chart of accounts"
      operationId: "getAccounts"
      parameters:
        - name: Xero-Tenant-Id
          in: header
          required: true
          description: "Tenant GUID from GET /connections"
          schema:
            type: string
        - name: where
          in: query
          description: "Filter by an any element (e.g. Status==\"ACTIVE\" AND Type==\"BANK\")"
          schema:
            type: string
        - name: order
          in: query
          description: "Order by an any element"
          schema:
            type: string
      responses:
        "200":
          description: "A successful response with an array of Account objects"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Accounts"
  /api.xro/2.0/Accounts/{AccountID}:
    get:
      summary: "Retrieves a single account by ID"
      operationId: "getAccount"
      parameters:
        - name: Xero-Tenant-Id
          in: header
          required: true
          description: "Tenant GUID from GET /connections"
          schema:
            type: string
        - name: AccountID
          in: path
          required: true
          description: "Unique identifier for Account object"
          schema:
            type: string
            format: uuid
      responses:
        "200":
          description: "A successful response with an array containing a single Account"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Accounts"
  /api.xro/2.0/Invoices:
    get:
      summary: "Retrieves sales invoices or purchase bills"
      operationId: "getInvoices"
      parameters:
        - name: Xero-Tenant-Id
          in: header
          required: true
          description: "Tenant GUID from GET /connections"
          schema:
            type: string
        - name: where
          in: query
          description: "Filter by an any element (e.g. Status == \"DRAFT\")"
          schema:
            type: string
        - name: order
          in: query
          description: "Order by an any element"
          schema:
            type: string
        - name: IDs
          in: query
          description: "Filter by a comma-separated list of InvoicesIDs."
          schema:
            type: array
            items:
              type: string
              format: uuid
        - name: page
          in: query
          description: "e.g. page=1 – Up to 100 invoices will be returned in a single API call."
          schema:
            type: integer
        - name: includeArchived
          in: query
          description: "e.g. includeArchived=true"
          schema:
            type: boolean
        - name: unitdp
          in: query
          description: "e.g. unitdp=4 – You can opt in to use four decimal places for unit amounts"
          schema:
            type: integer
      responses:
        "200":
          description: "A successful response with an array of Invoices"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Invoices"
  /api.xro/2.0/Invoices/{InvoiceID}:
    get:
      summary: "Retrieves a specific sales invoice or purchase bill"
      operationId: "getInvoice"
      parameters:
        - name: Xero-Tenant-Id
          in: header
          required: true
          description: "Tenant GUID from GET /connections"
          schema:
            type: string
        - name: InvoiceID
          in: path
          required: true
          description: "Unique identifier for an Invoice"
          schema:
            type: string
            format: uuid
        - name: unitdp
          in: query
          description: "e.g. unitdp=4 – You can opt in to use four decimal places for unit amounts"
          schema:
            type: integer
      responses:
        "200":
          description: "A successful response with an array containing the specific invoice"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Invoices"
  /api.xro/2.0/Contacts:
    get:
      summary: "Retrieves all contacts"
      operationId: "getContacts"
      parameters:
        - name: Xero-Tenant-Id
          in: header
          required: true
          description: "Tenant GUID from GET /connections"
          schema:
            type: string
        - name: where
          in: query
          description: "Filter by an any element (e.g. ContactStatus==\"ACTIVE\")"
          schema:
            type: string
        - name: order
          in: query
          description: "Order by an any element"
          schema:
            type: string
        - name: IDs
          in: query
          description: "Filter by a comma-separated list of ContactIDs"
          schema:
            type: array
            items:
              type: string
              format: uuid
        - name: page
          in: query
          description: "e.g. page=1 – Up to 100 contacts will be returned in a single API call."
          schema:
            type: integer
        - name: includeArchived
          in: query
          description: "e.g. includeArchived=true"
          schema:
            type: boolean
      responses:
        "200":
          description: "A successful response with an array of Contacts"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Contacts"
  /api.xro/2.0/Contacts/{ContactID}:
    get:
      summary: "Retrieves a specific contact by ID"
      operationId: "getContact"
      parameters:
        - name: Xero-Tenant-Id
          in: header
          required: true
          description: "Tenant GUID from GET /connections"
          schema:
            type: string
        - name: ContactID
          in: path
          required: true
          description: "Unique identifier for a Contact"
          schema:
            type: string
            format: uuid
      responses:
        "200":
          description: "A successful response with an array containing the specific contact"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Contacts"
components:
  securitySchemes:
    OAuth2:
      type: oauth2
      description: "Xero's OAuth 2.0 flow for authentication"
      flows:
        authorizationCode:
          authorizationUrl: https://login.xero.com/identity/connect/authorize
          tokenUrl: https://identity.xero.com/connect/token
          refreshUrl: https://identity.xero.com/connect/token
          scopes:
            openid: "Grant read-only access to your open id"
            email: "Grant read-only access to your email"
            profile: "your profile information"
            offline_access: "Maintain a refresh token"
            accounting.transactions.read: "Grant read-only access to invoices, credit notes, etc."
            accounting.reports.read: "Grant read-only access to accounting reports"
            accounting.journals.read: "Grant read-only access to journals"
            accounting.settings.read: "Grant read-only access to organisation and account settings"
            accounting.contacts.read: "Grant read-only access to contacts and contact groups"
            accounting.attachments.read: "Grant read-only access to attachments"
            accounting.budgets.read: "Grant read-only access to budgets"
  responses:
    400Error:
      description: "A failed request due to validation errors"
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
  schemas:
    Account:
      type: object
      required:
      - Name
      - Type
      properties:
        Code:
          type: string
          description: "Customer defined alpha numeric account code (max length = 10)"
        Name:
          type: string
          description: "Name of account (max length = 150)"
          maxLength: 150
        AccountID:
          type: string
          format: uuid
          readOnly: true
        Type:
          $ref: '#/components/schemas/AccountType'
        BankAccountNumber:
          type: string
          description: "For bank accounts only"
        Status:
          type: string
          description: "ACTIVE or ARCHIVED"
          enum: [ACTIVE, ARCHIVED]
        Description:
          type: string
          description: "Description of the Account (max length = 4000)"
        BankAccountType:
          type: string
          description: "For bank accounts only"
          enum: [BANK, CREDITCARD, PAYPAL, NONE, ""]
        CurrencyCode:
          $ref: '#/components/schemas/CurrencyCode'
        TaxType:
          type: string
          description: "The tax type from taxRates"
        EnablePaymentsToAccount:
          type: boolean
          description: "Boolean – can payments be applied to this account"
        ShowInExpenseClaims:
          type: boolean
          description: "Boolean – is this account available for expense claims"
        Class:
          type: string
          readOnly: true
          enum: [ASSET, EQUITY, EXPENSE, LIABILITY, REVENUE]
        SystemAccount:
          type: string
          readOnly: true
          description: "If this is a system account then this element is returned"
        ReportingCode:
          type: string
        ReportingCodeName:
          type: string
          readOnly: true
        HasAttachments:
          type: boolean
          readOnly: true
        UpdatedDateUTC:
          type: string
          format: date-time
          readOnly: true
        AddToWatchlist:
          type: boolean
        ValidationErrors:
          type: array
          items:
            $ref: '#/components/schemas/ValidationError'
    Accounts:
      type: object
      properties:
        Accounts:
          type: array
          items:
            $ref: '#/components/schemas/Account'
    Contact:
      type: object
      required:
        - Name
      properties:
        ContactID:
          type: string
          format: uuid
          readOnly: true
        ContactNumber:
          type: string
          maxLength: 50
        AccountNumber:
          type: string
          maxLength: 50
        ContactStatus:
          type: string
          enum: [ACTIVE, ARCHIVED]
        Name:
          type: string
          maxLength: 255
        FirstName:
          type: string
          maxLength: 255
        LastName:
          type: string
          maxLength: 255
        EmailAddress:
          type: string
          maxLength: 255
        BankAccountDetails:
          type: string
        IsSupplier:
          type: boolean
        IsCustomer:
          type: boolean
        DefaultCurrency:
          $ref: '#/components/schemas/CurrencyCode'
        UpdatedDateUTC:
          type: string
          format: date-time
          readOnly: true
        HasAttachments:
          type: boolean
          readOnly: true
        ValidationErrors:
          type: array
          items:
            $ref: '#/components/schemas/ValidationError'
    Contacts:
      type: object
      properties:
        Contacts:
          type: array
          items:
            $ref: '#/components/schemas/Contact'
    Invoice:
      type: object
      required:
        - Type
        - Contact
        - LineItems
      properties:
        Type:
          type: string
          description: "ACCPAY for bills, ACCREC for sales invoices"
          enum: [ACCPAY, ACCREC]
        Contact:
          $ref: '#/components/schemas/Contact'
        LineItems:
          type: array
          items:
            $ref: '#/components/schemas/LineItem'
        Date:
          type: string
          format: date
        DueDate:
          type: string
          format: date
        LineAmountTypes:
          $ref: '#/components/schemas/LineAmountTypes'
        InvoiceNumber:
          type: string
          maxLength: 255
        Reference:
          type: string
        BrandingThemeID:
          type: string
          format: uuid
        CurrencyCode:
          $ref: '#/components/schemas/CurrencyCode'
        CurrencyRate:
          type: number
          format: double
        Status:
          type: string
          enum: [DRAFT, SUBMITTED, DELETED, AUTHORISED, PAID, VOIDED]
        SubTotal:
          type: number
          format: double
          readOnly: true
        TotalTax:
          type: number
          format: double
          readOnly: true
        Total:
          type: number
          format: double
          readOnly: true
        InvoiceID:
          type: string
          format: uuid
          readOnly: true
        HasAttachments:
          type: boolean
          readOnly: true
        AmountDue:
          type: number
          format: double
          readOnly: true
        AmountPaid:
          type: number
          format: double
          readOnly: true
        UpdatedDateUTC:
          type: string
          format: date-time
          readOnly: true
        ValidationErrors:
          type: array
          items:
            $ref: '#/components/schemas/ValidationError'
    Invoices:
      type: object
      properties:
        Invoices:
          type: array
          items:
            $ref: '#/components/schemas/Invoice'
    LineItem:
      type: object
      required:
        - Description
      properties:
        LineItemID:
          type: string
          format: uuid
          readOnly: true
        Description:
          type: string
        Quantity:
          type: number
          format: double
        UnitAmount:
          type: number
          format: double
        ItemCode:
          type: string
        AccountCode:
          type: string
        TaxType:
          type: string
        TaxAmount:
          type: number
          format: double
          readOnly: true
        LineAmount:
          type: number
          format: double
        Tracking:
          type: array
          items:
            $ref: '#/components/schemas/LineItemTracking'
    LineItemTracking:
      type: object
      properties:
        TrackingCategoryID:
          type: string
          format: uuid
        TrackingOptionID:
          type: string
          format: uuid
        Name:
          type: string
          readOnly: true
        Option:
          type: string
          readOnly: true
    ReportWithRows:
      type: object
      properties:
        Reports:
          type: array
          items:
            $ref: '#/components/schemas/ReportWithRow'
    ReportWithRow:
      type: object
      properties:
        ReportID:
          type: string
        ReportName:
          type: string
        ReportTitle:
          type: string
        ReportType:
          type: string
        ReportTitles:
          type: array
          items:
            type: string
        ReportDate:
          type: string
        Rows:
          type: array
          items:
            $ref: "#/components/schemas/ReportRows"
        UpdatedDateUTC:
          type: string
          format: date-time
          readOnly: true
    ReportRows:
      type: object
      properties:
        RowType:
          $ref: '#/components/schemas/RowType'
        Title:
          type: string
        Cells:
          type: array
          items:
            $ref: '#/components/schemas/ReportCell'
        Rows:
          type: array
          items:
            type: object
    ReportCell:
      type: object
      properties:
        Value:
          type: string
        Attributes:
          type: array
          items:
            $ref: '#/components/schemas/ReportAttribute'
    ReportAttribute:
      type: object
      properties:
        Id:
          type: string
        Value:
          type: string
    RowType:
      type: string
      enum: [Header, Section, Row, SummaryRow]
    AccountType:
      type: string
      enum: [BANK, CURRENT, CURRLIAB, DEPRECIATN, DIRECTCOSTS, EQUITY, EXPENSE, FIXED, INVENTORY, LIABILITY, NONCURRENT, OTHERINCOME, OVERHEADS, PREPAYMENT, REVENUE, SALES, TERMLIAB, PAYG]
    CurrencyCode:
      type: string
    LineAmountTypes:
      type: string
      enum: [Inclusive, Exclusive, NoTax]
    Error:
      type: object
      properties:
        ErrorNumber:
          type: integer
        Type:
          type: string
        Message:
          type: string
        Elements:
          type: array
          items:
            $ref: '#/components/schemas/Element'
    Element:
      type: object
      properties:
        ValidationErrors:
          type: array
          items:
            $ref: '#/components/schemas/ValidationError'
        Data:
          type: object
          description: "Contains the raw data for the object with the validation error"
    ValidationError:
      type: object
      properties:
        Message:
          type: string
          description: "The validation error message"```
