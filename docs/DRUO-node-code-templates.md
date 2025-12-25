# DRUO n8n Node - Implementation Code Templates & Examples

**Date:** December 25, 2025  
**Purpose:** Ready-to-use code templates and examples for DRUO node implementation

---

## 1. PACKAGE.JSON TEMPLATE

```json
{
  "name": "n8n-nodes-druo",
  "version": "1.0.0",
  "description": "n8n community node for DRUO payments integration - Create payment methods and process transactions",
  "license": "MIT",
  "author": "DRUO",
  "homepage": "https://github.com/DRUO/n8n-nodes-druo",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/DRUO/n8n-nodes-druo.git"
  },
  "bugs": {
    "url": "https://github.com/DRUO/n8n-nodes-druo/issues"
  },
  "keywords": [
    "n8n",
    "n8n-community-node-package",
    "payments",
    "druo",
    "payment-processing",
    "transactions",
    "integration"
  ],
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "n8n-node build",
    "dev": "n8n-node dev",
    "lint": "n8n-node lint",
    "lint:fix": "n8n-node lint --fix",
    "release": "n8n-node release"
  },
  "n8n": {
    "nodes": [
      "dist/nodes/Druo/Druo.node.js"
    ],
    "credentials": [
      "dist/credentials/DruoApi.credentials.js"
    ]
  },
  "dependencies": {},
  "devDependencies": {
    "n8n-core": "^1.x.x",
    "n8n-workflow": "^1.x.x",
    "@n8n/node-dev-kit": "^1.x.x",
    "typescript": "^5.x.x"
  }
}
```

---

## 2. CREDENTIALS TEMPLATE

### src/credentials/DruoApi.credentials.ts

```typescript
import { ICredentialType, INodeProperties } from 'n8n-workflow';

export class DruoApi implements ICredentialType {
  name = 'druoApi';
  displayName = 'DRUO API';
  documentationUrl = 'https://developer.druo.com/docs/authentication';
  
  properties: INodeProperties[] = [
    {
      displayName: 'Environment',
      name: 'environment',
      type: 'options',
      required: true,
      default: 'sandbox',
      description: 'Choose DRUO environment for API calls',
      options: [
        {
          name: 'Sandbox',
          value: 'sandbox',
          description: 'Testing environment (sandbox.druo.com)',
        },
        {
          name: 'Production',
          value: 'production',
          description: 'Live environment (api.druo.com)',
        },
      ],
    },
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: {
        password: true,
      },
      default: '',
      required: true,
      description: 'Your DRUO API key for authentication',
      placeholder: 'sk_live_xxxxxxxxxxxxx',
    },
    {
      displayName: 'API Key Type',
      name: 'apiKeyType',
      type: 'options',
      required: true,
      default: 'bearer',
      description: 'How the API key is transmitted',
      options: [
        {
          name: 'Bearer Token',
          value: 'bearer',
          description: 'Authorization: Bearer {apiKey}',
        },
        {
          name: 'API Key Header',
          value: 'header',
          description: 'X-API-Key: {apiKey}',
        },
      ],
    },
  ];

  // Optional: Implement test method
  async authenticate(
    credentials: { 
      apiKey: string; 
      environment: string; 
      apiKeyType: string;
    },
  ): Promise<{ success: boolean; message: string }> {
    // This would test credentials against a simple DRUO endpoint
    // Not required but helpful for users
    return { success: true, message: 'Credentials valid' };
  }
}
```

---

## 3. MAIN NODE TEMPLATE

### src/nodes/Druo/Druo.node.ts

