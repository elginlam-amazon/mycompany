# 🚀 Executive Report: Java 8 → Java 25 Upgrade (`lsfusion.solutions:mycompany`)  
  
## 📋 Executive Summary  
  
AWS Transform Custom (transformation `AWS/java-version-upgrade`, region `ap-southeast-2`) upgraded the lsFusion solution `mycompany` (Maven, single module, `7.0-SNAPSHOT`) from Java 8 to Java 25. All changes were confined to the build configuration in `pom.xml`, and no Java source file needed editing. The build now succeeds on Amazon Corretto 25 and produces class files with bytecode major version 69 (Java 25). The validation summary reports **OVERALL STATUS: COMPLETE**, with no unmet criteria. The run used 41.94 agent minutes, and the result is committed as `f4360954` on branch `atx-result-staging-20261109_115136_8ea7155d`. The original branch `main-upgrade` was left unchanged.  
  
## 🏗️ Application Changes  
  
- 📦 **Scope:** The project contains 7 Java source files plus lsFusion `.lsf` scripts. Only `pom.xml` changed (+64 lines, 0 deleted).  
- 🔐 **Crypto actions (`mycompany.utils.crypto`), unchanged source:** CodeKB mapping of the pre-upgrade code shows these actions:  
  - `CipherAESCBCPKCS7Action` for AES-CBC/PKCS7 encryption  
  - `CipherRSAOAEPMGF1Action` for RSA-OAEP encryption  
  - `GenerateAESKeyIVAction` for AES key and IV generation  
  - `GenerateCSRAction` for CSR generation with BouncyCastle  
  - `SignXAdESEnvelopedAction` for XAdES enveloped XML signatures, using `javax.xml.crypto.dsig` with EC or RSA keys  
- 🏗️ **Autodesk integration (`region.us.integration.autodesk`), unchanged source:**  
  - `AutodeskObjectTreeImportAction` imports the object tree.  
  - `AutodeskPropertiesImportAction` parses Autodesk JSON with Jackson and loads the properties into lsFusion.  
- 🧩 **Common base:** All 7 actions extend lsFusion `InternalAction`. They only use JDK crypto/XML, Jackson, BouncyCastle and commons-lang3 APIs, which work unchanged on Java 25.  
- 🚫 **Not applicable:** Jakarta EE migration, Spring, database/ORM, Lombok and Gradle were skipped because the module uses none of them. The only `javax.*` imports are the JDK packages `javax.crypto` and `javax.xml`.  
  
## 🛠️ Tools Used  
  
- ☕ **Java SDK:** Amazon Corretto 1.8.0_512 for the baseline build and Amazon Corretto 25 (`/usr/lib/jvm/java-25-amazon-corretto`) as the target.  
- 🤖 **AWS Transform Custom CLI (`atx`):** `atx custom def exec --transformation-name AWS/java-version-upgrade`, run in non-interactive mode with config `atx-config-java.json`. It handled planning, editing, building, committing and validation.  
- 🔨 **Apache Maven 3.9.16:** Used for `clean package`, `help:effective-pom` and `dependency:tree`.  
- 🧵 **AspectJ:** `aspectjtools`/`aspectjrt` 1.9.25 and `dev.aspectj:aspectj-maven-plugin` 1.14.  
- 🧠 **CodeKB:** Mapped the old code surface (34 components) to confirm that no source changes were needed.  
- 🔍 **`javap`:** Checked the bytecode version of the build output.  
  
## 💻 Code Changes (`pom.xml` only)  
  
