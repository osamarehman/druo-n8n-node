# DRUO n8n Community Node - Complete Implementation Guide

**Date:** December 25, 2025  
**Target:** n8n Community Node Package (`n8n-nodes-druo`)  
**Status:** Complete Best Practices & Setup Guide

---

## 1. OVERVIEW

This guide consolidates:
- n8n's official community node standards and verification guidelines
- Code standards and best practices
- Node file structure recommendations
- Complete technical requirements for DRUO node implementation
- Step-by-step implementation workflow
- Verification checklist for npm publication

---

## 2. COMMUNITY NODE STANDARDS & REQUIREMENTS

### 2.1 Package Naming & Metadata

**Package Name Format:**
```
n8n-nodes-druo
OR
@druo/n8n-nodes-druo (if using organization scope)
```

**package.json Requirements:**
```json
{
  "name": "n8n-nodes-druo",
  "version": "1.0.0",
  "description": "n8n community node for DRUO payments integration",
  "license": "MIT",
  "author": "DRUO",
  "keywords": [
    "n8n",
    "n8n-community-node-package",
    "payments",
    "druo",
    "integration"
  ],
  "n8n": {
    "nodes": [
      "dist/nodes/Druo/Druo.node.js"
    ],
    "credentials": [
      "dist/credentials/DruoApi.credentials.js"
    ]
  },
  "scripts": {
    "build": "n8n-node build",
    "dev": "n8n-node dev",
    "lint": "n8n-node lint",
    "lint:fix": "n8n-node lint --fix",
    "release": "n8n-node release"
  }
}
```

### 2.2 Mandatory Requirements

✅ **Package name:** Must start with `n8n-nodes-` or `@scope/n8n-nodes-`

✅ **Keywords:** Must include `n8n-community-node-package`

✅ **metadata:** Nodes and credentials registered in `package.json` under `n8n` attribute

✅ **License:** MIT (required for verification)

✅ **Linting:** Must pass `npm run lint` without errors

✅ **Testing:** Must work locally with `npm run dev`

✅ **Repository:** Should have public GitHub repo linked in npm

### 2.3 Verification-Ready Requirements (if submitting for approval)

✅ **No runtime external dependencies** - Keep package lightweight, use only n8n built-ins

✅ **No environment variable access** - All data through node parameters

✅ **No file system access** - No `fs`, `path`, or file I/O operations

✅ **English only** - All UI text, parameters, help text in English

✅ **Clear documentation** - README with installation, configuration, usage examples

✅ **Example workflows** - JSON exports showing real use cases

---

## 3. NODE FILE STRUCTURE (Best Practice)

### 3.1 Directory Layout

```
n8n-nodes-druo/
├── src/
│   ├── nodes/
│   │   └── Druo/
│   │       ├── Druo.node.ts          # Main node class
│   │       ├── Druo.node.json        # Node metadata/codex
│   │       └── actions/
│   │           ├── PaymentMethod/
│   │           │   ├── Create.operation.ts
│   │           │   ├── Get.operation.ts
│   │           │   ├── List.operation.ts
│   │           │   ├── Update.operation.ts
│   │           │   └── Delete.operation.ts
│   │           ├── Transaction/
│   │           │   ├── Create.operation.ts
│   │           │   ├── Get.operation.ts
│   │           │   ├── List.operation.ts
│   │           │   └── Refund.operation.ts
│   │           └── [other resources as per API]
│   ├── credentials/
│   │   └── DruoApi.credentials.ts    # API credentials handler
│   └── index.ts                       # Exports
├── dist/                              # Compiled output (auto-generated)
├── package.json
├── tsconfig.json
├── .eslintrc.js
├── README.md
├── LICENSE (MIT)
├── .gitignore
└── CHANGELOG.md
```

### 3.2 Key File Descriptions

**Druo.node.ts** - Main node class
- Implements `INodeType` interface
- Defines resources and operations via `getNodeProperties()`
- Implements `execute()` method with operation routing

**Druo.node.json** - Node metadata
- Display name, description, icon, documentation links

**Credentials file (DruoApi.credentials.ts)** - API authentication
- Handles API key/Bearer token authentication
- Implements `ICredentialType` interface
- Validates credentials before execution

**Operation files** - Modular operations
- Each file exports `description` object and `execute` async function
- Handles single operation logic
- Keeps code clean and maintainable

---

## 4. CODE STANDARDS & BEST PRACTICES

