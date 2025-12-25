# DRUO n8n Node - Verification Checklist & Project Timeline

**Date:** December 25, 2025  
**Purpose:** Final verification checklist, timeline, and submission guidelines

---

## 1. PRE-DEVELOPMENT REQUIREMENTS CHECKLIST

Before starting implementation, gather these from DRUO:

### API Information
- [ ] Base URL for Sandbox: `https://sandbox.druo.com` or similar
- [ ] Base URL for Production: `https://api.druo.com` or similar
- [ ] Authentication method documented
  - [ ] API Key (where in request? Header? Query? Body?)
  - [ ] Bearer Token (standard Authorization header?)
  - [ ] OAuth2 (if applicable)
- [ ] Complete endpoint list with:
  - [ ] HTTP method (GET, POST, PATCH, DELETE)
  - [ ] URL path (e.g., `/payment-methods`, `/transactions`)
  - [ ] Required headers
  - [ ] Request payload schema (JSON)
  - [ ] Response schema (JSON)
- [ ] Error codes and messages documented
- [ ] Rate limiting documented (if any)
- [ ] Pagination method (if applicable)

### Test Credentials
- [ ] Sandbox API key provided
- [ ] Sandbox account IDs for testing
- [ ] Test card numbers (if needed)
- [ ] Access to DRUO Dashboard for credential management

### Business Requirements
- [ ] Final list of Payment Method operations
  - [ ] Create (what fields required/optional?)
  - [ ] Get
  - [ ] List (pagination? filtering?)
  - [ ] Update (what fields can be updated?)
  - [ ] Delete
- [ ] Final list of Transaction operations
  - [ ] Create (charge/payment submission)
  - [ ] Get
  - [ ] List (pagination? filtering?)
  - [ ] Refund (full/partial?)
- [ ] Any other resource operations (Account info, Settlements, etc.)?
- [ ] Special requirements or edge cases?

---

## 2. DEVELOPMENT ENVIRONMENT SETUP

### 2.1 Prerequisites

```bash
# Check Node.js version (need 16+)
node --version

# Check npm version (need 7+)
npm --version

# Install n8n-node globally (optional but recommended)
npm install -g n8n-node

# Verify installation
n8n-node --version
```

### 2.2 Create Project Directory

```bash
# Create using n8n-node
npm create @n8n/node@latest n8n-nodes-druo

# OR install globally and use:
n8n-node new n8n-nodes-druo

# Navigate to project
cd n8n-nodes-druo

# Verify structure created
ls -la
# Should see: src/, dist/, node_modules/, package.json, tsconfig.json, etc.
```

### 2.3 Install Dependencies

```bash
npm install

# Verify TypeScript installed
npx tsc --version
```

---

## 3. IMPLEMENTATION PHASES

### PHASE 1: Setup & Planning (Day 1, ~2 hours)

**Deliverables:**
- ✅ Git repository created (GitHub)
- ✅ Project scaffolded with n8n-node
- ✅ node.json metadata created
- ✅ README skeleton drafted
- ✅ API documentation reviewed and summarized

**Checklist:**
```bash
[ ] Git repo initialized
[ ] n8n-node project created
[ ] Initial commit: "Initial project scaffold"
[ ] Branch created for development: git checkout -b develop
[ ] Node icon added (druo.svg)
[ ] API endpoints documented locally
```

**Commands:**
```bash
npm create @n8n/node@latest n8n-nodes-druo
cd n8n-nodes-druo
git init
git add .
git commit -m "Initial project scaffold"
git branch develop
git checkout develop
npm install
npm run build
```

---

### PHASE 2: Credentials Implementation (Day 1-2, ~3 hours)

**Deliverables:**
- ✅ DruoApi.credentials.ts implemented
- ✅ Credentials support both environments (Sandbox/Production)
- ✅ Credentials support authentication methods
- ✅ Documentation links included

**Checklist:**
```
[ ] DruoApi.credentials.ts created
[ ] Properties defined:
    [ ] Environment (Sandbox/Production)
    [ ] API Key with password: true
    [ ] API Key Type (Bearer/Header)
[ ] TSLint passes for credentials file
[ ] Type definitions correct
[ ] Test with npm run dev (credentials appear in UI)
```

**File:** `src/credentials/DruoApi.credentials.ts`

---

