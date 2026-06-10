# Creating Demo Data in Dataverse
Step-by-step guide for seeding a Dataverse environment with demo accounts, products, leads, opportunities, and opportunity line items using MCP `create_record` / `update_record` / `read_query` tools.
---
## Step 1 — Look Up Reference Data
Products and opportunities require existing reference records. Query these first and save the GUIDs.
```sql-- Unit of measure schedule (required for product)SELECT TOP 1 uomscheduleid, name FROM uomschedule
-- Default unit (required for product and opportunity line items)SELECT TOP 1 uomid, name FROM uom
-- Transaction currency (required for opportunity)SELECT TOP 1 transactioncurrencyid, currencyname FROM transactioncurrency
-- Existing accounts (if any — needed as customerid on opportunity)SELECT TOP 5 accountid, name FROM account```
---
## Step 2 — Create Accounts
Opportunities require a `customerid` lookup. Create accounts first if none exist.
```jsonc// create_record on "account"{  "name":         "<Company Name>",  "industrycode": <int>,           // see Choice Reference below  "description":  "<what the company does>"}```
Create enough accounts so opportunities can reference them. 3–5 is a good starting point.
---
## Step 3 — Create Products
```jsonc// create_record on "product"{  "name":                  "<Product Name>",  "productnumber":         "<SKU>",  "price":                 <decimal>,  "description":           "<product description>",  "quantitydecimal":       0,  "productstructure":      1,       // 1=Product, 2=Product Family, 3=Bundle  "producttypecode":       1,       // 1=Sales Inventory, 2=Misc Charges, 3=Services, 4=Flat Fees  "defaultuomid":          {"recordId":"<uom-guid>",         "relatedTable":"uom"},  "defaultuomscheduleid":  {"recordId":"<uomschedule-guid>", "relatedTable":"uomschedule"}}```
> Gotcha: Products are created in Draft state (`statecode=2`). They cannot be attached to opportunities until activated. See Step 4.
---
## Step 4 — Activate Products
Products must be moved from Draft to Active before they can be used as opportunity line items.
```jsonc// update_record on "product", recordId = <product-guid>{"statecode": 0, "statuscode": 1}```
Apply this to every product created in Step 3.
---
## Step 5 — Create Leads
Leads are standalone — no lookups required.
```jsonc// create_record on "lead"{  "firstname":              "<First>",  "lastname":               "<Last>",            // required  "subject":                "<interest summary>", // required  "companyname":            "<Company>",  "jobtitle":               "<Title>",  "emailaddress1":          "<email>",  "telephone1":             "<phone>",  "address1_city":          "<City>",  "address1_stateorprovince":"<State>",  "industrycode":           <int>,  "description":            "<details>",  "estimatedamount":        <decimal>,  "budgetstatus":           <int>,   // 0=No Budget, 1=May Buy, 2=Can Buy, 3=Will Buy  "leadqualitycode":        <int>,   // 1=Hot, 2=Warm, 3=Cold  "leadsourcecode":         <int>    // 1–10 — see Choice Reference}```
Tips for realistic variety:- Mix `leadqualitycode` across Hot / Warm / Cold- Vary `budgetstatus` and `leadsourcecode`- Use different cities and industries- Reference product names in `subject` / `description` to tie the story together
---
## Step 6 — Create Opportunities
Each opportunity requires `customerid` (account or contact) and `transactioncurrencyid`.
```jsonc// create_record on "opportunity"{  "name":                    "<Account - Deal Name>",  "customerid":              {"recordId":"<account-guid>", "relatedTable":"account"},  "transactioncurrencyid":   {"recordId":"<currency-guid>","relatedTable":"transactioncurrency"},  "estimatedvalue":          <decimal>,  "budgetamount":            <decimal>,  "closeprobability":        <int 0-100>,  "estimatedclosedate":      "YYYY-MM-DD",  "budgetstatus":            <int>,  "need":                    <int>,   // 0=Must have, 1=Should have, 2=Good to have, 3=No need  "opportunityratingcode":   <int>,   // 1=Hot, 2=Warm, 3=Cold  "salesstage":              <int>,   // 0=Qualify, 1=Develop, 2=Propose, 3=Close  "msdyn_forecastcategory":  <int>,   // 100000001=Pipeline, 100000002=Best case, 100000003=Committed  "purchasetimeframe":       <int>,   // 0=Immediate, 1=This Qtr, 2=Next Qtr, 3=This Year  "description":             "<deal details>"}```
Tips for realistic variety:- Spread `salesstage` across Qualify → Close- Mix `opportunityratingcode` and `msdyn_forecastcategory`- Set `closeprobability` higher for later sales stages- Assign multiple opportunities to the same account for richer data
---
## Step 7 — Attach Products to Opportunities
Create `opportunityproduct` line items to link products to opportunities.
```jsonc// create_record on "opportunityproduct"{  "opportunityid":          {"recordId":"<opp-guid>",      "relatedTable":"opportunity"},  "productid":              {"recordId":"<product-guid>",  "relatedTable":"product"},  "uomid":                  {"recordId":"<uom-guid>",      "relatedTable":"uom"},  "transactioncurrencyid":  {"recordId":"<currency-guid>", "relatedTable":"transactioncurrency"},  "opportunityproductname": "<product display name>",  "priceperunit":           <decimal>,  "quantity":               <decimal>,  "ispriceoverridden":      true,     // use manual pricing (no price list required)  "isproductoverridden":    false     // false = catalog product, true = write-in}```
Multi-product opportunities: create one `opportunityproduct` per product per opportunity.
> Prerequisite: Products must be Active (Step 4). Draft products will fail with _"The product cannot be added because it is not active"_.
---
## Choice Value Reference
| Field | Values ||---|---|| `industrycode` | 1=Accounting, 2=Agriculture, 3=Broadcasting, 4=Brokers, 5=Building Supply, 6=Business Services, 7=Consulting, 8=Consumer Services, 12=Durable Mfg, 16=Financial, 20=Insurance, 24=Petrochemical, 25=Service Retail, 29=Specialty Realty, 30=Transportation, 31=Utility, 33=Wholesale _(partial list — use `describe` for all)_ || `budgetstatus` | 0=No Committed Budget, 1=May Buy, 2=Can Buy, 3=Will Buy || `leadqualitycode` | 1=Hot, 2=Warm, 3=Cold || `leadsourcecode` | 1=Advertisement, 2=Employee Referral, 3=External Referral, 4=Partner, 5=Public Relations, 6=Seminar, 7=Trade Show, 8=Web, 9=Word of Mouth, 10=Other || `opportunityratingcode` | 1=Hot, 2=Warm, 3=Cold || `need` | 0=Must have, 1=Should have, 2=Good to have, 3=No need || `salesstage` | 0=Qualify, 1=Develop, 2=Propose, 3=Close || `msdyn_forecastcategory` | 100000001=Pipeline, 100000002=Best case, 100000003=Committed, 100000004=Omitted, 100000005=Won, 100000006=Lost || `purchasetimeframe` | 0=Immediate, 1=This Quarter, 2=Next Quarter, 3=This Year, 4=Unknown || `productstructure` | 1=Product, 2=Product Family, 3=Product Bundle || `producttypecode` | 1=Sales Inventory, 2=Misc Charges, 3=Services, 4=Flat Fees || `statecode` (product) | 0=Active, 1=Retired, 2=Draft, 3=Under Revision || `statecode` (opportunity) | 0=Open, 1=Won, 2=Lost || `statecode` (lead) | 0=Open, 1=Qualified, 2=Disqualified |