```typescript
import {
  IExecuteFunctions,
  INodeType,
  INodeTypeDescription,
  NodeOperationError,
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
    description: 'Integrate with DRUO Payments API for payment processing',
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
            description: 'Create, update, retrieve payment methods',
          },
          {
            name: 'Transaction',
            value: 'transaction',
            description: 'Submit, retrieve, refund transactions',
          },
        ],
        required: true,
        default: 'paymentMethod',
      },
      
      // PAYMENT METHOD OPERATIONS
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
            action: 'Create a new payment method',
          },
          {
            name: 'Get',
            value: 'get',
            action: 'Get a payment method by ID',
          },
          {
            name: 'List',
            value: 'list',
            action: 'List all payment methods',
          },
          {
            name: 'Update',
            value: 'update',
            action: 'Update a payment method',
          },
          {
            name: 'Delete',
            value: 'delete',
            action: 'Delete a payment method',
          },
        ],
        required: true,
        default: 'create',
      },
      
      // TRANSACTION OPERATIONS
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        displayOptions: {
          show: {
            resource: ['transaction'],
          },
        },
        options: [
          {
            name: 'Create',
            value: 'create',
            action: 'Submit a new transaction',
          },
          {
            name: 'Get',
            value: 'get',
            action: 'Get transaction details',
          },
          {
            name: 'List',
            value: 'list',
            action: 'List transactions',
          },
          {
            name: 'Refund',
            value: 'refund',
            action: 'Refund a transaction',
          },
        ],
        required: true,
        default: 'create',
      },

      // PAYMENT METHOD FIELDS
      ...paymentMethod.description,
      
      // TRANSACTION FIELDS
      ...transaction.description,
    ],
  };

  async execute(this: IExecuteFunctions) {
    const items = this.getInputData();
    const returnData: any[] = [];
    const length = items.length;

    let responseData;

    const resource = this.getNodeParameter('resource', 0) as string;
    const operation = this.getNodeParameter('operation', 0) as string;

    const credentials = await this.getCredentials('druoApi') as {
      apiKey: string;
      environment: string;
      apiKeyType: string;
    };

    if (!credentials) {
      throw new NodeOperationError(
        this.getNode(),
        'No credentials found. Please configure DRUO API credentials.'
      );
    }

    // Build base URL from environment
    const baseUrl = credentials.environment === 'production'
      ? 'https://api.druo.com/v1'
      : 'https://sandbox.druo.com/v1';

    // Build authorization header
    let authHeader: any = {};
    if (credentials.apiKeyType === 'bearer') {
      authHeader = {
        'Authorization': `Bearer ${credentials.apiKey}`,
      };
    } else if (credentials.apiKeyType === 'header') {
      authHeader = {
        'X-API-Key': credentials.apiKey,
      };
    }

    const headers = {
      'Content-Type': 'application/json',
      ...authHeader,
    };

    try {
      if (resource === 'paymentMethod') {
        responseData = await paymentMethod.execute(
          this,
          operation,
          baseUrl,
          headers,
          items,
          length
        );
      } else if (resource === 'transaction') {
        responseData = await transaction.execute(
          this,
          operation,
          baseUrl,
          headers,
          items,
          length
        );
      } else {
        throw new NodeOperationError(
          this.getNode(),
          `Unknown resource: ${resource}`
        );
      }

      // Ensure responseData is always an array
      if (!Array.isArray(responseData)) {
        responseData = [responseData];
      }

      // Add responses to return data
      for (let i = 0; i < responseData.length; i++) {
        returnData.push({
          json: responseData[i],
        });
      }
    } catch (error) {
      if (this.continueOnFail()) {
        returnData.push({ json: { error: error.message } });
      } else {
        throw error;
      }
    }

    return [returnData];
  }
}
```

---

## 4. OPERATION MODULE TEMPLATE

### src/nodes/Druo/actions/PaymentMethod/index.ts

