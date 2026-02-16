# Step 4: Implement Amplify Expense Service

Replace the MockExpenseService with a real Amplify Gen 2 implementation.

## Tasks

### 4.1 Create Amplify Expense Service

Create `ZuriSpend/src/services/amplifyExpenseService.ts`:

```typescript
import { client } from "./amplifyClient";
import {
  Expense,
  Category,
  CreateExpenseInput,
  UpdateExpenseInput,
} from "../types";
import { IExpenseService } from "./expenseService";

export class AmplifyExpenseService implements IExpenseService {
  async getExpenses(): Promise<Expense[]> {
    const { data, errors } = await client.models.Expense.list();

    if (errors) {
      throw new Error(errors[0].message);
    }

    return (data ?? [])
      .map(this.mapExpense)
      .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime());
  }

  async getExpenseById(id: string): Promise<Expense | null> {
    const { data, errors } = await client.models.Expense.get({ id });

    if (errors) {
      throw new Error(errors[0].message);
    }

    return data ? this.mapExpense(data) : null;
  }

  async addExpense(expense: CreateExpenseInput): Promise<Expense> {
    const { data, errors } = await client.models.Expense.create({
      amount: expense.amount,
      categoryId: expense.categoryId,
      description: expense.description,
      date: expense.date,
      currency: expense.currency,
      invoiceKey: expense.invoiceKey,
    });

    if (errors || !data) {
      throw new Error(errors?.[0].message ?? "Failed to create expense");
    }

    return this.mapExpense(data);
  }

  async updateExpense(expense: UpdateExpenseInput): Promise<Expense> {
    const { data, errors } = await client.models.Expense.update({
      id: expense.id,
      amount: expense.amount,
      categoryId: expense.categoryId,
      description: expense.description,
      date: expense.date,
      currency: expense.currency,
      invoiceKey: expense.invoiceKey,
    });

    if (errors || !data) {
      throw new Error(errors?.[0].message ?? "Failed to update expense");
    }

    return this.mapExpense(data);
  }

  async deleteExpense(id: string): Promise<boolean> {
    const { errors } = await client.models.Expense.delete({ id });

    if (errors) {
      throw new Error(errors[0].message);
    }

    return true;
  }

  async getCategories(): Promise<Category[]> {
    const { data, errors } = await client.models.Category.list();

    if (errors) {
      throw new Error(errors[0].message);
    }

    return (data ?? []).map(this.mapCategory);
  }

  async uploadInvoice(expenseId: string, imageUri: string): Promise<string> {
    // This will be implemented in step 05-storage-setup.md
    throw new Error("Not implemented - see storage setup");
  }

  async deleteInvoice(expenseId: string): Promise<boolean> {
    // This will be implemented in step 05-storage-setup.md
    throw new Error("Not implemented - see storage setup");
  }

  async getInvoiceUri(expenseId: string): Promise<string | null> {
    // This will be implemented in step 05-storage-setup.md
    throw new Error("Not implemented - see storage setup");
  }

  private mapExpense(data: Record<string, unknown>): Expense {
    return {
      id: data.id as string,
      amount: data.amount as number,
      categoryId: data.categoryId as string,
      description: data.description as string,
      date: data.date as string,
      currency: data.currency as string,
      invoiceKey: data.invoiceKey as string | undefined,
      owner: data.owner as string | undefined,
      createdAt: data.createdAt as string | undefined,
      updatedAt: data.updatedAt as string | undefined,
    };
  }

  private mapCategory(data: Record<string, unknown>): Category {
    return {
      id: data.id as string,
      name: data.name as string,
      icon: data.icon as string,
      color: data.color as string,
      owner: data.owner as string | undefined,
      createdAt: data.createdAt as string | undefined,
      updatedAt: data.updatedAt as string | undefined,
    };
  }
}
```

### 4.2 Update Service Interface

Update `ZuriSpend/src/services/expenseService.ts` to use the new types:

