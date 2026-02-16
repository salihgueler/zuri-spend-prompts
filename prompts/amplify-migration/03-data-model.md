# Step 3: Define DynamoDB Data Model

Create the Amplify Gen 2 data model for expenses and categories.

## Tasks

### 3.1 Define Data Schema

Update `amplify/data/resource.ts`:

```typescript
import { type ClientSchema, a, defineData } from "@aws-amplify/backend";

const schema = a.schema({
  Category: a
    .model({
      name: a.string().required(),
      icon: a.string().required(),
      color: a.string().required(),
    })
    .authorization((allow) => [allow.owner()]),

  Expense: a
    .model({
      amount: a.float().required(),
      categoryId: a.string().required(),
      description: a.string().required(),
      date: a.date().required(),
      currency: a.string().required().default("CHF"),
      invoiceKey: a.string(), // S3 key for invoice image
    })
    .authorization((allow) => [allow.owner()]),
});

export type Schema = ClientSchema<typeof schema>;

export const data = defineData({
  schema,
  authorizationModes: {
    defaultAuthorizationMode: "userPool",
  },
});
```

### 3.2 Update Backend Definition

Update `amplify/backend.ts`:

```typescript
import { defineBackend } from "@aws-amplify/backend";
import { auth } from "./auth/resource";
import { data } from "./data/resource";

defineBackend({
  auth,
  data,
});
```

### 3.3 Update TypeScript Types

Update `ZuriSpend/src/types/index.ts`:

```typescript
/**
 * ZuriSpend Type Definitions
 * These types mirror the Amplify Gen 2 schema
 */

export interface Category {
  id: string;
  name: string;
  icon: string;
  color: string;
  owner?: string;
  createdAt?: string;
  updatedAt?: string;
}

export interface Expense {
  id: string;
  amount: number;
  categoryId: string; // Changed from 'category' to match schema
  description: string;
  date: string; // ISO date string (YYYY-MM-DD)
  currency: string;
  invoiceKey?: string; // S3 key instead of URI
  owner?: string;
  createdAt?: string;
  updatedAt?: string;
}

// For backward compatibility during migration
export type CategoryName =
  | "food"
  | "transport"
  | "lodging"
  | "activities"
  | "shopping";

export type ExpenseListItem =
  | { type: "header"; title: string; key: string }
  | { type: "expense"; data: Expense; key: string };

// Input types for creating/updating
export type CreateExpenseInput = Omit<
  Expense,
  "id" | "owner" | "createdAt" | "updatedAt"
>;
export type UpdateExpenseInput = Partial<CreateExpenseInput> & { id: string };

export type CreateCategoryInput = Omit<
  Category,
  "id" | "owner" | "createdAt" | "updatedAt"
>;
```

### 3.4 Generate Amplify Client Types

After running `npx ampx sandbox`, the types will be generated. Create a typed client helper:

Create `ZuriSpend/src/services/amplifyClient.ts`:

```typescript
import { generateClient } from "aws-amplify/data";
import type { Schema } from "../../amplify/data/resource";

export const client = generateClient<Schema>();
```

### 3.5 Create Seed Data Script

Create `amplify/data/seed-categories.ts` for seeding default categories:

```typescript
/**
 * Default categories to seed for new users
 * Run this after user signs up for the first time
 */

export const defaultCategories = [
  { name: "Food", icon: "🍽️", color: "#F59E0B" },
  { name: "Transport", icon: "🚃", color: "#3B82F6" },
  { name: "Lodging", icon: "🏨", color: "#8B5CF6" },
  { name: "Activities", icon: "⛷️", color: "#10B981" },
  { name: "Shopping", icon: "🛍️", color: "#EC4899" },
];
```

### 3.6 Create Category Seeding Hook

Create `ZuriSpend/src/hooks/useSeedCategories.ts`:

```typescript
import { useEffect, useState } from "react";
import { client } from "../services/amplifyClient";
import { defaultCategories } from "../../amplify/data/seed-categories";

export function useSeedCategories() {
  const [isSeeding, setIsSeeding] = useState(false);
  const [isSeeded, setIsSeeded] = useState(false);

  useEffect(() => {
    seedCategoriesIfNeeded();
  }, []);

  const seedCategoriesIfNeeded = async () => {
    try {
      setIsSeeding(true);

      // Check if user already has categories
      const { data: existingCategories } = await client.models.Category.list();

      if (existingCategories && existingCategories.length > 0) {
        setIsSeeded(true);
        return;
      }

      // Seed default categories
      await Promise.all(
        defaultCategories.map((cat) =>
          client.models.Category.create({
            name: cat.name,
            icon: cat.icon,
            color: cat.color,
          }),
        ),
      );

      setIsSeeded(true);
    } catch (error) {
      console.error("Failed to seed categories:", error);
    } finally {
      setIsSeeding(false);
    }
  };

  return { isSeeding, isSeeded };
}
```

## Data Model Notes

### Authorization

- Both `Category` and `Expense` use owner-based authorization
- Each user can only see and modify their own data
- The `owner` field is automatically managed by Amplify

### Field Changes from Mock

- `category` → `categoryId` (references Category.id)
- `invoiceUri` → `invoiceKey` (S3 key, not full URL)

### Relationships

- Expenses reference categories by `categoryId`
- No explicit GraphQL relationship defined (kept simple for flexibility)

## Verification

- [ ] `npx ampx sandbox` deploys successfully
- [ ] DynamoDB tables created (check AWS Console)
- [ ] Types generated in `amplify_outputs.json`
- [ ] Client can be imported without errors

## Next Step

Proceed to `04-expense-service.md` to implement the Amplify expense service.