### 4.1 TypeScript & Modern JavaScript

✅ Use **TypeScript** exclusively
✅ ES6+ syntax: `const`/`let`, arrow functions, template literals, destructuring
✅ Async/await for promises (not `.then()` chains)
✅ Clear, descriptive variable and function names

### 4.2 Resources & Operations Pattern

```typescript
// Node must expose operations through Resource > Operation parameters
const nodeProperties = {
  displayName: 'DRUO',
  properties: [
    {
      displayName: 'Resource',
      name: 'resource',
      type: 'options',
      options: [
        {
          name: 'Payment Method',
          value: 'paymentMethod',
        },
        {
          name: 'Transaction',
          value: 'transaction',
        },
        // ... other resources
      ],
      required: true,
      default: 'paymentMethod',
    },
    {
      displayName: 'Operation',
      name: 'operation',
      type: 'options',
      displayOptions: {
        show: {
          resource: ['paymentMethod'],
        },
      },
      options: [
        {
          name: 'Create',
          value: 'create',
          action: 'Create a payment method',
        },
        {
          name: 'Get',
          value: 'get',
          action: 'Get a payment method',
        },
        // ... other operations for this resource
      ],
      required: true,
      default: 'create',
    },
    // Operation-specific fields follow with displayOptions
  ]
}
```

### 4.3 Operation Execution Flow

Each operation should:
1. ✅ Validate input parameters
2. ✅ Build request (method, URL, headers, body)
3. ✅ Call API using `this.helpers.httpRequest()`
4. ✅ Handle errors gracefully
5. ✅ Return data in expected format

**Example operation structure:**
```typescript
// src/nodes/Druo/actions/PaymentMethod/Create.operation.ts

import { INodeProperties } from 'n8n-workflow';

export const description: INodeProperties[] = [
  {
    displayName: 'Account ID',
    name: 'accountId',
    type: 'string',
    required: true,
    default: '',
    description: 'The DRUO account ID',
  },
  {
    displayName: 'Method Type',
    name: 'methodType',
    type: 'options',
    options: [
      { name: 'Credit Card', value: 'credit_card' },
      { name: 'Bank Account', value: 'bank_account' },
    ],
    required: true,
    default: 'credit_card',
  },
  // ... other fields specific to create operation
];

export async function execute(
  this: IExecuteFunctions,
  itemIndex: number,
  nodeProperties: any,
) {
  const accountId = this.getNodeParameter('accountId', itemIndex);
  const methodType = this.getNodeParameter('methodType', itemIndex);
  
  const body = {
    account_id: accountId,
    type: methodType,
    // ... build full request body from parameters
  };

  const response = await this.helpers.httpRequest({
    method: 'POST',
    url: `${baseUrl}/payment-methods`,
    headers: {
      'Authorization': `Bearer ${credentials.apiKey}`,
      'Content-Type': 'application/json',
    },
    body,
  });

  return [{ json: response }];
}
```

### 4.4 HTTP Request Helper (No External Dependencies)

✅ Use **only** the built-in n8n HTTP helper:
```typescript
const response = await this.helpers.httpRequest({
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE',
  url: 'https://api.druo.com/endpoint',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json',
  },
  body: {...}, // For POST/PUT/PATCH
  qs: {...},   // Query string params
  json: true,  // Auto-parse JSON response
});
```

### 4.5 Input Data Safety

⚠️ **Critical:** Never modify incoming data directly
```typescript
// ❌ WRONG - modifies shared data
const items = this.getInputData();
items[0].json.modified = 'value';
return items;

// ✅ CORRECT - clones data
const items = this.getInputData();
const newItems = [];
for (let i = 0; i < items.length; i++) {
  newItems.push({
    json: { ...items[i].json, modified: 'value' },
  });
}
return newItems;
```

### 4.6 Credentials Pattern

```typescript
// src/credentials/DruoApi.credentials.ts

import { ICredentialType, INodeProperties } from 'n8n-workflow';

export class DruoApi implements ICredentialType {
  name = 'druoApi';
  displayName = 'DRUO API';
  
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: {
        password: true,
      },
      required: true,
      default: '',
    },
    {
      displayName: 'Environment',
      name: 'environment',
      type: 'options',
      options: [
        { name: 'Sandbox', value: 'sandbox' },
        { name: 'Production', value: 'production' },
      ],
      default: 'sandbox',
    },
  ];
}
```

### 4.7 Error Handling

