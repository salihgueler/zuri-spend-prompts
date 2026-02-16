# Step 7: Error Handling and Loading States

Add comprehensive error handling and improve loading states throughout the app.

## Tasks

### 7.1 Create Error Boundary Component

Create `ZuriSpend/src/components/ErrorBoundary.tsx`:

```typescript
import { Component, ReactNode } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { colors, typography, spacing, borderRadius } from '../theme/constants';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <View style={styles.container}>
          <Text style={styles.icon}>⚠️</Text>
          <Text style={styles.title}>Something went wrong</Text>
          <Text style={styles.message}>
            {this.state.error?.message ?? 'An unexpected error occurred'}
          </Text>
          <TouchableOpacity style={styles.button} onPress={this.handleRetry}>
            <Text style={styles.buttonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.background,
    alignItems: 'center',
    justifyContent: 'center',
    padding: spacing.xl,
  },
  icon: {
    fontSize: 64,
    marginBottom: spacing.lg,
  },
  title: {
    fontSize: typography.sizes.xl,
    fontWeight: typography.weights.bold,
    color: colors.textPrimary,
    marginBottom: spacing.sm,
  },
  message: {
    fontSize: typography.sizes.md,
    color: colors.textSecondary,
    textAlign: 'center',
    marginBottom: spacing.xl,
  },
  button: {
    backgroundColor: colors.primary,
    paddingHorizontal: spacing.xl,
    paddingVertical: spacing.md,
    borderRadius: borderRadius.sm,
  },
  buttonText: {
    color: colors.surface,
    fontSize: typography.sizes.md,
    fontWeight: typography.weights.semibold,
  },
});
```

### 7.2 Create Toast/Alert Hook

Create `ZuriSpend/src/hooks/useToast.ts`:

```typescript
import { Alert } from "react-native";

interface ToastOptions {
  title: string;
  message?: string;
  type?: "success" | "error" | "info";
}

export function useToast() {
  const showToast = ({ title, message, type = "info" }: ToastOptions) => {
    const icon = type === "success" ? "✓" : type === "error" ? "✕" : "ℹ";
    Alert.alert(`${icon} ${title}`, message);
  };

  const showError = (error: unknown) => {
    const message =
      error instanceof Error ? error.message : "An error occurred";
    showToast({ title: "Error", message, type: "error" });
  };

  const showSuccess = (title: string, message?: string) => {
    showToast({ title, message, type: "success" });
  };

  return { showToast, showError, showSuccess };
}
```

### 7.3 Create Loading Component

Create `ZuriSpend/src/components/LoadingOverlay.tsx`:

```typescript
import { View, ActivityIndicator, Text, StyleSheet, Modal } from 'react-native';
import { colors, typography, spacing } from '../theme/constants';

interface LoadingOverlayProps {
  visible: boolean;
  message?: string;
}

export function LoadingOverlay({ visible, message }: LoadingOverlayProps) {
  if (!visible) return null;

  return (
    <Modal transparent visible={visible} animationType="fade">
      <View style={styles.overlay}>
        <View style={styles.container}>
          <ActivityIndicator size="large" color={colors.primary} />
          {message && <Text style={styles.message}>{message}</Text>}
        </View>
      </View>
    </Modal>
  );
}

const styles = StyleSheet.create({
  overlay: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.5)',
    alignItems: 'center',
    justifyContent: 'center',
  },
  container: {
    backgroundColor: colors.surface,
    padding: spacing.xl,
    borderRadius: 12,
    alignItems: 'center',
    minWidth: 150,
  },
  message: {
    marginTop: spacing.md,
    fontSize: typography.sizes.md,
    color: colors.textPrimary,
  },
});
```

### 7.4 Create Empty State Component

Create `ZuriSpend/src/components/EmptyState.tsx`:

```typescript
import { View, Text, StyleSheet } from 'react-native';
import { colors, typography, spacing } from '../theme/constants';

interface EmptyStateProps {
  icon: string;
  title: string;
  message: string;
}

export function EmptyState({ icon, title, message }: EmptyStateProps) {
  return (
    <View style={styles.container}>
      <Text style={styles.icon}>{icon}</Text>
      <Text style={styles.title}>{title}</Text>
      <Text style={styles.message}>{message}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: spacing.xl,
  },
  icon: {
    fontSize: 64,
    marginBottom: spacing.lg,
  },
  title: {
    fontSize: typography.sizes.lg,
    fontWeight: typography.weights.semibold,
    color: colors.textPrimary,
    marginBottom: spacing.sm,
    textAlign: 'center',
  },
  message: {
    fontSize: typography.sizes.md,
    color: colors.textSecondary,
    textAlign: 'center',
  },
});
```