### PHASE 3: Node Base & Structure (Day 2, ~4 hours)

**Deliverables:**
- ✅ Druo.node.ts main class implemented
- ✅ Resource selection parameter added
- ✅ Operation selection parameters for each resource
- ✅ Module imports setup
- ✅ Basic execute() method routing

**Checklist:**
```
[ ] Druo.node.ts created with INodeType interface
[ ] description object includes:
    [ ] displayName, name, icon, group, version
    [ ] subtitle showing operation
    [ ] inputs/outputs defined
    [ ] credentials requirement
    [ ] resource parameter
[ ] Operation parameters for PaymentMethod
[ ] Operation parameters for Transaction
[ ] execute() method routes to correct modules
[ ] Module imports: paymentMethod, transaction
[ ] TSLint passes
[ ] npm run build succeeds
[ ] npm run dev shows node in n8n UI
```

---

### PHASE 4: Payment Method Operations (Days 2-3, ~6 hours)

**Deliverables:**
- ✅ Create.operation.ts - Create payment method
- ✅ Get.operation.ts - Retrieve payment method
- ✅ List.operation.ts - List payment methods
- ✅ Update.operation.ts - Update payment method
- ✅ Delete.operation.ts - Delete payment method
- ✅ index.ts - Module exports and parameter definitions

**Checklist per Operation:**
```
Create:
[ ] Accept Account ID
[ ] Accept Type (card, bank_account, etc.)
[ ] Accept type-specific fields
[ ] Build request body correctly
[ ] Call POST /payment-methods
[ ] Handle errors (400, 401, 409)
[ ] Return response

Get:
[ ] Accept Payment Method ID
[ ] Call GET /payment-methods/{id}
[ ] Handle 404, 401 errors
[ ] Return response

List:
[ ] Accept Account ID
[ ] Accept Limit and Offset
[ ] Build query string correctly
[ ] Call GET /payment-methods?account_id=...
[ ] Handle errors
[ ] Return paginated response

Update:
[ ] Accept Payment Method ID
[ ] Accept fields to update
[ ] Call PATCH /payment-methods/{id}
[ ] Handle 404, 401 errors
[ ] Return updated response

Delete:
[ ] Accept Payment Method ID
[ ] Call DELETE /payment-methods/{id}
[ ] Handle 404, 401 errors
[ ] Return success message
```

**Testing:**
```bash
npm run dev
# Test each operation in n8n UI with sandbox credentials
# Verify parameters appear correctly
# Check error handling with invalid inputs
```

---

### PHASE 5: Transaction Operations (Days 3-4, ~6 hours)

**Deliverables:**
- ✅ Create.operation.ts - Submit transaction
- ✅ Get.operation.ts - Get transaction status
- ✅ List.operation.ts - List transactions
- ✅ Refund.operation.ts - Refund transaction
- ✅ index.ts - Module exports and parameter definitions

**Checklist per Operation:**
```
Create:
[ ] Accept Account ID
[ ] Accept Payment Method ID
[ ] Accept Amount (in cents)
[ ] Accept Currency
[ ] Accept optional Description
[ ] Validate amount > 0
[ ] Build request body
[ ] Call POST /transactions
[ ] Handle errors (400, 401, 402)
[ ] Return transaction response

Get:
[ ] Accept Transaction ID
[ ] Call GET /transactions/{id}
[ ] Handle 404, 401 errors
[ ] Return transaction details

List:
[ ] Accept Account ID
[ ] Accept optional Status filter
[ ] Accept Limit and Offset
[ ] Call GET /transactions with query params
[ ] Handle errors
[ ] Return paginated list

Refund:
[ ] Accept Transaction ID
[ ] Accept optional Refund Amount
[ ] Build refund request
[ ] Call POST /transactions/{id}/refund
[ ] Handle errors
[ ] Return refund response
```

**Testing:**
```bash
npm run dev
# Test each operation in n8n UI
# Create payment method → Create transaction flow
# Test refund operation
# Verify all parameters work correctly
```

---

### PHASE 6: Validation & Error Handling (Day 4, ~3 hours)

**Deliverables:**
- ✅ All operations have try/catch blocks
- ✅ Meaningful error messages for common errors
- ✅ Input validation for required fields
- ✅ Proper HTTP status code handling
- ✅ No console.logs in production code