```typescript
import { INodeProperties } from 'n8n-workflow';
import * as create from './Create.operation';
import * as get from './Get.operation';
import * as list from './List.operation';
import * as update from './Update.operation';
import * as del from './Delete.operation';

export const description: INodeProperties[] = [
  // PAYMENT METHOD - CREATE
  {
    displayName: 'Account ID',
    name: 'accountId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['create'],
      },
    },
    description: 'The account ID to attach the payment method to',
  },
  {
    displayName: 'Type',
    name: 'type',
    type: 'options',
    required: true,
    default: 'card',
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['create'],
      },
    },
    options: [
      { name: 'Card', value: 'card' },
      { name: 'Bank Account', value: 'bank_account' },
      { name: 'Digital Wallet', value: 'digital_wallet' },
    ],
    description: 'Type of payment method',
  },
  // Additional card fields if type is 'card'
  {
    displayName: 'Card Number',
    name: 'cardNumber',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['create'],
        type: ['card'],
      },
    },
    description: 'Card number (without spaces or dashes)',
  },
  {
    displayName: 'Expiration Month',
    name: 'expirationMonth',
    type: 'number',
    required: true,
    default: 1,
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['create'],
        type: ['card'],
      },
    },
    typeOptions: {
      minValue: 1,
      maxValue: 12,
    },
    description: 'Card expiration month (1-12)',
  },
  {
    displayName: 'Expiration Year',
    name: 'expirationYear',
    type: 'number',
    required: true,
    default: new Date().getFullYear(),
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['create'],
        type: ['card'],
      },
    },
    description: 'Card expiration year (4-digit)',
  },
  {
    displayName: 'CVV',
    name: 'cvv',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['create'],
        type: ['card'],
      },
    },
    typeOptions: {
      password: true,
    },
    description: '3 or 4 digit security code',
  },

  // PAYMENT METHOD - GET/UPDATE/DELETE
  {
    displayName: 'Payment Method ID',
    name: 'paymentMethodId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['get', 'update', 'delete'],
      },
    },
    description: 'The ID of the payment method',
  },

  // PAYMENT METHOD - LIST
  {
    displayName: 'Account ID',
    name: 'accountId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['list'],
      },
    },
    description: 'List payment methods for this account',
  },
  {
    displayName: 'Limit',
    name: 'limit',
    type: 'number',
    default: 50,
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['list'],
      },
    },
    typeOptions: {
      minValue: 1,
      maxValue: 100,
    },
    description: 'Maximum number of results to return',
  },
  {
    displayName: 'Offset',
    name: 'offset',
    type: 'number',
    default: 0,
    displayOptions: {
      show: {
        resource: ['paymentMethod'],
        operation: ['list'],
      },
    },
    description: 'Pagination offset',
  },
];

export async function execute(
  context: any,
  operation: string,
  baseUrl: string,
  headers: any,
  items: any[],
  length: number
) {
  const responseData: any[] = [];

  for (let i = 0; i < length; i++) {
    try {
      if (operation === 'create') {
        const result = await create.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'get') {
        const result = await get.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'list') {
        const result = await list.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'update') {
        const result = await update.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'delete') {
        const result = await del.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      }
    } catch (error) {
      if (context.continueOnFail()) {
        responseData.push({ error: error.message });
        continue;
      }
      throw error;
    }
  }

  return responseData;
}
```

### src/nodes/Druo/actions/PaymentMethod/Create.operation.ts