### 7.5 Update useExpenses with Better Error Handling

Update `ZuriSpend/src/hooks/useExpenses.ts`:

```typescript
import { useState, useEffect, useCallback } from "react";
import { Expense, Category } from "../types";
import { expenseService } from "../services";

interface UseExpensesResult {
  expenses: Expense[];
  categories: Category[];
  isLoading: boolean;
  isRefreshing: boolean;
  error: string | null;
  refresh: () => Promise<void>;
  getCategoryById: (id: string) => Category | undefined;
  retry: () => void;
}

export function useExpenses(): UseExpensesResult {
  const [expenses, setExpenses] = useState<Expense[]>([]);
  const [categories, setCategories] = useState<Category[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [isRefreshing, setIsRefreshing] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const loadData = useCallback(async (isRefresh = false) => {
    try {
      if (isRefresh) {
        setIsRefreshing(true);
      } else {
        setIsLoading(true);
      }
      setError(null);

      const [expensesData, categoriesData] = await Promise.all([
        expenseService.getExpenses(),
        expenseService.getCategories(),
      ]);

      setExpenses(expensesData);
      setCategories(categoriesData);
    } catch (err) {
      const message =
        err instanceof Error ? err.message : "Failed to load data";
      setError(message);
      console.error("useExpenses error:", err);
    } finally {
      setIsLoading(false);
      setIsRefreshing(false);
    }
  }, []);

  useEffect(() => {
    loadData();
  }, [loadData]);

  const refresh = useCallback(async () => {
    await loadData(true);
  }, [loadData]);

  const retry = useCallback(() => {
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
    isRefreshing,
    error,
    refresh,
    getCategoryById,
    retry,
  };
}
```

### 7.6 Wrap App with Error Boundary

Update `ZuriSpend/app/_layout.tsx`:

```typescript
import { ErrorBoundary } from '../src/components/ErrorBoundary';

export default function RootLayout() {
  return (
    <ErrorBoundary>
      <AuthProvider>
        <RootLayoutNav />
      </AuthProvider>
    </ErrorBoundary>
  );
}
```

### 7.7 Add Error State to Home Screen

Update `ZuriSpend/app/(tabs)/index.tsx`:

```typescript
import { EmptyState } from '../../src/components/EmptyState';

export default function HomeScreen() {
  const { expenses, categories, isLoading, isRefreshing, error, refresh, retry } = useExpenses();

  if (error && !isLoading) {
    return (
      <View style={styles.container}>
        <EmptyState
          icon="⚠️"
          title="Failed to load expenses"
          message={error}
        />
        <TouchableOpacity style={styles.retryButton} onPress={retry}>
          <Text style={styles.retryButtonText}>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }

  // ... rest of component
}
```

### 7.8 Update Component Exports

Update `ZuriSpend/src/components/index.ts`:

```typescript
export { CategoryPill } from "./CategoryPill";
export { ExpenseCard } from "./ExpenseCard";
export { FloatingActionButton } from "./FloatingActionButton";
export { InvoicePicker } from "./InvoicePicker";
export { ErrorBoundary } from "./ErrorBoundary";
export { LoadingOverlay } from "./LoadingOverlay";
export { EmptyState } from "./EmptyState";
```

### 7.9 Update Hook Exports

Update `ZuriSpend/src/hooks/index.ts`:

```typescript
export { useExpenses } from "./useExpenses";
export { useSeedCategories } from "./useSeedCategories";
export { useToast } from "./useToast";
```

## Error Handling Patterns

### API Errors

```typescript
try {
  await expenseService.addExpense(data);
  showSuccess("Expense added");
} catch (error) {
  showError(error);
}
```

### Network Errors

The Amplify client handles network errors automatically. Wrap operations in try-catch to display user-friendly messages.

### Validation Errors

Handle validation before API calls to provide immediate feedback.

## Verification

- [ ] Error boundary catches rendering errors
- [ ] API errors display user-friendly messages
- [ ] Loading states show during data fetching
- [ ] Empty states display when no data
- [ ] Retry functionality works
- [ ] Toast/alerts display correctly

## Next Step

Proceed to `08-testing-deployment.md` for testing and deployment instructions.
