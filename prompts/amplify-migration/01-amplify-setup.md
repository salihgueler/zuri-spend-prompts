# Step 1: Initialize Amplify Gen 2 Backend

Set up AWS Amplify Gen 2 in the ZuriSpend React Native Expo app.

## Prerequisites

- AWS account with appropriate permissions
- Node.js 18+ installed
- npm or yarn package manager

## Tasks

### 1.1 Install Amplify Dependencies

Install the required Amplify packages:

```bash
cd ZuriSpend
npm install aws-amplify @aws-amplify/react-native @react-native-async-storage/async-storage react-native-get-random-values
```

### 1.2 Initialize Amplify Gen 2

Run the Amplify Gen 2 initialization:

```bash
npx ampx sandbox
```

This creates the `amplify/` folder structure with:

- `amplify/auth/resource.ts` - Authentication configuration
- `amplify/data/resource.ts` - Data model configuration
- `amplify/backend.ts` - Backend definition
- `amplify/tsconfig.json` - TypeScript config for Amplify

### 1.3 Configure Amplify in the App

Create `ZuriSpend/src/amplify-config.ts`:

```typescript
import { Amplify } from "aws-amplify";
import outputs from "../amplify_outputs.json";

export function configureAmplify() {
  Amplify.configure(outputs);
}
```

### 1.4 Update App Entry Point

Update `ZuriSpend/app/_layout.tsx` to configure Amplify on app start:

```typescript
import { configureAmplify } from "../src/amplify-config";
import "react-native-get-random-values";

// Configure Amplify before rendering
configureAmplify();
```

### 1.5 Update .gitignore

Add Amplify-specific entries to `ZuriSpend/.gitignore`:

```
# Amplify Gen 2
amplify_outputs.json
.amplify/
```

## Verification

- [ ] `amplify/` folder exists with resource files
- [ ] Amplify packages installed in package.json
- [ ] App starts without errors
- [ ] `npx ampx sandbox` runs successfully

## Next Step

Proceed to `02-auth-setup.md` to configure Cognito authentication.