```typescript
import {
  Expense,
  Category,
  CreateExpenseInput,
  UpdateExpenseInput,
} from "../types";

export interface IExpenseService {
  getExpenses(): Promise<Expense[]>;
  getExpenseById(id: string): Promise<Expense | null>;
  addExpense(expense: CreateExpenseInput): Promise<Expense>;
  updateExpense(expense: UpdateExpenseInput): Promise<Expense>;
  deleteExpense(id: string): Promise<boolean>;
  getCategories(): Promise<Category[]>;
  uploadInvoice(expenseId: string, imageUri: string): Promise<string>;
  deleteInvoice(expenseId: string): Promise<boolean>;
  getInvoiceUri(expenseId: string): Promise<string | null>;
}

// Keep MockExpenseService for development/testing
// ... existing mock implementation ...
```

### 4.3 Create Service Factory

Create `ZuriSpend/src/services/serviceFactory.ts`:

```typescript
import { IExpenseService, MockExpenseService } from "./expenseService";
import { AmplifyExpenseService } from "./amplifyExpenseService";

type ServiceMode = "mock" | "amplify";

const SERVICE_MODE: ServiceMode = "amplify"; // Change to 'mock' for local dev without backend

export function createExpenseService(): IExpenseService {
  switch (SERVICE_MODE) {
    case "amplify":
      return new AmplifyExpenseService();
    case "mock":
    default:
      return new MockExpenseService();
  }
}

export const expenseService = createExpenseService();
```

### 4.4 Update Service Exports

Update `ZuriSpend/src/services/index.ts`:

```typescript
export { expenseService } from "./serviceFactory";
export type { IExpenseService } from "./expenseService";
export { MockExpenseService } from "./expenseService";
export { AmplifyExpenseService } from "./amplifyExpenseService";
```

### 4.5 Update useExpenses Hook

Update `ZuriSpend/src/hooks/useExpenses.ts` to handle the categoryId field:

```typescript
import { useState, useEffect, useCallback } from "react";
import { Expense, Category } from "../types";
import { expenseService } from "../services";

interface UseExpensesResult {
  expenses: Expense[];
  categories: Category[];
  isLoading: boolean;
  error: string | null;
  refresh: () => Promise<void>;
  getCategoryById: (id: string) => Category | undefined;
}

export function useExpenses(): UseExpensesResult {
  const [expenses, setExpenses] = useState<Expense[]>([]);
  const [categories, setCategories] = useState<Category[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const loadData = useCallback(async () => {
    try {
      setIsLoading(true);
      setError(null);
      const [expensesData, categoriesData] = await Promise.all([
        expenseService.getExpenses(),
        expenseService.getCategories(),
      ]);
      setExpenses(expensesData);
      setCategories(categoriesData);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Failed to load data");
    } finally {
      setIsLoading(false);
    }
  }, []);

  useEffect(() => {
    loadData();
  }, [loadData]);

  const getCategoryById = useCallback(
    (id: string): Category | undefined => categories.find((c) => c.id === id),
    [categories],
  );

  return {
    expenses,
    categories,
    isLoading,
    error,
    refresh: loadData,
    getCategoryById,
  };
}
```

### 4.6 Update Screen Components

Update references from `expense.category` to `expense.categoryId` in:

- `ZuriSpend/app/(tabs)/index.tsx`
- `ZuriSpend/app/(tabs)/stats.tsx`
- `ZuriSpend/app/add-expense.tsx`
- `ZuriSpend/app/expense/[id].tsx`

Example change in `index.tsx`:

```typescript
// Before
const filteredExpenses = useMemo(() => {
  if (!selectedCategory) return expenses;
  return expenses.filter((e) => e.category === selectedCategory);
}, [expenses, selectedCategory]);

// After
const filteredExpenses = useMemo(() => {
  if (!selectedCategory) return expenses;
  return expenses.filter((e) => e.categoryId === selectedCategory);
}, [expenses, selectedCategory]);
```

## Verification

- [ ] App compiles without TypeScript errors
- [ ] Expenses load from DynamoDB
- [ ] New expenses are saved to DynamoDB
- [ ] Expenses can be updated
- [ ] Expenses can be deleted
- [ ] Categories load correctly
- [ ] Filtering by category works

## Next Step

Proceed to `05-storage-setup.md` to configure S3 storage for invoice images.
