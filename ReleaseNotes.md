## V1.0.94 (2026-03-02)

## Enhancements

### 1. Collection-Level Scripts

Collections now support **Pre-Scripts** and **Post-Scripts**, giving you the ability to run JavaScript logic before or after every request in a collection — without having to duplicate script logic across individual requests.

**How it works:**

- Open any collection and navigate to the **Scripts** tab.
- Write a **Pre-Script** to execute logic before each request in the collection fires (e.g., set auth tokens, inject headers, resolve dynamic variables).
- Write a **Post-Script** to execute logic after each request completes (e.g., validate response structure, chain data to the next request, log results).
- Collection-level scripts run in addition to any request-level scripts. The execution order is:
  1. Collection Pre-Script
  2. Request Pre-Script
  3. Request executes
  4. Request Post-Script
  5. Collection Post-Script

**Why it matters:**
Previously, teams had to copy the same boilerplate auth or token-refresh logic into every single request's script. With collection-level scripts, you write it once at the collection and it applies to all requests inside it automatically.

---

### 2. Reusable Global Script Libraries

KeyRunner now supports a **Script Library** — a workspace-level store of reusable JavaScript modules that can be imported into any script (collection scripts, request scripts, or other library scripts).

**How it works:**

- Access the Script Library from the bottom navigation panel.
- Create a new library by clicking the **+** icon and giving it a name.
- Write your reusable functions inside the library and export them using Node.js module syntax:

```javascript
// my-library
function getAuthHeader(token) {
  return `Bearer ${token}`;
}

module.exports = { getAuthHeader };
```

- To use the library in any script, import it with `kr.require`:

```javascript
const myLib = await kr.require('my-library');
const header = myLib.getAuthHeader(kr.environment.get('token'));
kr.request.setHeader('Authorization', header);
```

- The **Libraries** picker button (shown in the script editor toolbar) lets you browse and insert the `kr.require` import statement without typing it manually.
- Use the **copy** icon next to the library name to copy it to your clipboard for quick referencing.
- Libraries are scoped to your workspace and automatically available to all members.

**Why it matters:**
Eliminates duplication of utility functions like token generators, retry helpers, and response validators. One change to the library propagates everywhere it is used.

---

### 3. Kafka Integration

KeyRunner now supports **Apache Kafka** as a first-class connection type, enabling you to produce and consume messages from Kafka topics directly from the app.

#### Getting Started

1. Access **Kafka** from Connectors in bottom pannel.
2. Enter a **name** for your connection and the **broker addresses** (comma-separated, e.g., `broker1:9092, broker2:9092`).
3. Configure authentication if your cluster requires it (see below).
4. Click **Connect** to establish the connection.
5. Once connected, you can produce and consume messages from topics.

#### Producing Messages

- Switch to the **Message** tab.
- Select an existing topic from the dropdown (populated automatically when you connect) or click **Add Topic** to create a new one.
- Choose the message format — **JSON** or **TEXT** — and compose your message in the editor.
- Click **Send** to publish the message to the selected topic.

#### Consuming Messages

- The **Response** panel (bottom half of the split view) shows incoming messages from a subscribed topic in real time.
- Select a topic from the consumer dropdown to start receiving messages.
- Use the **Clear Logs** button to reset the message stream.

#### Supported Authentication Mechanisms

KeyRunner supports the full range of Kafka SASL authentication options:

| Mechanism         | Description                                              |
| ----------------- | -------------------------------------------------------- |
| **None**          | Unauthenticated / plaintext brokers                      |
| **PLAIN**         | Username + Password over SASL/PLAIN                      |
| **SCRAM-SHA-256** | Username + Password using SCRAM-SHA-256                  |
| **SCRAM-SHA-512** | Username + Password using SCRAM-SHA-512                  |
| **AWS IAM (MSK)** | AWS Managed Streaming for Kafka using IAM authentication |

**AWS IAM (MSK) — Profile Mode:**
Uses your local AWS credential chain (environment variables, `~/.aws/credentials`, or AWS SSO). Optionally specify a named AWS CLI profile. No credentials are stored in KeyRunner — a short-lived signed token is generated on each connection.

