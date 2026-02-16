# ZuriSpend: Amplify Gen 2 Migration Guide

This guide provides step-by-step prompts to migrate ZuriSpend from mock data to AWS Amplify Gen 2 backend.

## Overview

The migration replaces the `MockExpenseService` with real AWS services:

| Feature        | Mock Implementation   | Amplify Gen 2   |
| -------------- | --------------------- | --------------- |
| Authentication | None                  | Amazon Cognito  |
| Data Storage   | In-memory array       | Amazon DynamoDB |
| File Storage   | Local URIs            | Amazon S3       |
| API            | Direct function calls | AppSync GraphQL |

## Prerequisites

- AWS Account with appropriate permissions
- Node.js 18+
- Expo CLI installed
- AWS CLI configured (optional, for debugging)

## Migration Steps

Execute these prompts in order:

### [Step 1: Initialize Amplify](./01-amplify-setup.md)

- Install Amplify dependencies
- Initialize Amplify Gen 2 project
- Configure Amplify in the app

### [Step 2: Configure Authentication](./02-auth-setup.md)

- Set up Cognito User Pool
- Create auth service and context
- Build sign in/sign up screens
- Add protected routing

### [Step 3: Define Data Model](./03-data-model.md)

- Create DynamoDB schema for Expense and Category
- Update TypeScript types
- Set up category seeding for new users

### [Step 4: Implement Expense Service](./04-expense-service.md)

- Create AmplifyExpenseService
- Implement CRUD operations
- Create service factory for easy switching

### [Step 5: Configure Storage](./05-storage-setup.md)

- Set up S3 bucket for invoices
- Create storage service
- Implement upload/download/delete

### [Step 6: Update UI Components](./06-ui-updates.md)

- Update screens for new data structure
- Handle signed URLs for images
- Add sign out functionality

### [Step 7: Error Handling](./07-error-handling.md)

- Add error boundary
- Create loading and empty states
- Improve error messages

### [Step 8: Testing & Deployment](./08-testing-deployment.md)

- Local testing checklist
- Deploy to AWS
- Production configuration

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    ZuriSpend App                        │
├─────────────────────────────────────────────────────────┤
│  UI Layer (React Native / Expo Router)                  │
│  ├── (auth) screens                                     │
│  ├── (tabs) screens                                     │
│  └── Modal screens                                      │
├─────────────────────────────────────────────────────────┤
│  Service Layer                                          │
│  ├── authService (Cognito)                              │
│  ├── expenseService (DynamoDB via AppSync)              │
│  └── storageService (S3)                                │
├─────────────────────────────────────────────────────────┤
│  AWS Amplify Gen 2                                      │
│  ├── amplify/auth/resource.ts                           │
│  ├── amplify/data/resource.ts                           │
│  └── amplify/storage/resource.ts                        │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                    AWS Cloud                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Cognito   │  │  DynamoDB   │  │     S3      │     │
│  │  User Pool  │  │   Tables    │  │   Bucket    │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

## Key Changes Summary

### Types

- `expense.category` → `expense.categoryId`
- `expense.invoiceUri` → `expense.invoiceKey`
- Added `owner`, `createdAt`, `updatedAt` fields

### Services

- `MockExpenseService` → `AmplifyExpenseService`
- New `authService` for authentication
- New `storageService` for S3 operations

### New Files

- `amplify/` - Backend resource definitions
- `src/contexts/AuthContext.tsx` - Auth state management
- `src/services/amplifyClient.ts` - Typed GraphQL client
- `src/services/amplifyExpenseService.ts` - DynamoDB operations
- `src/services/storageService.ts` - S3 operations
- `app/(auth)/` - Authentication screens

## Estimated Time

| Step                    | Time          |
| ----------------------- | ------------- |
| Step 1: Setup           | 30 min        |
| Step 2: Auth            | 1-2 hours     |
| Step 3: Data Model      | 30 min        |
| Step 4: Expense Service | 1 hour        |
| Step 5: Storage         | 1 hour        |
| Step 6: UI Updates      | 1-2 hours     |
| Step 7: Error Handling  | 30 min        |
| Step 8: Testing         | 1-2 hours     |
| **Total**               | **6-9 hours** |

## Tips

1. Run `npx ampx sandbox` in a separate terminal during development
2. Use the mock service for rapid UI development
3. Test each step before moving to the next
4. Check CloudWatch logs for debugging backend issues
5. Use the Amplify Console to inspect data and users