**Checklist:**
```
[ ] All operations wrapped in try/catch
[ ] 401 errors: "Authentication failed. Check credentials."
[ ] 400 errors: "Invalid request: [DRUO message]"
[ ] 404 errors: "[Resource] not found"
[ ] 402/decline errors: "Transaction declined by processor"
[ ] Required parameters validated before use
[ ] Amount validation (must be > 0)
[ ] Card number validation if present
[ ] No sensitive data logged
[ ] nodeOperationError used for all errors
[ ] Error messages are English
```

---

### PHASE 7: Code Quality & Linting (Day 4, ~2 hours)

**Deliverables:**
- ✅ All code passes linter
- ✅ No TypeScript errors
- ✅ No console warnings
- ✅ Consistent code style

**Checklist:**
```bash
[ ] npm run lint passes
[ ] npm run lint:fix applied if needed
[ ] npm run build succeeds
[ ] No TypeScript errors
[ ] npm run dev works without errors
[ ] Code follows n8n standards
[ ] All files in src/ are TypeScript (.ts)
```

**Commands:**
```bash
npm run lint
npm run lint:fix  # If needed
npm run build
npm run dev       # Final test run
```

---

### PHASE 8: Documentation (Day 5, ~2 hours)

**Deliverables:**
- ✅ Comprehensive README.md
- ✅ Example workflows (JSON exports)
- ✅ Installation instructions
- ✅ Configuration guide
- ✅ Troubleshooting section

**README Sections:**
```
[ ] Installation (UI and npm)
[ ] Configuration (credential setup)
[ ] Operations table (all resources/operations)
[ ] Parameter descriptions
[ ] Example workflows (2-3 realistic examples)
[ ] API documentation links
[ ] Troubleshooting (common errors)
[ ] Support/contact information
[ ] License (MIT)
```

**Example Workflows to Create:**
```json
Workflow 1: Create Payment Method
- Trigger: Manual
- Add DRUO node: Create Payment Method
- Show sample inputs and outputs

Workflow 2: Create Payment Method + Transaction
- Trigger: Manual
- Add DRUO node: Create Payment Method
- Add DRUO node: Create Transaction (use PM from step above)
- Show complete flow

Workflow 3: List and Refund
- Trigger: Manual
- Add DRUO node: List Transactions
- Add DRUO node: Refund Transaction
```

---

### PHASE 9: Final Testing & Preparation (Day 5, ~2 hours)

**Deliverables:**
- ✅ All operations tested end-to-end
- ✅ Project linting passes
- ✅ Project builds successfully
- ✅ README complete
- ✅ Example workflows created
- ✅ Git history clean

**Testing Checklist:**
```
[ ] Create payment method - works
[ ] Get payment method - works
[ ] List payment methods - works
[ ] Update payment method - works (if supported)
[ ] Delete payment method - works
[ ] Create transaction - works
[ ] Get transaction - works
[ ] List transactions - works
[ ] Refund transaction - works
[ ] Error handling tested (invalid inputs)
[ ] Credentials validation works
[ ] Environment switching (Sandbox/Prod) works
```

**Pre-Publication Checklist:**
```bash
[ ] npm run lint  # Passes
[ ] npm run build  # Succeeds
[ ] npm run dev    # No errors
[ ] git log        # Clean history
[ ] package.json   # All metadata correct
[ ] package.json   # Keywords include n8n-community-node-package
[ ] LICENSE file   # MIT present
[ ] README.md      # Complete and accurate
[ ] node.json      # Icon and description
[ ] src/           # Only TypeScript files
[ ] .gitignore     # Proper entries
```

---

## 4. PUBLICATION TIMELINE

### Days 1-5: Development & Testing
- **Day 1**: Setup, Planning, Credentials (5 hours)
- **Day 2**: Node Structure, Payment Method start (7 hours)
- **Day 3**: Payment Methods complete, Transactions start (10 hours)
- **Day 4**: Transactions complete, Testing, Linting (5 hours)
- **Day 5**: Documentation, Final Testing, Publish (4 hours)

**Total**: ~31 hours of development work

### Publication Day (Day 5)

```bash
# Final verification
npm run lint
npm run build
npm run dev

# Update version in package.json (if needed)
npm version patch

# Login to npm
npm login

# Release to npm (automated)
npm run release

# Or manual release
npm publish
```