- ⚙️ **Compiler properties:** Added `maven.compiler.source=25` and `maven.compiler.target=25`. These override the Java 1.8 default inherited from the third-party parent `lsfusion.platform.build:logics:7.0-SNAPSHOT`.  
- ⬆️ **AspectJ version:** Raised `aspectj.version` from 1.9.2 (inherited) to **1.9.25**, the AspectJ release with Java 25 compliance support.  
- 🔌 **Weaving plugin replacement:**  
  - The inherited, unmaintained `com.nickwongdev:aspectj-maven-plugin` 1.12.1 execution was disabled with `<phase>none</phase>`.  
  - `dev.aspectj:aspectj-maven-plugin:1.14` was added with compliance, source and target set to 25 and `aspectjtools` 1.9.25 as a plugin dependency.  
  - The `lsfusion.platform:server` aspect library is still configured, so aspect weaving behaves as before.  
- 📌 **Runtime alignment:** Pinned `org.aspectj:aspectjrt` to `${aspectj.version}`, which removed the 1.9.2/1.9.25 version-mismatch warning.  
- 🧹 **Split-package fix:** `lsfusion.platform:server` is now declared locally with an exclusion for `xml-apis:xml-apis`.  
  - The excluded artifact is `xml-apis:1.4.01`, which arrived transitively through batik-dom.  
  - It duplicated JDK `java.xml` packages (`javax.xml.*`, `org.w3c.dom`), and ajc failed with "accessible from more than one module".  
- 🔒 **CVE check:** The new AspectJ artifacts passed the vulnerability scan with no known CVEs.  
  
## ✅ Validation Results  
  
| Criterion | Status |  
|---|---|  
| 🏗️ Build with Java 25 (`mvn -B clean package`) | ✅ PASS (BUILD SUCCESS, ~39 s, bytecode major version 69) |  
| 🧪 Tests pass at baseline rate or better | ➖ N/A (no test sources; 0 tests before and after) |  
| 📛 No non-JDK `javax.*` imports | ✅ PASS (only `javax.crypto` and `javax.xml`) |  
| 📚 Mandatory dependency versions | ➖ N/A (no Spring, Lombok, Jackson, Jakarta or Gradle declared locally) |  
| 📝 Skipped updates documented | ✅ PASS (recorded in the worklog) |  
  
- ⚠️ The only warnings left are AspectJ `[Xlint:adviceDidNotMatch]` warnings from aspects in the platform jar. They do not cause errors.  
  
## ⏱️ Time Savings Estimate  
  
- 🧑‍💻 **Manual effort estimate:** About 1.5–3 engineer-days. The work is small in file count, but diagnosing it by hand would take time:  
  - Tracing Java 8 settings through the remote parent POM chain (`logics` → `base`)  
  - Finding that the `com.nickwongdev` AspectJ plugin cannot target Java 25 and swapping in a replacement without losing the woven aspect library  
  - Tracing the ajc split-package error back to a transitive `xml-apis` dependency through `dependency:tree`  
  - Running repeated full builds (a single build took up to about 7.7 minutes) and verifying the results  
- 🤖 **Automated effort:** 41.94 agent minutes, without human intervention.  
- 💰 **Estimated saving:** About 85–95% of the engineering time for this module.  
  
## 🔜 Next Steps  
  
- 🧪 **Runtime testing:** Deploy to a Java 25 environment and run each crypto action, especially the XAdES signing, against known-good samples. Also run both Autodesk import actions.  
- 🔐 **XML signature behaviour:** Java 17+ disables some XML signature algorithms by default (for example SHA-1 based ones). Confirm that the partners who consume the signatures accept the SHA-256 ECDSA/RSA signatures this code produces.  
- 🧵 **Aspect weaving check:** Confirm at runtime that the `lsfusion.platform:server` aspects are still applied. Review the `adviceDidNotMatch` warnings.  
- 🖥️ **Platform alignment:** Confirm that the lsFusion platform 7.0 server and its dependencies (BouncyCastle, Jackson, batik) are supported on Java 25. Update the Docker base image and the CI/CD `JAVA_HOME` to Corretto 25.  
- 🧪 **Add tests:** The module has no automated tests. Add unit tests for the crypto actions (encrypt/decrypt round trips, signature verification) to reduce risk in future upgrades.  
- 🔀 **Merge:** Review and merge `atx-result-staging-20261109_115136_8ea7155d` into `main-upgrade` using the generated `git_instructions.md`.  
- 🧭 **Long term:** Ask the lsFusion platform maintainers to move the Java version and the AspectJ plugin choice into the parent POM, so the local overrides can be removed.

