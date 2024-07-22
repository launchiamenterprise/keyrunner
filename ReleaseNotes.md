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
