# v1.0.82 (2025-05-27)

## Features

- **Collection Export - Variables**  
Users can now export variables along with collections.

- **Workspace-Aware Secret Scanner**  
The secret scanner now correctly scopes scanning to the selected workspace instead of defaulting.

## Enterprise Only

- **Admin Dashboard (UI + API)** 
Introduced a new admin dashboard to help manage tenants and system-wide configurations more efficiently.

- **Tenant Scoped Redacted Strings**  
Tenant admins can now manage redacted strings at the tenant level.

- **Data Masking Enhancements for Projects**  
Projects now combine both project-level and tenant-level redacted strings for accurate data masking.

- **SAML Session Token API**  
A new endpoint for exchanging SAML tokens and receiving session tokens.

- **Environment Variables Tracking**  
Now tracks and reports the number of environment variables vs KCSV variables used per user.

- **Ephemeral Adhoc KeyConnector Responses**  
Adhoc request response data (including logs) is now auto-deleted after 5 minutes to reduce data persistence risks.

---
# v1.0.81 (2025-04-23)

## Features

- **Timeline for Requests, Workspaces, and Projects - Enterprise only**  
  Added a unified timeline view to track activity across requests, workspaces, and projects.

- **Collection-Level OWASP Validator**  
  Built-in security validator at the collection level to help catch common OWASP issues early.

## Enhancements

- **Secrets Access Logging - Enterprise only**  
  Secret access logs now include the user ID for better traceability.

---
# v1.0.80 (2025-04-16)

## Features

- **Magic Link Login for Non-SSO Tenants - Enterprise only**  
  Introduced magic link authentication flow for users not using SSO — simpler, secure access.

## Enhancements

- **Custom Libraries in Scripts**  
  Users can now import and use their own libraries in scripts for more flexibility.

---

# v1.0.79 (2025-04-09)
## Features

- **Environment Variables Sharing**  
  Added ability to share non-sensitive environment variables.

- **Sensitive Values Protection**  
  Env vars are now obfuscated by default after saving. Users can toggle visibility with a "peek" button.

##  Bug Fixes

- **Vault Connection**  
  Fixed issue where the app failed to connect to Google Secret Manager if any secret was blank.  
  Error thrown: *“invalid secret”* — now resolved.

## Enterprise Fixes

- **Workspace Dropdown Improvements**  
  Workspace dropdown now displays both name and scope (local/project) for better clarity.

- **Console Output Feedback**  
  Initial tuning based on early feedback. Will iterate further with more concrete suggestions.
---



## v1.0.78 (2025-04-08)
### Enhancements:
- OWASP API Top 10 checks: Validate requests against common security issues
- Minor UI enhancements.
- Added a modal for reporting issues on Github directly from home screen.

---

## v1.0.77 (2025-04-03)
### Bug Fix:
- Hotfix for issue with running multiple iterations in a collection.

---

## v1.0.76 (2025-03-27)
### Enhancements:
- Sensedia API management Integration.
- Vault enhancement: Support for multiple vaults.
- Git enhancements: Commit changes, fetch, and clone.
- Support for nested variables in imported collections.
- Git integration now supports branch selection.

### Bug Fixes:
- Fixed issue with auto-updater.

---
# v1.0.74 (2025-01-21)
## New Features
- **User Management**:  
  Introduced user management functionality to enhance user role and access control.