### Post-Publication (Day 6)

- [ ] Verify npm package published: `npm info n8n-nodes-druo`
- [ ] Test installation: `npm install n8n-nodes-druo` in clean environment
- [ ] Verify in n8n: Settings → Community Nodes → Search "druo"
- [ ] Create GitHub release
- [ ] Announce on n8n community forums

### Optional Verification (Days 7-10)

- [ ] Run verification scanner: `npx @n8n/scan-community-package n8n-nodes-druo`
- [ ] Fix any issues identified
- [ ] Submit to n8n Creator Portal
- [ ] Respond to n8n review feedback
- [ ] Once approved, available in verified node list

---

## 5. COMPLETE VERIFICATION CHECKLIST

### 5.1 Package Metadata

```
PACKAGE NAME
[ ] Starts with n8n-nodes- or @scope/n8n-nodes-
[ ] Package name: n8n-nodes-druo
[ ] Lowercase, no spaces

PACKAGE.JSON
[ ] name: "n8n-nodes-druo"
[ ] version: "1.0.0"
[ ] description: Clear and concise
[ ] license: "MIT"
[ ] keywords includes n8n-community-node-package
[ ] n8n.nodes points to dist/nodes/Druo/Druo.node.js
[ ] n8n.credentials points to dist/credentials/DruoApi.credentials.js
[ ] repository.url is public GitHub repo
[ ] author is DRUO

LICENSE
[ ] LICENSE file exists in root
[ ] Contains MIT license text
```

### 5.2 File Structure

```
PROJECT ROOT
[ ] src/ directory exists
[ ] dist/ directory exists (after build)
[ ] package.json present
[ ] tsconfig.json present
[ ] .gitignore present
[ ] README.md present
[ ] LICENSE present
[ ] .eslintrc.js present

SRC/NODES
[ ] src/nodes/Druo/Druo.node.ts exists
[ ] src/nodes/Druo/Druo.node.json exists
[ ] src/nodes/Druo/actions/ directory exists
[ ] src/nodes/Druo/actions/PaymentMethod/ with operations
[ ] src/nodes/Druo/actions/Transaction/ with operations

SRC/CREDENTIALS
[ ] src/credentials/DruoApi.credentials.ts exists

SRC/
[ ] src/index.ts exports all nodes and credentials
```

### 5.3 Code Quality

```
TYPESCRIPT
[ ] All source files are .ts (no .js in src/)
[ ] No any types except where necessary
[ ] All imports/exports valid
[ ] No circular dependencies

LINTING
[ ] npm run lint passes without errors
[ ] npm run lint passes without warnings
[ ] No console.log, console.error, console.warn
[ ] Consistent spacing and indentation
[ ] No trailing whitespace

BUILDING
[ ] npm run build succeeds
[ ] No TypeScript compilation errors
[ ] dist/ directory populated correctly
[ ] dist/nodes/Druo/Druo.node.js exists
[ ] dist/credentials/DruoApi.credentials.js exists
```

### 5.4 Credentials Implementation

```
CREDENTIAL CLASS
[ ] Implements ICredentialType interface
[ ] name property set (druoApi)
[ ] displayName property set (DRUO API)
[ ] properties array defined
[ ] documentationUrl included (if applicable)

CREDENTIAL PROPERTIES
[ ] Environment field (Sandbox/Production)
[ ] API Key field with password: true
[ ] API Key Type field (Bearer/Header)
[ ] All descriptions in English
[ ] All options clearly labeled

ERROR HANDLING
[ ] Invalid credentials handled
[ ] Missing required fields flagged
[ ] Type validation present
```

### 5.5 Node Implementation

```
NODE CLASS
[ ] Implements INodeType interface
[ ] description object complete
[ ] displayName set
[ ] name set (lowercase, no spaces)
[ ] icon path correct (file:druo.svg)
[ ] inputs/outputs array defined
[ ] credentials requirement declared
[ ] version set to 1

NODE DESCRIPTION
[ ] displayName readable
[ ] description accurate
[ ] icon file exists and is SVG
[ ] properties array complete
[ ] Resource parameter defined
[ ] Operation parameters per resource

EXECUTE METHOD
[ ] Gets input data safely
[ ] Gets credentials safely
[ ] Routes to correct operation module
[ ] Handles errors with try/catch
[ ] Returns data in correct format
[ ] No modifications to input data
```