✅ Always handle errors gracefully:
```typescript
try {
  const response = await this.helpers.httpRequest(options);
  return [{ json: response }];
} catch (error) {
  if (error.response?.status === 401) {
    throw new NodeOperationError(
      this.getNode(),
      'Invalid DRUO API credentials'
    );
  }
  if (error.response?.status === 400) {
    throw new NodeOperationError(
      this.getNode(),
      `Invalid request: ${error.response.data?.message || error.message}`
    );
  }
  throw error; // Re-throw if not handled
}
```

---

## 5. n8n-NODE TOOL WORKFLOW

### 5.1 Initial Setup

**Step 1: Create project with n8n-node**
```bash
npm create @n8n/node@latest n8n-nodes-druo

# Or install globally and use:
npm install -g n8n-node
n8n-node new n8n-nodes-druo
```

**Step 2: Answer interactive prompts**
- Node name: `n8n-nodes-druo`
- Node type: **Other** (full programmatic control)
- Authentication: **Bearer Token** (or API Key, depending on DRUO API)

**Step 3: Navigate to project**
```bash
cd n8n-nodes-druo
npm install
```

### 5.2 Development Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Run local n8n with node loaded for testing (localhost:5678) |
| `npm run build` | Compile TypeScript to JavaScript in `dist/` |
| `npm run lint` | Check code against n8n standards (must pass before release) |
| `npm run lint:fix` | Auto-fix linting issues where possible |
| `npm run release` | Build, lint, version bump, git tag, and publish to npm |

### 5.3 Local Development Cycle

1. **Edit TypeScript files** in `src/nodes/` and `src/credentials/`
2. **Run `npm run dev`** - Automatically rebuilds and reloads n8n
3. **Test in n8n UI** at localhost:5678
4. **Debug issues** - Check n8n console and node logs
5. **Repeat** until all operations work correctly

### 5.4 Pre-Release Checklist

```bash
# 1. Ensure all dependencies are correct
npm ls --depth=0

# 2. Lint the entire project
npm run lint

# 3. Build the project
npm run build

# 4. Test locally one more time
npm run dev

# 5. Update package version
# (release command handles this, but verify manually if needed)

# 6. Ensure git is clean
git status

# 7. Release to npm (requires npm login)
npm login
npm run release
```

---

## 6. DRUO API INTEGRATION SPECIFICS

### 6.1 API Information Required

Before implementation, gather from DRUO:

**Authentication:**
- [ ] Base URL (e.g., `https://api.druo.com` or sandbox)
- [ ] Authentication method (API Key header? Bearer token? OAuth2?)
- [ ] How to obtain test credentials
- [ ] API key/token field names

**API Endpoints Mapping:**
- [ ] Complete list of all endpoints
- [ ] HTTP method for each endpoint
- [ ] Request/response schemas
- [ ] Pagination handling (if applicable)
- [ ] Rate limits and error codes

**Payment Operations:**
- [ ] Create payment method (card, bank account, etc.)
- [ ] Get payment method details
- [ ] List payment methods (with filtering/pagination)
- [ ] Update payment method
- [ ] Delete/disable payment method
- [ ] Submit transaction
- [ ] Get transaction status
- [ ] Refund transaction
- [ ] List transactions
- [ ] [Any other key operations]

### 6.2 Resource Organization

Recommended resource grouping based on typical payment API:

```
Resources:
├── PaymentMethod
│   ├── Create
│   ├── Get
│   ├── List
│   ├── Update
│   └── Delete
├── Transaction
│   ├── Create
│   ├── Get
│   ├── List
│   └── Refund
├── Account (if supported)
│   ├── Get Info
│   └── Get Balance
└── Settlement/Payout (if supported)
    ├── List
    └── Get Details
```

### 6.3 Node Configuration Pattern

```typescript
// For each resource, define operation discovery:

getNodeProperties() {
  return [
    // Resource selector
    {
      displayName: 'Resource',
      name: 'resource',
      type: 'options',
      options: [
        { name: 'Payment Method', value: 'paymentMethod' },
        { name: 'Transaction', value: 'transaction' },
      ],
      default: 'paymentMethod',
    },
    // Operation selector
    {
      displayName: 'Operation',
      name: 'operation',
      type: 'options',
      displayOptions: { show: { resource: ['paymentMethod'] } },
      options: [
        { name: 'Create', value: 'create' },
        { name: 'Get', value: 'get' },
        { name: 'List', value: 'list' },
      ],
      default: 'create',
    },
    // Operation-specific fields...
  ];
}
```