## Verification results

Copied from this iteration's report.json, progress.json and runtime-validation.log after the final build. This section is not written by the AI.

- 🛠️ Post-upgrade verification build: SUCCESS
- 🧪 Automated tests: none discovered or run (compilation only)
- ⏱️ Transformation duration: 54 minute(s)
- ⏳ Estimated time saved: 0 minute(s)
- 🔎 CodeKB calls (atx): 1

## Golden-master output checks (optional)

This module writes PDF, Excel or JasperReports output, and a green build does not show that the output is unchanged. Optionally, capture each output file before and after the upgrade and compare the captures structurally: PDFBox text per page and Apache POI cell values, never the bytes (PDF and XLSX embed timestamps). Migration knowledge doc 36 has the steps.

High-risk output paths found by the static scan:

- `src/main/lsfusion/accounting/reporting/Accounting_balanceSheet.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/accounting/reporting/Accounting_profitAndLoss.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/accounting/reporting/cashflow/Accounting_cashFlowStatement.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/inventory/adjustments/print/Inventory_printAdjustment.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/inventory/lot/LotBarCode.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/inventory/lot/LotBarCode58x40.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/inventory/shipments/print/DeliveryNote.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/invoicing/invoices/creditnotes/CreditNote.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/invoicing/invoices/print/Invoice.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/invoicing/payments/incoming/print/PaymentReceipt.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/invoicing/reporting/ReconciliationStatement.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/manufacturing/bom/Manufacturing_printStructureAndCost.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/manufacturing/orders/print/Manufacturing_printManufacturingOrder.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/masterdata/items/print/ItemBarCode58x40.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/masterdata/items/print/ItemBarCode58x60.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/masterdata/partners/employees/print/EmployeeBadgeA4_2x5.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/masterdata/partners/employees/print/EmployeeBadgeA4_3x2.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/purchase/orders/print/PurchaseOrder.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/pl/invoicing/invoices/print/Faktura.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/pl/purchase/orders/print/PurchaseOrderPl.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/inventory/receipts/print/ReceiptRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/inventory/shipments/print/ShipmentRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/inventory/shipments/print/Torg13.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/invoicing/invoices/print/InvoicePaymentRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/invoicing/invoices/print/InvoiceRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/invoicing/invoices/print/Torg12.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/invoicing/invoices/print/UPD.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/invoicing/payments/incoming/print/IncomingCashRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/invoicing/payments/outgoing/print/OutgoingCashRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/purchase/orders/print/PurchaseOrderRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/purchase/orders/print/PurchaseQuoteRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/orders/print/SalesAnnexRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/orders/print/SalesCompletionStatementRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/orders/print/SalesOrderRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/orders/print/SalesProposalRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/orders/print/SalesWorkOrderRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/pricelists/print/LabelPromotionRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/pricelists/print/LabelRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/region/ru/sales/pricelists/print/PricelistRu.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/sales/orders/print/SalesOrder.jrxml` (JasperReports: JasperReports template (.jrxml))
- `src/main/lsfusion/sales/orders/print/SalesQuotation.jrxml` (JasperReports: JasperReports template (.jrxml))

No golden-master capture was made in this iteration (the scaffolding is optional).

## Kiro credit usage

💳 Kiro CLI credits used: 3.09  
Includes 0.31 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at 35745c4ae0c51aeaff0d32980c79830db70c1888.
- Previously indexed commit: 35745c4ae0c51aeaff0d32980c79830db70c1888
- Source commit at transformation start: 35745c4ae0c51aeaff0d32980c79830db70c1888
- Duration: 0.0s
