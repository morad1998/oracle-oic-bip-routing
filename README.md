# oracle-oic-bip-routing
An enterprise architecture pattern for dynamic document generation and smart approval routing using Oracle Integration Cloud (OIC) and BI Publisher (BIP).

# Oracle Cloud: Smart Routing & Dynamic Document Generation (OIC + BIP)

This repository demonstrates an enterprise-grade architecture pattern for handling complex, multi-tier approval routing and dynamic PDF document generation using **Oracle Integration Cloud (OIC)** and **Oracle BI Publisher (BIP)**.

## 🏗️ Architecture Overview

Many enterprise workflows require generating official documents (like Warranty Certificates, Custom Quotes, or Compliance Reports) based on transactional data. However, if the transactional data violates standard company limits, the document generation must be halted, and an approval workflow must be triggered.

This pattern shifts the complex evaluation logic into a highly performant **BI Publisher SQL Data Model**, which returns a simple `<VALIDATION_FLAG>`. **OIC** then acts as the traffic controller, reading the flag and routing the process accordingly.

### **The Workflow Lifecycle**
1. **Trigger:** A frontend application (e.g., Visual Builder Studio) sends a JSON payload containing the Order Number and the Requester's Identity to an OIC REST endpoint.
2. **First Pass (Evaluation):** OIC makes a SOAP call (`runReport`) to BIP to fetch XML data. 
3. **SQL Engine:** The BIP SQL query evaluates the order lines against strict business rules and checks the requester's security roles. It outputs `<VALIDATION_FLAG>Y</VALIDATION_FLAG>` (Standard) or `N` (Custom).
4. **Smart Routing (OIC Switch):**
   * **If Y (Standard):** OIC makes a *second* SOAP call to BIP, this time requesting `attributeFormat="pdf"`. OIC receives the Base64 PDF, converts it, and emails it directly to the user.
   * **If N (Custom):** OIC halts document generation and routes an actionable approval email to the appropriate management team.

---

## 🛠️ Implementation Guide

### 1. The OIC REST Payload
Your OIC integration should be triggered by a REST endpoint accepting the following JSON structure. *Note: Passing the Requester's username/email is critical for enforcing user-level security in the BIP SQL.*

```json
{
  "OrderNumber": "ORD-12345",
  "RequesterUserName": "user.name@company.com",
  "RequesterEmail": "user.name@company.com",
  "SalesEngineerName": "Jane Doe",
  "ProjectName": "Enterprise Alpha"
}