- **Test Functionality in Scripts**:  
  Added the ability to include pre and post-test scripts .(https://github.com/launchiamenterprise/keyrunner/issues/99)
- **SSL Verification**:  
  Implemented support for using system certificates for SSL verification.

## Bug Fixes
- **Environment Variables**:  
  Fixed an issue where null values were not allowed in environment variable values.(https://github.com/launchiamenterprise/keyrunner/issues/100)

--- 
# v1.0.72 (2024-11-15)
## Patch
https://github.com/launchiamenterprise/keyrunner/issues/95

# v1.0.71 (2024-11-05)
## New Features
- **Support Collection Run**: Added the ability to run entire collections, enabling users to execute multiple requests in sequence.
- **Playground Variables**: Introduced support for playground variables, enhancing flexibility for customization and testing.
- **Pre and Post Scripts in Playground**: Enabled pre and post-request scripts in the playground to allow more complex workflows and conditional testing.

## Enhancements
- **Console UI with JSON Logs**: Upgraded the console UI to support JSON-formatted logs, providing a structured view for easier debugging and analysis.
- **AWS Secret Store Integration**: Added a `secretName` field for AWS secret store functionality.

## Bug Fixes
- **Cookie Functionality**: Setting Cookies for HTTP response of 302 status code.

---
# v1.0.70
## Bug Fixes
1. **Console Error Resolved**: Fixed an error that appeared in the console after executing a request in the previous version - v1.0.69. 
2. **History Feature minor bug**: The "History" feature in the left navigation created an empty json file in an edge case which is now fixed. 
3. **Request Body Variables Update**: Addressed the issue where request body variables were not reflecting the latest values set through Pre-Script execution.

## UX Improvements
1. **Enhanced Request Tab Visual Indicators**: Added additional visual status indicators to the request tabs. These indicators will now clearly show when a tab contains data.

---
# v1.0.69 (2024-10-21)
## Features:
1. **Collections' Folder State**: The state of opened folders within Collections is now retained.
2. **Version Check**: Added an easy way to check for new version releases and their release notes.
3. **JSON Request Comments**: Added support for commenting out specific JSON parameters in the request body.
4. **Tab Status Indicators**: Introduced status indicators to highlight when a request tab contains data.
5. **Console Enhancements**: Various improvements and enhancements to the Console for a better user experience.
6. **OAuth Redirect Update**: OAuth redirect is now supported via the following endpoint: https://api.keyrunner.app/v1/oauthRedirect
7. **Script Functionality**: General improvements to scripting functionality.
   
## Bug Fixes:
1. **Request Body Variables**: Resolved issue where variables were not taking the latest values set through Pre-Script execution.
2. **Pre-Script Performance**: Improved performance of  Pre-Script execution.
3. **Pre-Script JS Error Reporting**: Fixed issue where Pre-Script JS execution errors were not showing in the Keyrunner Console (now includes error details like JS error and line number).
4. **OAuth Redirect**: Fixed issue where OAuth redirect was not accepted by the identity provider.
5. **Postman Collection Import**: Resolved issue where importing a Postman collection didn’t import the included collection’s variables.
6. **Empty Environment Variables**: Fixed bug that generated thousands of empty key/value environment variables after running a request containing Pre-Script.
7. **Incorrect URL**: Corrected an issue where URLs were incorrect when one of the parameter values contained an `=` as a substring.

---
# v1.0.68 (2024-10-11)
## Bug Fixes:
1. **"Check for Updates"**: Fixed broken functionality in the "KeyRunner" menu's "Check for Updates" option.
2. **Nav Dropdown Labels**: Restored missing labels on the two main centered select boxes (dropdowns) in the top navigation.

---
# v1.0.64 (2024-10-01)
### New Features
#### Secret Scanner:
Introducing Secret Scanner to help identify and manage secrets such as API keys, tokens, and credentials within KeyRunner files. The scanner automatically detects sensitive information and displays it in the UI for easy remediation and management.

#### Collection Level Variables:
You can now define and manage variables at the collection level. This allows better control and reuse of variables across your workspaces, improving efficiency and consistency in managing environments.

#### GCP, Azure, AWS Secret Store Connectors:
Added support for Google Cloud Platform (GCP), Azure, and AWS Secret Manager connectors. These integrations allow you to securely manage and access secrets from these cloud providers directly within KeyRunner.

### Bug Fixes
#### Certificate Management - Minor UI Fixes
Fixed several minor UI inconsistencies in the Certificate Management module for a smoother user experience.

#### Multiple Sub-Collections - GUI Issue
Resolved an issue where adding multiple sub-collections would cause the ellipsis menu to disappear. The ellipsis now appears correctly in all scenarios.

#### PUT Call Using Binary Fails
Fixed an issue where making a PUT call with binary data would return an error ("Something went wrong"). This functionality now works as expected.

#### Imported Environment - Unable to Add Variables
Resolved a bug preventing users from adding new variables to imported environments.

---
# v1.0.60

### New Features
Theming: Introduced a theming feature to customize the appearance.
Request Timeout Settings: Added settings for request timeouts at both request level and global level.
Export Responses: Ability to export responses from a single response or playground.

### Enhancements:
VS Code Extension: Open the app directly from the activity bar instead of the sidebar.
Response Body Rendering: Now rendering the response body even when the server returns only an HTTP status code.

### UI Improvements:
Expandable Body: Adjusted body expansion to use viewport height (VH) for better readability.
---

# v1.0.57

### New Features
- **Added mTLS Certificate-Based Authentication**
  - Enhances security by requiring mutual TLS authentication for more robust client-server communication.

### Bug Fixes
- **Resolved Content-Length Issue**
  - Fixed an issue where `text/xml` Content-Type POST requests were incorrectly passing a Content-Length of 0.


# V1.0.56
### New Features:
- Expanded HTTP Methods: Added support for more HTTP methods.
- Environment Variables Overlay: New overlay panel to view environment variable values.
- Scripting Import Capability: Import functionality for pre and post scripts.
- Parent Level Authorization: Feature to add parent level authorization in the playground.
- Export Flows in playground

### Bug Fixes:
- Content-Type Header: Resolved issue where the Content-Type defaulted to "text/plain".
- Request/Response Headers: Addressed various issues causing runtime errors.
- URL Suggestions: Fixed the issue with filtering duplicate URL suggestions.
- Binary File Upload: Resolved issue where the file was missing after selection.
- Favorite Collections: Fixed favoriting collections, bringing them to the top of the collections tab.


# V1.0.53
- Scripting (pre-script & post-script) --> https://docs.keyrunner.app/docs/scripting.html
- KeyRunner console logs
- Hashicorp Vault Integration --> https://docs.keyrunner.app/docs/hashicorp.html
- Enhancement to playground nodes (Ability to access conected node's response in request body) --> https://docs.keyrunner.app/docs/Flows.html

# V1.0.45
- Add Swagger import capabilities
- Improve documentation UX & UI
  

# V1.0.44
- Enterpise features for schedulers
- Logging feature - Access logs, Audit Logs
- Theme changes, Minor CSS changes
- Params description tags for all input fields
- Private API network