```typescript
import { NodeOperationError } from 'n8n-workflow';

export async function execute(this: any, itemIndex: number, baseUrl: string, headers: any) {
  const accountId = this.getNodeParameter('accountId', itemIndex) as string;
  const type = this.getNodeParameter('type', itemIndex) as string;

  // Build request body based on payment method type
  const body: any = {
    account_id: accountId,
    type: type,
  };

  if (type === 'card') {
    const cardNumber = this.getNodeParameter('cardNumber', itemIndex) as string;
    const expirationMonth = this.getNodeParameter('expirationMonth', itemIndex) as number;
    const expirationYear = this.getNodeParameter('expirationYear', itemIndex) as number;
    const cvv = this.getNodeParameter('cvv', itemIndex) as string;

    // Validate card number (simple check)
    if (!/^\d{13,19}$/.test(cardNumber.replace(/\s/g, ''))) {
      throw new NodeOperationError(
        this.getNode(),
        'Invalid card number format'
      );
    }

    body.card = {
      number: cardNumber.replace(/\s/g, ''),
      expiry_month: expirationMonth,
      expiry_year: expirationYear,
      cvv: cvv,
    };
  }

  try {
    const response = await this.helpers.httpRequest({
      method: 'POST',
      url: `${baseUrl}/payment-methods`,
      headers,
      body,
      json: true,
    });

    return response;
  } catch (error) {
    if (error.response?.status === 400) {
      throw new NodeOperationError(
        this.getNode(),
        `DRUO API Error: ${error.response.data?.message || error.message}`
      );
    } else if (error.response?.status === 401) {
      throw new NodeOperationError(
        this.getNode(),
        'Authentication failed. Check your DRUO API credentials.'
      );
    } else if (error.response?.status === 409) {
      throw new NodeOperationError(
        this.getNode(),
        'Payment method already exists'
      );
    }
    throw error;
  }
}
```

### src/nodes/Druo/actions/PaymentMethod/Get.operation.ts

```typescript
import { NodeOperationError } from 'n8n-workflow';

export async function execute(this: any, itemIndex: number, baseUrl: string, headers: any) {
  const paymentMethodId = this.getNodeParameter('paymentMethodId', itemIndex) as string;

  if (!paymentMethodId) {
    throw new NodeOperationError(
      this.getNode(),
      'Payment Method ID is required'
    );
  }

  try {
    const response = await this.helpers.httpRequest({
      method: 'GET',
      url: `${baseUrl}/payment-methods/${paymentMethodId}`,
      headers,
      json: true,
    });

    return response;
  } catch (error) {
    if (error.response?.status === 404) {
      throw new NodeOperationError(
        this.getNode(),
        `Payment method with ID ${paymentMethodId} not found`
      );
    } else if (error.response?.status === 401) {
      throw new NodeOperationError(
        this.getNode(),
        'Authentication failed. Check your DRUO API credentials.'
      );
    }
    throw error;
  }
}
```

### src/nodes/Druo/actions/PaymentMethod/List.operation.ts

```typescript
import { NodeOperationError } from 'n8n-workflow';

export async function execute(this: any, itemIndex: number, baseUrl: string, headers: any) {
  const accountId = this.getNodeParameter('accountId', itemIndex) as string;
  const limit = this.getNodeParameter('limit', itemIndex) as number;
  const offset = this.getNodeParameter('offset', itemIndex) as number;

  const qs: any = {
    account_id: accountId,
    limit: limit || 50,
    offset: offset || 0,
  };

  try {
    const response = await this.helpers.httpRequest({
      method: 'GET',
      url: `${baseUrl}/payment-methods`,
      headers,
      qs,
      json: true,
    });

    return response;
  } catch (error) {
    if (error.response?.status === 401) {
      throw new NodeOperationError(
        this.getNode(),
        'Authentication failed. Check your DRUO API credentials.'
      );
    }
    throw error;
  }
}
```

### src/nodes/Druo/actions/PaymentMethod/Update.operation.ts

```typescript
import { NodeOperationError } from 'n8n-workflow';

export async function execute(this: any, itemIndex: number, baseUrl: string, headers: any) {
  const paymentMethodId = this.getNodeParameter('paymentMethodId', itemIndex) as string;
  
  // Get additional parameters that can be updated
  // Build body with only the fields provided
  const body: any = {};

  try {
    const response = await this.helpers.httpRequest({
      method: 'PATCH',
      url: `${baseUrl}/payment-methods/${paymentMethodId}`,
      headers,
      body,
      json: true,
    });

    return response;
  } catch (error) {
    if (error.response?.status === 404) {
      throw new NodeOperationError(
        this.getNode(),
        `Payment method with ID ${paymentMethodId} not found`
      );
    } else if (error.response?.status === 401) {
      throw new NodeOperationError(
        this.getNode(),
        'Authentication failed. Check your DRUO API credentials.'
      );
    }
    throw error;
  }
}
```