---

## 7. VERIFICATION GUIDELINES CHECKLIST

Before submitting to n8n for verification:

### 7.1 Technical Requirements

- [ ] **Package name** starts with `n8n-nodes-` or `@org/n8n-nodes-`
- [ ] **Keywords** include `n8n-community-node-package`
- [ ] **License** is MIT (in package.json and LICENSE file)
- [ ] **No runtime dependencies** - only devDependencies allowed
- [ ] **No environment variable access** - all via node parameters
- [ ] **No file system access** - no `fs`, `path` operations
- [ ] **Linting passes** - `npm run lint` returns no errors
- [ ] **Builds successfully** - `npm run build` completes
- [ ] **Works locally** - `npm run dev` loads in n8n

### 7.2 Code Quality

- [ ] **TypeScript** - all code in TypeScript, no JavaScript
- [ ] **Follows n8n code standards** - reviewed against guidelines
- [ ] **Proper error handling** - try/catch blocks, meaningful error messages
- [ ] **Input validation** - all parameters validated before use
- [ ] **No console logs** - removed all debug logging in production code
- [ ] **Proper data handling** - incoming data cloned, not modified

### 7.3 Documentation

- [ ] **README.md** includes:
  - Installation instructions
  - Credential configuration steps
  - Examples of each operation with sample inputs/outputs
  - Links to DRUO API documentation
  - Troubleshooting common issues
- [ ] **Example workflows** (JSON exports) showing:
  - Basic payment method creation
  - Transaction submission
  - Any other key flows
- [ ] **node.json** includes description and icon

### 7.4 English Language

- [ ] All parameter names in English
- [ ] All descriptions/help text in English
- [ ] All error messages in English
- [ ] README and documentation in English

### 7.5 Credentials & Security

- [ ] Sensitive fields marked with `password: true` in credentials
- [ ] API key never logged or exposed in error messages
- [ ] Credentials properly authenticated before operations
- [ ] No hard-coded test credentials in code

### 7.6 Node Metadata

- [ ] Node name and display name set correctly
- [ ] Node description accurate and helpful
- [ ] Icon/image for node included
- [ ] Documentation URL links to DRUO or n8n docs

---

## 8. PUBLISHING TO NPM

### 8.1 Pre-Publication Steps

1. **Create npm account** (if not already done)
   ```bash
   npm adduser
   ```

2. **Verify npm login**
   ```bash
   npm whoami
   ```

3. **Update package.json** with final details:
   - Correct version number (start with 1.0.0)
   - Author information
   - Repository URL
   - Homepage/documentation links

4. **Test build locally**
   ```bash
   npm run build
   npm run lint
   ```

### 8.2 Publishing with n8n-node Release Command

```bash
npm run release
```

This command automatically:
- ✅ Builds the project
- ✅ Runs lint checks
- ✅ Prompts for version bump
- ✅ Updates CHANGELOG.md
- ✅ Creates git tags
- ✅ Publishes to npm
- ✅ Creates GitHub release

### 8.3 Manual Publishing (if preferred)

```bash
# 1. Build
npm run build

# 2. Lint
npm run lint

# 3. Update version in package.json
npm version patch  # or minor, major

# 4. Publish
npm publish

# 5. Create git tag
git tag v1.0.0
git push origin main v1.0.0
```

### 8.4 Verify Publication

```bash
# Check npm
npm info n8n-nodes-druo

# Install from npm in test environment
npm install n8n-nodes-druo
```

---

## 9. OPTIONAL: SUBMIT FOR n8n VERIFICATION

After successful npm publication, can optionally submit for verification:

### 9.1 Submission Steps

1. Go to https://creators.n8n.io/nodes
2. Sign up or log in
3. Click "Submit Node"
4. Provide:
   - NPM package name: `n8n-nodes-druo`
   - GitHub repository URL
   - Brief description
   - Contact information

5. n8n team reviews for:
   - Code quality and standards compliance
   - No runtime dependencies
   - No env/filesystem access
   - Proper documentation
   - UX/design consistency

### 9.2 Review Timeline

- Initial review: 1-2 weeks
- May request changes or clarifications
- Once approved, appears in n8n nodes panel
- Users can discover and install via **Settings → Community Nodes**

### 9.3 Post-Verification Support