### 5.6 Operations Implementation

```
EACH OPERATION FILE
[ ] Exports execute() async function
[ ] Uses this.helpers.httpRequest() only
[ ] Builds correct URL
[ ] Sets correct HTTP method
[ ] Builds request body correctly
[ ] Validates inputs before request
[ ] Handles all error cases
[ ] Returns response data
[ ] No external dependencies

ERROR HANDLING
[ ] 401 errors: Authentication error message
[ ] 400 errors: Invalid request message
[ ] 404 errors: Not found message
[ ] Network errors: Generic error
[ ] API-specific errors: Forwarded message
[ ] Error messages are English
[ ] No sensitive data in errors
```

### 5.7 Documentation

```
README.MD
[ ] Installation instructions (UI and npm)
[ ] Configuration steps with screenshots
[ ] All resources described
[ ] All operations described
[ ] Parameter descriptions complete
[ ] Example inputs and outputs
[ ] Example workflows (2-3)
[ ] Troubleshooting section
[ ] API documentation links
[ ] Support/contact information
[ ] License section (MIT)
[ ] No sensitive examples

EXAMPLE WORKFLOWS
[ ] Valid JSON format
[ ] Realistic use cases
[ ] Can be imported into n8n
[ ] Show parameter values
[ ] Include comments explaining flow

NODE.JSON
[ ] Display name
[ ] Description
[ ] Icon reference
[ ] Any additional metadata
```

### 5.8 Verification-Ready Checks

```
DEPENDENCIES
[ ] Zero external runtime dependencies
[ ] Only n8n-core and n8n-workflow
[ ] devDependencies OK (typescript, linter)
[ ] npm ls --depth=0 shows clean list

ENVIRONMENT & FILESYSTEM
[ ] No process.env access
[ ] No fs or path imports
[ ] No require('fs') or import fs
[ ] No file read/write operations
[ ] All data from node parameters

LANGUAGE
[ ] All UI text in English
[ ] All parameter names in English
[ ] All descriptions in English
[ ] All error messages in English
[ ] No special characters in titles

SECURITY
[ ] API keys not logged
[ ] Sensitive fields marked password: true
[ ] No hard-coded credentials
[ ] No test data in code
[ ] HTTPS used for API calls
```

### 5.9 Pre-Publication Final Check

```bash
# Run complete verification
npm run lint              # Should pass
npm run build            # Should succeed
npm run dev              # Should start clean

# Manual verification
npm info n8n-nodes-druo  # Should not exist yet

# Check package size
npm pack
ls -lh *.tgz             # Should be < 500KB

# Git status
git status               # Should be clean
git log --oneline        # Should have clear history

# Version check
npm version              # Should show 1.0.0 (or desired version)
```

---

## 6. PUBLICATION COMMANDS

### 6.1 Automated Release (Recommended)

```bash
# From project root
npm run release

# This will:
# 1. Build the project
# 2. Run lint checks
# 3. Prompt for version (default: patch)
# 4. Update CHANGELOG.md
# 5. Create git tag
# 6. Create GitHub release
# 7. Publish to npm
```

### 6.2 Manual Release

```bash
# 1. Ensure logged in to npm
npm login

# 2. Build
npm run build

# 3. Lint
npm run lint

# 4. Update version
npm version patch  # or minor, major, prerelease

# 5. Publish
npm publish

# 6. Create git tag
git tag v1.0.0
git push origin main v1.0.0

# 7. Create GitHub release via GitHub web UI
```

### 6.3 Verification After Publication

```bash
# Check npm
npm info n8n-nodes-druo

# Install in test environment
mkdir test-install && cd test-install
npm init -y
npm install n8n-nodes-druo

# Verify files
ls -la node_modules/n8n-nodes-druo/dist/

# Clean up
cd ..
rm -rf test-install
```

---

## 7. OPTIONAL: N8N VERIFICATION SUBMISSION

### 7.1 Pre-Submission Scan

```bash
# Run n8n's verification scanner
npx @n8n/scan-community-package n8n-nodes-druo

# Should show:
# ✓ No external dependencies
# ✓ No environment variable access
# ✓ No filesystem access
# ✓ MIT license
# ✓ English-only text
# ✓ Proper metadata
```