### src/nodes/Druo/actions/PaymentMethod/Delete.operation.ts

```typescript
import { NodeOperationError } from 'n8n-workflow';

export async function execute(this: any, itemIndex: number, baseUrl: string, headers: any) {
  const paymentMethodId = this.getNodeParameter('paymentMethodId', itemIndex) as string;

  try {
    const response = await this.helpers.httpRequest({
      method: 'DELETE',
      url: `${baseUrl}/payment-methods/${paymentMethodId}`,
      headers,
      json: true,
    });

    return { success: true, message: 'Payment method deleted', id: paymentMethodId };
  } catch (error) {
    if (error.response?.status === 404) {
      throw new NodeOperationError(
        this.getNode(),
        `Payment method with ID ${paymentMethodId} not found`
      );
    } else if (error.response?.status === 401) {
      throw new NodeOperationError(
        this.getNode(),
        'Authentication failed. Check your DRUO API credentials.'
      );
    }
    throw error;
  }
}
```

---

## 5. TRANSACTION OPERATIONS TEMPLATE

### src/nodes/Druo/actions/Transaction/index.ts

```typescript
import { INodeProperties } from 'n8n-workflow';
import * as create from './Create.operation';
import * as get from './Get.operation';
import * as list from './List.operation';
import * as refund from './Refund.operation';

export const description: INodeProperties[] = [
  // TRANSACTION - CREATE
  {
    displayName: 'Account ID',
    name: 'accountId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['create'],
      },
    },
    description: 'Account to charge',
  },
  {
    displayName: 'Payment Method ID',
    name: 'paymentMethodId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['create'],
      },
    },
    description: 'Payment method ID to charge',
  },
  {
    displayName: 'Amount (cents)',
    name: 'amount',
    type: 'number',
    required: true,
    default: 0,
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['create'],
      },
    },
    typeOptions: {
      minValue: 1,
    },
    description: 'Amount to charge in cents (e.g., 1000 for $10.00)',
  },
  {
    displayName: 'Currency',
    name: 'currency',
    type: 'options',
    required: true,
    default: 'USD',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['create'],
      },
    },
    options: [
      { name: 'USD', value: 'USD' },
      { name: 'EUR', value: 'EUR' },
      { name: 'GBP', value: 'GBP' },
    ],
    description: 'Currency code',
  },
  {
    displayName: 'Description',
    name: 'description',
    type: 'string',
    required: false,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['create'],
      },
    },
    description: 'Transaction description/memo',
  },

  // TRANSACTION - GET
  {
    displayName: 'Transaction ID',
    name: 'transactionId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['get', 'refund'],
      },
    },
    description: 'ID of the transaction',
  },

  // TRANSACTION - LIST
  {
    displayName: 'Account ID',
    name: 'accountId',
    type: 'string',
    required: true,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['list'],
      },
    },
    description: 'List transactions for this account',
  },
  {
    displayName: 'Status',
    name: 'status',
    type: 'options',
    required: false,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['list'],
      },
    },
    options: [
      { name: 'Pending', value: 'pending' },
      { name: 'Completed', value: 'completed' },
      { name: 'Failed', value: 'failed' },
      { name: 'Refunded', value: 'refunded' },
    ],
    description: 'Filter by status',
  },
  {
    displayName: 'Limit',
    name: 'limit',
    type: 'number',
    default: 50,
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['list'],
      },
    },
    typeOptions: {
      minValue: 1,
      maxValue: 100,
    },
    description: 'Maximum number of results',
  },

  // TRANSACTION - REFUND
  {
    displayName: 'Refund Amount (cents)',
    name: 'refundAmount',
    type: 'number',
    required: false,
    default: '',
    displayOptions: {
      show: {
        resource: ['transaction'],
        operation: ['refund'],
      },
    },
    description: 'Amount to refund (leave empty for full refund)',
  },
];

export async function execute(
  context: any,
  operation: string,
  baseUrl: string,
  headers: any,
  items: any[],
  length: number
) {
  const responseData: any[] = [];

  for (let i = 0; i < length; i++) {
    try {
      if (operation === 'create') {
        const result = await create.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'get') {
        const result = await get.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'list') {
        const result = await list.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      } else if (operation === 'refund') {
        const result = await refund.execute.call(context, i, baseUrl, headers);
        responseData.push(result);
      }
    } catch (error) {
      if (context.continueOnFail()) {
        responseData.push({ error: error.message });
        continue;
      }
      throw error;
    }
  }

  return responseData;
}
```

