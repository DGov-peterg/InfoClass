
# Information Classification Resources
This repository supports the implementation of the [WA Government Information Classification Policy](https://www.wa.gov.au/government/document-collections/western-australian-information-classification-policy)

## Microsoft Purview Config Analyzer
Office of Digital Government has collaborated with Microsoft to develop a Microsoft Purview Configuration Analyzer to identify configurations within entity tenants.  The config utilises a Microsoft Powershell script can be downloaded from [SimplifiedPurviewConfigAnalyserReport.ps1](./SimplifiedPurviewConfigAnalyserReport.ps1)

### Purpose
This script collects Microsoft Purview (Compliance) configuration metadata using the Microsoft Compliance and Exchange Online PowerShell modules. It outputs the results as a concise JSON file and then converts that data into an Excel report for further analysis.

### Overview
This script is designed for Microsoft 365 administrators and auditors to extract metadata about Purview (Compliance) configuration in a tenant.  

> **Note:**  It does not extract or process any user content or sensitive data—only
> configuration metadata.

### Prerequisites & Access Requirements
-   **PowerShell 7.x or Windows PowerShell 5.1**
-   **Modules Required:**
	- ExchangeOnlineManagement
	- Microsoft.Graph (if used for authentication)
	- Import-Module for Compliance Center (if not already loaded)
-   **Network Access:** Ability to connect to Microsoft 365 services.


### Access/Roles Required
| Role | Data Extracted |
|--|--|
| Compliance Administrator | All metadata, including Insider Risk Management policies |
| Compliance Data Administrator	| All metadata, including Insider Risk Management policies |
| Global Administrator | All metadata, including Insider Risk Management policies |
| Global Reader | All metadata except Insider Risk Management policies |
| Security Reader | Most metadata, but may not include all compliance data | 
> **Note:** If the user only has the Global Reader role, all metadata except Insider Risk Management data will be  extraced. For full extraction, use Compliance Admin, Compliance Data Admin, or Global Admin.

### What the Script Does

1.  **Connects to Microsoft Compliance and Exchange Online PowerShell.**
2.  **Fetches configuration metadata** for:
	-   Sensitivity Labels
	-   Label Policies
	-   Retention Policies
	-   DLP Policies
	-   Compliance Tags
	-   Insider Risk Management (if permitted)
	-   Auto Sensitivity Label Policies
3.  **Processes and flattens the metadata** for reporting.
4.  **Outputs the data** as a JSON file and an Excel workbook with multiple tabs.

### Role-Based Data Extraction
-   **Full Metadata Extraction:**  
    If the user has **Compliance Admin**, **Compliance Data Admin**, or **Global Admin** roles, all configuration metadata—including Insider Risk Management—will be extracted.
-   **Partial Metadata Extraction:**  
    If the user has only the **Global Reader** role, all configuration metadata **except Insider Risk Management** will be extracted.  The script will skip or leave blank the Insider Risk Management tab in the Excel output.

### Script Functions & Responsibilities
 1. Write-Log
	-   **Purpose:** Handles logging of errors, warnings, and informational messages to a log file.
	-   **Access:** No special access required.
2. Connect-ComplianceCenter
	-   **Purpose:** Establishes a session with the Microsoft Compliance Center.
	-   **Access:** Requires at least Global Reader for read-only metadata; higher roles for full access.
3. Get-InformationProtectionSettings
	-   **Purpose:** Collects all label, policy, and compliance configuration metadata.
	-   **Access:**
		-   **Compliance Admin/Global Admin:** All metadata
		-   **Global Reader:** All except Insider Risk Management
4. EvaluateSensitivityLabels
	-   **Purpose:** Evaluates and tests only published sensitivity labels for compliance with naming conventions.
	-   **Access:** No special access; operates on already-fetched metadata.
5. ExtractColumns
	-   **Purpose:** Flattens and selects specific columns from complex objects for Excel export.
	-   **Access:** No special access; operates on already-fetched metadata.
6. ExtractLabelActions
	-   **Purpose:** Extracts and flattens label action details for reporting.
	-   **Access:** No special access; operates on already-fetched metadata.
7. GenerateExcelFromJSON
	-   **Purpose:** Converts the processed JSON metadata into an Excel workbook with multiple tabs (one per metadata type).
	-   **Access:** No special access; operates on already-fetched metadata.

### Output Files

-   **JSON File:** Contains all extracted metadata in a structured format.
-   **Excel File:** Contains the following tabs (if data is available):
	-   TenantDetails
	-   GetLabel
	-   GetLabelPolicy
	-   SensitivityLabelTests
	-   LabelActions
	-   GetRetentionCompliancePolicy
	-   GetDlpCompliancePolicy
	-   GetComplianceTag
	-   InsiderRiskManagement (if permitted)
	-   GetAutoSensitivityLabelPolicy

### Known Limitations
-   **Insider Risk Management Data:**  
    Not available to users with only the Global Reader role.
-   **No User Content:**  
    The script only extracts configuration metadata, not user emails, files, or other sensitive content.
-   **Module Availability:**  
    The script assumes required modules are installed and available.


### Risk Profiling using Co-Pilot
1. Credential Handling
	- User credentials are only requested via `Read-Host` for UPN; no passwords are handled or stored in the script.
	- No credentials are written to disk or logged.
	- No hardcoded credentials.

2. Data Handling

	- Only metadata is collected (labels, policies, tags, etc.), not user content or sensitive emails/files.
	- Output is written to JSON and Excel files in the user’s local profile directory (`$env:LOCALAPPDATA\Microsoft\PurviewConfigAnalyser`).
	- No data is sent to external servers or third parties.
	- No network communication except for connecting to Microsoft 365 services via official modules.

3. Module Installation
	- The script will install missing modules (`ImportExcel`, `ExchangeOnlineManagement`) if not present. This is standard, but users should be aware that it uses the PowerShell Gallery as a source.

4. File System Access
	- Creates directories and writes files in the user’s local app data folder.
	- Deletes temporary/preprocessed JSON files after use.
	- No access or modification to system or sensitive files outside its working directory.
5. Logging
	- Logs are written to a file in the user’s local app data folder.
	- No sensitive data is logged (only operational messages, errors, and warnings).
6. Command Execution
	- No use of `Invoke-Expression`, `Start-Process`, or other risky dynamic execution.
	- No registry or system configuration changes.
	- No elevation of privileges or use of `RunAs`.
7. Error Handling
	- Errors are caught and logged.
	- No error messages are sent externally.

8. User Interaction
	- Prompts for UPN only; does not prompt for or handle passwords directly.
	- 
### Security Summary
 - No risky elements such as credential leaks, external data exfiltration, privilege escalation, or system modifications are present in this script.
- All actions are limited to metadata extraction and local reporting.
- The script is safe to run in a standard administrative environment, provided the user understands it will install modules if missing.