- Maintain node in npm
- Fix bugs and security issues
- Keep documentation updated
- Respond to user issues/PRs

---

## 10. PROJECT STRUCTURE TEMPLATE

### 10.1 src/nodes/Druo/Druo.node.ts

```typescript
import {
  IExecuteFunctions,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';

import * as paymentMethod from './actions/PaymentMethod';
import * as transaction from './actions/Transaction';

export class Druo implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'DRUO',
    name: 'druo',
    icon: 'file:druo.svg',
    group: ['transform'],
    version: 1,
    subtitle: '={{$parameter["operation"]}}',
    description: 'Interact with DRUO Payments API',
    defaults: {
      name: 'DRUO',
    },
    inputs: ['main'],
    outputs: ['main'],
    credentials: [
      {
        name: 'druoApi',
        required: true,
      },
    ],
    properties: [
      {
        displayName: 'Resource',
        name: 'resource',
        type: 'options',
        noDataExpression: true,
        options: [
          {
            name: 'Payment Method',
            value: 'paymentMethod',
          },
          {
            name: 'Transaction',
            value: 'transaction',
          },
        ],
        required: true,
        default: 'paymentMethod',
      },
      // Operation selection
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        displayOptions: {
          show: {
            resource: ['paymentMethod'],
          },
        },
        options: [
          {
            name: 'Create',
            value: 'create',
            action: 'Create a payment method',
          },
          {
            name: 'Get',
            value: 'get',
            action: 'Get a payment method',
          },
          {
            name: 'List',
            value: 'list',
            action: 'List payment methods',
          },
        ],
        required: true,
        default: 'create',
      },
      // Payment Method operations fields
      ...paymentMethod.operations,
    ],
  };

  async execute(this: IExecuteFunctions) {
    const resource = this.getNodeParameter('resource', 0) as string;
    const operation = this.getNodeParameter('operation', 0) as string;

    let responseData;

    const credentials = await this.getCredentials('druoApi');
    const baseUrl = credentials.environment === 'production'
      ? 'https://api.druo.com'
      : 'https://sandbox.druo.com';

    if (resource === 'paymentMethod') {
      responseData = await paymentMethod.execute(
        this,
        operation,
        baseUrl,
        credentials
      );
    } else if (resource === 'transaction') {
      responseData = await transaction.execute(
        this,
        operation,
        baseUrl,
        credentials
      );
    }

    return this.prepareOutputData(responseData);
  }
}
```

### 10.2 src/credentials/DruoApi.credentials.ts

```typescript
import { ICredentialType, INodeProperties } from 'n8n-workflow';

export class DruoApi implements ICredentialType {
  name = 'druoApi';
  displayName = 'DRUO API';
  documentationUrl = 'https://developer.druo.com';
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: {
        password: true,
      },
      default: '',
      required: true,
    },
    {
      displayName: 'Environment',
      name: 'environment',
      type: 'options',
      default: 'sandbox',
      options: [
        {
          name: 'Sandbox',
          value: 'sandbox',
        },
        {
          name: 'Production',
          value: 'production',
        },
      ],
    },
  ];
}
```

### 10.3 README.md Structure

```markdown
# n8n-nodes-druo

n8n community node for DRUO Payments integration.

## Installation

### Via n8n UI
Settings → Community nodes → Search "n8n-nodes-druo" → Install

### Via npm
```bash
npm install n8n-nodes-druo
```

## Configuration

1. Add DRUO node to workflow
2. Create new DRUO credentials
3. Enter your DRUO API key
4. Select environment (Sandbox/Production)
5. Configure operation parameters

## Operations

### Payment Method
- **Create** - Add new payment method
- **Get** - Retrieve payment method details
- **List** - List all payment methods
- **Update** - Modify payment method
- **Delete** - Remove payment method

### Transaction
- **Create** - Submit new transaction
- **Get** - Retrieve transaction details
- **List** - List transactions
- **Refund** - Refund a transaction

## Example Workflows

See `examples/` directory for workflow JSON exports.

## Support

For issues or questions:
- DRUO API docs: https://developer.druo.com
- n8n community: https://community.n8n.io

## License

MIT
```

---

## 11. QUICK START CHECKLIST

### Phase 1: Setup (Day 1)
- [ ] Create npm account (if needed)
- [ ] Create GitHub repository for druo-node
- [ ] Run `npm create @n8n/node@latest n8n-nodes-druo`
- [ ] Answer setup prompts
- [ ] Verify initial project structure