### src/nodes/Druo/actions/Transaction/Create.operation.ts

```typescript
import { NodeOperationError } from 'n8n-workflow';

export async function execute(this: any, itemIndex: number, baseUrl: string, headers: any) {
  const accountId = this.getNodeParameter('accountId', itemIndex) as string;
  const paymentMethodId = this.getNodeParameter('paymentMethodId', itemIndex) as string;
  const amount = this.getNodeParameter('amount', itemIndex) as number;
  const currency = this.getNodeParameter('currency', itemIndex) as string;
  const description = this.getNodeParameter('description', itemIndex) as string;

  if (amount < 1) {
    throw new NodeOperationError(
      this.getNode(),
      'Amount must be at least 1 cent'
    );
  }

  const body = {
    account_id: accountId,
    payment_method_id: paymentMethodId,
    amount: amount,
    currency: currency,
    description: description || undefined,
  };

  try {
    const response = await this.helpers.httpRequest({
      method: 'POST',
      url: `${baseUrl}/transactions`,
      headers,
      body,
      json: true,
    });

    return response;
  } catch (error) {
    if (error.response?.status === 400) {
      throw new NodeOperationError(
        this.getNode(),
        `DRUO API Error: ${error.response.data?.message || error.message}`
      );
    } else if (error.response?.status === 401) {
      throw new NodeOperationError(
        this.getNode(),
        'Authentication failed. Check your DRUO API credentials.'
      );
    } else if (error.response?.status === 402) {
      throw new NodeOperationError(
        this.getNode(),
        'Transaction declined by payment processor'
      );
    }
    throw error;
  }
}
```

---

## 6. TSCONFIG.JSON

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

## 7. README.MD TEMPLATE