```
# Setup steps:
1. Install AWS CLI:  brew install awscli
2. Configure profile:  aws configure --profile my-msk-profile
3. For SSO:  aws configure sso --profile my-msk-profile
            aws sso login --profile my-msk-profile
4. Ensure IAM role has kafka-cluster:Connect permission on the MSK cluster ARN
5. Enter region and (optionally) profile name in KeyRunner
```

**AWS IAM (MSK) — Access Keys Mode:**
Provide an IAM Access Key ID, Secret Access Key, and optional Session Token (for STS assumed roles). Credentials are stored encrypted on disk. A signed token is generated per connection — your keys are never sent directly to the broker.

**SSL:**
SSL is automatically enabled when selecting AWS IAM. For other mechanisms, SSL can be configured independently.

---

### 4. NTLM Authentication

KeyRunner now supports **NTLM** authentication for HTTP requests, enabling testing of APIs secured by Windows-integrated authentication commonly used in enterprise and Active Directory environments.

**How to use:**

1. Open any HTTP request.
2. Navigate to the **Authorization** tab.
3. Select **NTLM** from the authentication type dropdown.
4. Fill in the required fields:

| Field           | Description                                      |
| --------------- | ------------------------------------------------ |
| **Username**    | The Windows/AD username                          |
| **Password**    | The user's password                              |
| **Domain**      | The Windows domain (required for most AD setups) |
| **Workstation** | The workstation name (optional)                  |

5. Send the request — KeyRunner handles the NTLM handshake (challenge/response negotiation) automatically.