### Phase 2: Implementation (Days 2-5)
- [ ] Map all DRUO API endpoints to n8n operations
- [ ] Create credentials class (DruoApi)
- [ ] Create main node class (Druo.node.ts)
- [ ] Create operation modules for each resource
- [ ] Test each operation locally with `npm run dev`
- [ ] Fix any linting issues

### Phase 3: Verification & Testing (Day 6)
- [ ] Run full lint check: `npm run lint`
- [ ] Build project: `npm run build`
- [ ] Test all operations in local n8n instance
- [ ] Create example workflows (JSON exports)
- [ ] Write comprehensive README

### Phase 4: Publishing (Day 7)
- [ ] Update package.json with final info
- [ ] Run final lint and build
- [ ] Publish to npm: `npm run release`
- [ ] Verify npm package is accessible
- [ ] Create GitHub release

### Phase 5: Verification (Optional, Days 8+)
- [ ] Verify node passes `@n8n/scan-community-package`
- [ ] Submit to n8n Creator Portal
- [ ] Respond to review feedback
- [ ] Once approved, appears in n8n editor

---

## 12. RESOURCES & REFERENCES

### Official n8n Documentation
- Building Nodes: https://docs.n8n.io/integrations/creating-nodes/overview/
- n8n-node Tool: https://docs.n8n.io/integrations/creating-nodes/build/n8n-node/
- Code Standards: https://docs.n8n.io/integrations/creating-nodes/build/reference/code-standards/
- Node File Structure: https://docs.n8n.io/integrations/creating-nodes/build/reference/node-file-structure/
- Community Node Standards: https://docs.n8n.io/integrations/community-nodes/build-community-nodes/
- Verification Guidelines: https://docs.n8n.io/integrations/creating-nodes/build/reference/verification-guidelines/
- Submit for Verification: https://docs.n8n.io/integrations/creating-nodes/deploy/submit-community-nodes/

### n8n Creator Portal
- Submit Nodes: https://creators.n8n.io/nodes

### Example Nodes
- n8n GitHub Issues node (simple): https://github.com/n8n-io/n8n-nodes-starter
- Airtable node (complex): https://github.com/n8n-io/n8n/tree/master/packages/nodes-base/nodes/Airtable

### npm Documentation
- Publishing Packages: https://docs.npmjs.com/packages-and-modules/contributing-packages-to-the-registry

### DRUO Resources
- API Documentation: https://developer.druo.com
- Support: [DRUO support contact]

---

## 13. TROUBLESHOOTING GUIDE

### Issue: `npm run lint` fails with TypeScript errors

**Solution:**
```bash
npm run lint:fix  # Auto-fix where possible
# For remaining issues, review n8n code standards docs
```

### Issue: `npm run dev` won't start

**Solution:**
```bash
# Clear build cache
rm -rf dist/ node_modules/.cache

# Reinstall dependencies
npm install

# Try again
npm run dev
```

### Issue: Node doesn't appear in n8n UI

**Solution:**
- Verify `package.json` has correct `n8n.nodes` path
- Check `dist/nodes/` directory exists after build
- Restart n8n: Stop `npm run dev` and restart
- Check browser console for errors

### Issue: API calls return 401/403 errors

**Solution:**
- Verify credentials are correct
- Check API key has proper permissions in DRUO dashboard
- Verify base URL matches environment (sandbox vs. production)
- Check API authentication header format

### Issue: Verification fails with dependency errors

**Solution:**
- Run `npm ls --depth=0` to check dependencies
- Move external packages to devDependencies if possible
- Use n8n built-in helpers instead of external libraries
- Re-run `npx @n8n/scan-community-package n8n-nodes-druo`

---

## 14. BEST PRACTICES SUMMARY

✅ **Always:**
- Use `n8n-node` CLI tool for scaffolding
- Write all code in TypeScript
- Run `npm run lint` before publishing
- Clone incoming data, never modify directly
- Use n8n's built-in HTTP helper
- Include comprehensive documentation
- Test locally before publishing
- Use MIT license

❌ **Never:**
- Use external dependencies (except devDependencies)
- Access environment variables
- Read/write files
- Hard-code credentials
- Log sensitive information
- Modify incoming node data
- Use non-English text in code/UI
- Publish without linting

---

**Document Updated:** December 25, 2025  
**n8n-node Version:** Latest (as of guide date)  
**Status:** Ready for Implementation