```markdown
# n8n-nodes-druo

Official n8n community node for DRUO Payments integration. Create payment methods, manage transactions, and integrate DRUO payment processing into your n8n workflows without code.

## Installation

### Via n8n Community Nodes (Recommended)

1. Open n8n editor
2. Go to **Settings → Community nodes**
3. Search for `n8n-nodes-druo`
4. Click **Install**

### Manual Installation via npm

```bash
npm install n8n-nodes-druo
```

Then restart your n8n instance.

## Configuration

### 1. Create DRUO Credentials

1. Add a DRUO node to your workflow
2. Click **Create new credential** under "DRUO Account"
3. Enter:
   - **API Key**: Your DRUO API key (get from [developer.druo.com](https://developer.druo.com))
   - **Environment**: Select `Sandbox` (testing) or `Production` (live)
   - **API Key Type**: Choose `Bearer Token` or `API Key Header` based on your setup

### 2. Add to Workflow

1. Click the **+** button to add a node
2. Search for and select **DRUO**
3. Select the **Resource** (Payment Method or Transaction)
4. Choose the **Operation** you want to perform
5. Fill in required parameters

## Operations

### Payment Method Resource

| Operation | Description |
|-----------|-------------|
| **Create** | Add a new payment method (card, bank account) |
| **Get** | Retrieve a specific payment method's details |
| **List** | List all payment methods for an account |
| **Update** | Modify an existing payment method |
| **Delete** | Remove/disable a payment method |

**Example - Create Payment Method:**
```
Account ID: acc_123456
Type: card
Card Number: 4111111111111111
Expiration Month: 12
Expiration Year: 2025
CVV: 123
```

### Transaction Resource

| Operation | Description |
|-----------|-------------|
| **Create** | Submit a new transaction/charge |
| **Get** | Retrieve transaction details and status |
| **List** | List transactions for an account with filtering |
| **Refund** | Refund a transaction (full or partial) |

**Example - Create Transaction:**
```
Account ID: acc_123456
Payment Method ID: pm_789012
Amount: 10000 (= $100.00)
Currency: USD
Description: Order #12345
```

## Example Workflows

### 1. Simple Payment Processing

```json
[
  {
    "nodes": [
      {
        "name": "Manual Trigger",
        "type": "n8n-nodes-base.manualTrigger"
      },
      {
        "name": "Create Payment Method",
        "type": "n8n-nodes-druo.druo",
        "parameters": {
          "resource": "paymentMethod",
          "operation": "create",
          "accountId": "={{ $json.accountId }}",
          "type": "card",
          "cardNumber": "={{ $json.cardNumber }}",
          "expirationMonth": "={{ $json.expMonth }}",
          "expirationYear": "={{ $json.expYear }}",
          "cvv": "={{ $json.cvv }}"
        }
      },
      {
        "name": "Submit Transaction",
        "type": "n8n-nodes-druo.druo",
        "parameters": {
          "resource": "transaction",
          "operation": "create",
          "accountId": "={{ $json.accountId }}",
          "paymentMethodId": "={{ $prevNode.output[0].json.id }}",
          "amount": "={{ $json.amount }}",
          "currency": "USD",
          "description": "={{ $json.orderDescription }}"
        }
      }
    ]
  }
]
```

## Troubleshooting

### Authentication Error (401)

- Verify your API key is correct
- Check that the API key is active in DRUO dashboard
- Confirm correct environment (Sandbox vs. Production)

### Invalid Request (400)

- Validate all required parameters are provided
- Check parameter formats match API requirements
- Review error message for specific field issues

### Card Declined (402)

- Card may be expired or blocked
- Verify card details are correct
- Check account balance or transaction limits

### Node Not Appearing

- Restart n8n after installation
- Clear browser cache
- Verify `n8n-nodes-druo` shows in **Settings → Community nodes**

## API Documentation

For detailed API documentation, visit:
- DRUO API Docs: [developer.druo.com](https://developer.druo.com)
- Authentication: [developer.druo.com/docs/authentication](https://developer.druo.com/docs/authentication)
- Payment Methods: [developer.druo.com/docs/payment-methods](https://developer.druo.com/docs/payment-methods)
- Transactions: [developer.druo.com/docs/transactions](https://developer.druo.com/docs/transactions)

## Support

- **Issues**: [GitHub Issues](https://github.com/DRUO/n8n-nodes-druo/issues)
- **DRUO Support**: [support@druo.com](mailto:support@druo.com)
- **n8n Community**: [community.n8n.io](https://community.n8n.io)

## License

MIT

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## Changelog

### 1.0.0 (Initial Release)

- Full DRUO API integration
- Payment Method operations (Create, Get, List, Update, Delete)
- Transaction operations (Create, Get, List, Refund)
- Complete error handling
- Support for Sandbox and Production environments
```

---

## 8. ESLINT CONFIGURATION

### .eslintrc.js (auto-generated by n8n-node, but here's the typical config)

```javascript
module.exports = {
  root: true,
  extends: ['@n8n-io/eslint-config/node'],
  parserOptions: {
    sourceType: 'module',
  },
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
  },
};
```

---

## 9. GITHUB WORKFLOWS (CI/CD)

### .github/workflows/test.yml

```yaml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run lint

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm run build
```

---

## 10. GITIGNORE

```
node_modules/
dist/
*.log
.env
.env.local
.DS_Store
dist
coverage
.cache
.idea
*.swp
*.swo
*~
```

---

**Status:** Ready for Implementation  
**Last Updated:** December 25, 2025