**Why it matters:**
NTLM is widely used in corporate environments, particularly for internal APIs, SharePoint, and legacy Windows services. Until now, testing these endpoints required external tools. Resolves [GitHub issue #127](https://github.com/launchiamenterprise/keyrunner/issues/127).

---

### 5. Workspace Admin: Enable / Disable Collection Export

Workspace admins of **Private** and **Project** workspaces can now control whether members are allowed to export collections.

**How to use:**

1. Open your workspace settings (click the workspace name → **Manage Workspace**).
2. Under **Workspace Settings**, find the **Allow Collection Export** toggle.
3. Toggle it on or off:
   - **On** — workspace members can export collections in all formats (JSON, Postman, OpenAPI, etc.).
   - **Off** — export options are hidden/disabled for non-admin members.

**Who can change this?**
Only workspace members with the **ADMIN** role can see and modify this setting. It is available on **Private** and **Project** workspace types.

**Why it matters:**
Organizations with strict IP or data policies can now prevent unauthorized export of API definitions and test collections, while still allowing members to collaborate on requests. Resolves [GitHub issue #123](https://github.com/launchiamenterprise/keyrunner/issues/123).

---

## Bug Fixes

### Validate Request Now Respects Variable Values (Issue #121)

**Problem:** When using **Validate Request**, the validator was treating variable references (e.g., `{{base_url}}`) as literal strings instead of resolving them to their actual values from the active environment. This caused false validation failures for URLs, headers, and body fields that used environment variables.

**Fix:** The validation engine now resolves all variable references against the active environment before running validation checks. Variables defined in the environment, collection, or global scope are correctly substituted before validation is applied.

Resolves [GitHub issue #121](https://github.com/launchiamenterprise/keyrunner/issues/121).

---

### Postman Collection Import: Request URL Now Included (Issue #120)

**Problem:** When importing a KeyRunner JSON collection into Postman format, the exported file was missing the `url` field on requests. This caused imported requests in Postman to appear with blank URLs, requiring manual re-entry.

**Fix:** The Postman collection exporter now correctly maps the request URL (including path, query parameters, and protocol) to the Postman `url` object format in the exported JSON.

Resolves [GitHub issue #120](https://github.com/launchiamenterprise/keyrunner/issues/120).

---

## Improvements

### Performance: Collection Export for Private and Project Workspaces

Significant performance improvements have been made to the collection export pipeline for Private and Project workspaces. Collections with large numbers of requests, nested folders, and environment variables now export substantially faster, with reduced memory overhead.

### OpenAPI Spec and Postman Collection Import / Export

Multiple reliability and fidelity improvements across the import/export pipeline:

- **OpenAPI import** — Improved parsing of complex schema types, `$ref` resolution, and multi-server configurations.
- **Postman collection import** — Better handling of Postman v2.1 collection structures including pre-request scripts, test scripts, and nested folder hierarchies.
- **OpenAPI export** — More accurate mapping of KeyRunner request parameters to OpenAPI 3.x schema definitions.
- **Postman collection export** — Fixed edge cases where auth configurations and collection variables were not correctly mapped to the Postman format.

---

## v1.0.93 (2025-12-20)

## Enhancements
- Added support to **publish mock servers for external use** with secure, controlled access.
  - Mock servers in **Private** and **Public** workspaces can be published.
  - Mock servers in **Local** workspaces cannot be published to ensure data remains local.
  - Mock servers are **not published by default**.
  - A **Workspace Admin or Editor** must explicitly publish a mock server.
  - Once published:
    - The mock server URL becomes publicly accessible.
    - A secure access token is generated.
    - Requests must include the header `x-api-token: <your-token>`.
  - Each published mock server includes **Token Management**, allowing:
    - Token creation
    - Token revocation
    - Token rotation
- Added **authorization support for WebSocket requests**.
  - Authorization headers can be included during the WebSocket handshake.
  - Supports common authentication methods such as bearer tokens.
  - Removes the need for external workarounds when testing authenticated WebSockets.

## Bug Fixes
- Applied multiple **bug fixes** across the platform.
- Improved overall **stability, performance, and reliability**.

---
## v1.0.92 (2025-11-18)

## Enhancements
- Added integration with **1Password** for smoother credential management.
- Introduced a **search feature** for environment variables to improve navigation and usability.

## Bug Fixes
- Prevented imports of collections and environments when the **workspace field is empty**.
- Added **line numbers** for scripts in console logs to aid debugging.
- Ensured **collections and requests are removed** when a workspace is deleted.
- Corrected the **delay issue** in code snippets.
- Addressed several **UI related bugs** and applied minor improvements.

---
## v1.0.91 (2025-11-13)
## Bug Fixes
- Addressed several **UI related bugs** and applied minor improvements.

---
## v1.0.90 (2025-10-13)

## Enhancements
- Added ability to **rearrange the order** of query parameters and headers.  
- Implemented **column resizing** support for query parameters and headers.  
- Introduced **warning indicators** for duplicate keys in query parameters or headers.  
- Upgraded **Node.js version**.  

## Security
- Applied additional **security hardening measures** in client and API layers.  
- Updated dependencies.  

## Bug Fixes
- Fixed issue where **unselected query parameters** were still appearing in the generated URL.  

---
## v1.0.89 (2025-09-25)

## Bug Fixes
- Addressed multiple **security-related fixes** across the client and backend.  
- Resolved issue [#122](https://github.com/launchiamenterprise/keyrunner/issues/122)
- Resolved issue [#116](https://github.com/launchiamenterprise/keyrunner/issues/116) 
- Resolved issue [#72](https://github.com/launchiamenterprise/keyrunner/issues/72) 
- Partially fixed [#111](https://github.com/launchiamenterprise/keyrunner/issues/111): (completed fixes are reflected in the issue checklist).  
- Partially fixed [#5](https://github.com/launchiamenterprise/keyrunner/issues/5): (only the resolved checklist items are included in this release).  

## Security
- Completed **Vulnerability Assessment and Penetration Testing (VAPT)** of the infrastructure and application.  
- Closed high-priority findings from recent security testing, further strengthening platform resilience.  

---
## v1.0.88 (2025-09-16)

## Bug Fixes
- Added support for **empty keys in Basic Auth**, ensuring broader compatibility with edge cases.  
- Fixed **corporate firewall login issues** by filtering out expired system certificates.  
- Improved security by preventing **secrets from being displayed in the console** when passed as query parameters or path variables.  

## Features
- Released **Playground V2**, now supporting conditions, loops, and scripts for more advanced workflows.  
  
---

## v1.0.86 (2025-08-25)

## Bug Fixes
- Fixed issue with **VSCode login from behind corporate firewalls**, ensuring authentication works seamlessly in restricted environments.  

## Features
- Substantially reduced **desktop app build size**, improving installation speed and storage efficiency.  

## Internal
- Upgraded **Kubernetes** and **Kafka** clusters to latest stable versions for improved reliability and performance.  
- Streamlined **automation processes** using Terraform for more consistent and maintainable infrastructure management.  

---

## v1.0.85 (2025-08-18)

## Bug Fixes

- Fixed issue with **authorization inheritance** – Fixed authorization inheritance issue to display correct error message instead of failing silently.  
- Resolved **folder permissions** inconsistencies.  
- Fixed issue with **environment variable refresh** during insertion from scripts.  
- Corrected behavior for **query search params overwrite** – editing at the URL level no longer erases unselected query params, ensuring documentation remains consistent in shared collections.  
- **[Enterprise]** Fixed **filter and sort** functionality in **Private/Project workspace Manage Users**.  
- **[Enterprise]** Resolved **duplicate workspace name issue** – users can no longer create workspaces with identical names.  
- Fixed **OpenAPI Spec export** issue
  ([#119](https://github.com/launchiamenterprise/keyrunner/issues/119)).  
- **[Enterprise]** Fixed issue where **KCSV was not refreshed** when switching projects.  
- **[Enterprise]** Resolved issue when **assigning a role with filters applied** – dropdown now correctly allows role selection even when a filter is active.  

## Features

- Added **API_KEY** option for authentication
([#118](https://github.com/launchiamenterprise/keyrunner/issues/118)).  
- Added ability to **export test results in Playground**.  
- Added option to **copy logs from console**.  

## Internal

- Added **CI pipeline step** to run unit and integration test cases.  
- Expanded **integration test coverage** to improve stability.  
- Added more **end-to-end (e2e) test cases** to validate the application.

---  
## v1.0.84 (2025-08-06)

### Features
- **Playground V2**  
  Introduced the next iteration of the Playground with UI and UX enhancements.

- **Environment Variable Suffix Removed**  
  Variable syntax no longer requires `::SensitiveEnv` or `::NonSensitiveEnv` suffixes.  
  Backward compatible: existing variables using old suffixes will still resolve correctly. Applies to environment variables, collection variables, and request flow variables.

- **Improved Collection Run Output**  
  Collection run now includes a “View Test Results” option. Minor UI changes introduced to support this.

- **Resolved Variable Expansion in Code Snippets**  
  When environment variables are used in the request base URL, code snippets now show the resolved full URL.

### Bug Fixes

**Enterprise Version**
- Fixed missing scroll bar in logs view.
- “Enable Git” button is now visible at the bottom.
- Fixed infinite loading issue when request flow validation fails.
- Fixed layout issue where search bar breaks when side panel is resized to minimum width.  
  [#117](https://github.com/launchiamenterprise/keyrunner/issues/117)
- Fixed bug where variables were not resolving as expected in recent versions.  
  [#113](https://github.com/launchiamenterprise/keyrunner/issues/113)
- Resolved issue where all Playground actions ran at once instead of sequentially.  
  [#112](https://github.com/launchiamenterprise/keyrunner/issues/112)
- Fixed Playground flow bug causing incorrect execution behavior.  
  [#110](https://github.com/launchiamenterprise/keyrunner/issues/110)
- Added beautifier for JSON request body to improve readability.  
  [#82](https://github.com/launchiamenterprise/keyrunner/issues/82)

**Non-Enterprise Version**
- N/A

### Internal
- Partial enhancements implemented for flow actions.  
  [#111](https://github.com/launchiamenterprise/keyrunner/issues/111)
- Closed as intended behavior:  
  [#102](https://github.com/launchiamenterprise/keyrunner/issues/102)  
  [#16](https://github.com/launchiamenterprise/keyrunner/issues/16)

---

## v1.0.83 (2025-07-14)

### Features
- **Magic Link Based SSO**  
  Support added for passwordless sign-in using magic links.

### Bug Fixes

**Enterprise Version**
- Fixed issue where users couldn’t be removed from a project once added.
- “Manage” button now correctly appears for Project Admins and Editors.
- Added filter and search capability for user management.
- Fixed bug in delete request flow.

**Non-Enterprise Version**
- Beautify request body for better readability.
- mTLS support now accepts URLs via environment variables.

### Internal
- CI/CD pipeline added for releasing new app versions.
- Improved UI test automation coverage.

---

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