### 7.2 Submit to n8n Creator Portal

1. Visit https://creators.n8n.io/nodes
2. Sign up or log in
3. Click "Submit Node"
4. Fill in:
   - **Package Name:** n8n-nodes-druo
   - **npm URL:** https://www.npmjs.com/package/n8n-nodes-druo
   - **GitHub Repository:** https://github.com/DRUO/n8n-nodes-druo
   - **Description:** n8n community node for DRUO Payments
   - **Contact Email:** support@druo.com
   - **Agree to Verification Guidelines:** Yes

5. Submit for review

### 7.3 Review Timeline

- **Initial Review:** 1-2 weeks
- **Feedback (if any):** Address and resubmit
- **Approval:** Code merged, appears in verified nodes list
- **Availability:** Users can discover in n8n nodes panel

### 7.4 Post-Approval Responsibilities

- Maintain code quality
- Fix security issues promptly
- Respond to user issues on GitHub
- Keep dependencies updated
- Document breaking changes in CHANGELOG

---

## 8. TROUBLESHOOTING GUIDE

### Lint Fails

```bash
# See specific errors
npm run lint

# Auto-fix common issues
npm run lint:fix

# For remaining issues, review:
# https://docs.n8n.io/integrations/creating-nodes/build/reference/code-standards/
```

### Build Fails

```bash
# Clear cache
rm -rf dist/ node_modules/.cache

# Rebuild
npm run build

# Check TypeScript errors
npx tsc --noEmit
```

### npm run dev Fails to Start

```bash
# Clear everything
rm -rf dist/ node_modules/
npm install

# Try again
npm run dev

# Check port 5678 is available
lsof -i :5678  # Kill if needed
```

### Node Doesn't Appear in UI

```bash
# Stop npm run dev (Ctrl+C)
# Clear browser cache (Shift+Reload)
# npm run dev again
# Check browser console for errors (F12)
# Verify package.json has correct n8n.nodes path
```

### API Returns 401

```bash
# Check credentials in n8n UI
# Verify API key is correct
# Verify environment matches (Sandbox vs Production)
# Test API key manually with curl:
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://api.druo.com/v1/payment-methods
```

### npm publish Fails

```bash
# Verify npm login
npm whoami

# If not logged in:
npm login

# Check package.json version is valid semantic versioning
# Ensure version is higher than previously published
# Check package name matches npm registry (no existing)
```

---

## 9. POST-PUBLICATION WORKFLOW

### 9.1 Announce Release

- [ ] Create GitHub release with changelog
- [ ] Post on n8n community forums
- [ ] Tweet/share on social media
- [ ] Email to key stakeholders
- [ ] Add to DRUO documentation

### 9.2 Monitor & Support

- [ ] Monitor GitHub issues
- [ ] Respond to user questions
- [ ] Fix reported bugs promptly
- [ ] Update documentation as needed
- [ ] Plan feature updates

### 9.3 Versioning Guide

```
SEMANTIC VERSIONING: MAJOR.MINOR.PATCH

1.0.0 - Initial release
1.0.1 - Bug fix
1.1.0 - New feature (backward compatible)
2.0.0 - Breaking change
```

**Use `npm version` commands:**
```bash
npm version patch   # 1.0.0 → 1.0.1 (bug fix)
npm version minor   # 1.0.0 → 1.1.0 (new feature)
npm version major   # 1.0.0 → 2.0.0 (breaking change)
```

---

## 10. QUICK REFERENCE

### All Required Commands

```bash
# Setup
npm create @n8n/node@latest n8n-nodes-druo
cd n8n-nodes-druo
npm install

# Development
npm run dev        # Run with n8n and watch for changes
npm run build      # Compile TypeScript
npm run lint       # Check code style
npm run lint:fix   # Fix style issues

# Publishing
npm login          # One-time login to npm
npm run release    # Automated: build, lint, version, publish

# OR manual:
npm version patch
npm publish
```

### Critical Checklist (Before Publishing)

```bash
npm run lint      ✓ Must pass
npm run build     ✓ Must succeed
npm run dev       ✓ Must start without errors
git status        ✓ Must be clean
```

---

**Project Status:** Ready for Implementation  
**Last Updated:** December 25, 2025  
**Next Step:** Begin PHASE 1 - Setup & Planning
