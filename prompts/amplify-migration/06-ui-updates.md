# Step 6: Update UI Components

Update all UI components to work with the new Amplify data structure.

## Tasks

### 6.1 Update Home Screen

Update `ZuriSpend/app/(tabs)/index.tsx`:

```typescript
// Key changes:
// 1. Use categoryId instead of category
// 2. Add category seeding for new users

import { useSeedCategories } from '../../src/hooks/useSeedCategories';

export default function HomeScreen() {
  const { isSeeding } = useSeedCategories();
  const { expenses, categories, isLoading, refresh, getCategoryById } = useExpenses();

  // Show loading while seeding categories for new users
  if (isSeeding) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color={colors.primary} />
        <Text style={styles.loadingText}>Setting up your account...</Text>
      </View>
    );
  }

  const filteredExpenses = useMemo(() => {
    if (!selectedCategory) return expenses;
    return expenses.filter((e) => e.categoryId === selectedCategory);
  }, [expenses, selectedCategory]);

  // ... rest of component
}
```

### 6.2 Update Stats Screen

Update `ZuriSpend/app/(tabs)/stats.tsx`:

```typescript
// Key change: Use categoryId instead of category

const categoryStats = useMemo((): CategoryStats[] => {
  const statsMap = new Map<string, number>();
  monthExpenses.forEach((e) => {
    const current = statsMap.get(e.categoryId) ?? 0;
    statsMap.set(e.categoryId, current + e.amount);
  });

  return categories
    .map((cat) => ({
      category: cat,
      amount: statsMap.get(cat.id) ?? 0,
      percentage:
        totalThisMonth > 0
          ? ((statsMap.get(cat.id) ?? 0) / totalThisMonth) * 100
          : 0,
    }))
    .filter((s) => s.amount > 0)
    .sort((a, b) => b.amount - a.amount);
}, [monthExpenses, categories, totalThisMonth]);
```

### 6.3 Update Add Expense Screen

Update `ZuriSpend/app/add-expense.tsx`:

```typescript
import { expenseService } from '../src/services';
import { storageService } from '../src/services/storageService';

export default function AddExpenseScreen() {
  // ... existing state ...
  const [invoiceKey, setInvoiceKey] = useState<string | undefined>(undefined);
  const [localInvoiceUri, setLocalInvoiceUri] = useState<string | undefined>(undefined);

  const loadData = async () => {
    setInitialLoading(true);
    try {
      const cats = await expenseService.getCategories();
      setCategories(cats);

      if (params.id) {
        const expense = await expenseService.getExpenseById(params.id);
        if (expense) {
          setAmount(expense.amount.toString());
          setSelectedCategory(expense.categoryId); // Changed from category
          setDescription(expense.description);
          setInvoiceKey(expense.invoiceKey);

          // Load signed URL for existing invoice
          if (expense.invoiceKey) {
            const url = await storageService.getInvoiceUrl(expense.invoiceKey);
            setLocalInvoiceUri(url);
          }

          const [year, month, day] = expense.date.split('-').map(Number);
          setSelectedDate(new Date(year, month - 1, day));
        }
      }
    } finally {
      setInitialLoading(false);
    }
  };

  const handleSave = async () => {
    if (!validate()) return;

    setLoading(true);
    try {
      const expenseData = {
        amount: parseFloat(amount),
        categoryId: selectedCategory!, // Changed from category
        description: description.trim() || 'No description',
        date: formatDateToYYYYMMDD(selectedDate),
        currency: 'CHF',
        invoiceKey, // Use key instead of URI
      };

      let savedExpense;
      if (isEditing && params.id) {
        savedExpense = await expenseService.updateExpense({
          ...expenseData,
          id: params.id
        });
      } else {
        savedExpense = await expenseService.addExpense(expenseData);
      }

      // Upload new invoice if selected
      if (localInvoiceUri && !invoiceKey) {
        await expenseService.uploadInvoice(savedExpense.id, localInvoiceUri);
      }

      router.back();
    } finally {
      setLoading(false);
    }
  };

  const handleImageSelected = (uri: string) => {
    setLocalInvoiceUri(uri);
    setInvoiceKey(undefined); // Clear existing key when new image selected
  };

  const handleImageRemoved = async () => {
    if (isEditing && params.id && invoiceKey) {
      await expenseService.deleteInvoice(params.id);
    }
    setLocalInvoiceUri(undefined);
    setInvoiceKey(undefined);
  };

  // In render:
  <InvoicePicker
    imageUri={localInvoiceUri}
    invoiceKey={invoiceKey}
    onImageSelected={handleImageSelected}
    onImageRemoved={handleImageRemoved}
    disabled={loading}
    getSignedUrl={storageService.getInvoiceUrl}
  />
}
```

### 6.4 Update Expense Detail Screen

Update `ZuriSpend/app/expense/[id].tsx`:

```typescript
import { storageService } from '../../src/services/storageService';

export default function ExpenseDetailScreen() {
  const [invoiceUrl, setInvoiceUrl] = useState<string | null>(null);

  const loadExpense = useCallback(async () => {
    if (!id) return;
    setLoading(true);
    try {
      const [expenseData, categories] = await Promise.all([
        expenseService.getExpenseById(id),
        expenseService.getCategories(),
      ]);
      setExpense(expenseData);

      if (expenseData) {
        const cat = categories.find((c) => c.id === expenseData.categoryId);
        setCategory(cat ?? null);

        // Load signed URL for invoice
        if (expenseData.invoiceKey) {
          const url = await storageService.getInvoiceUrl(expenseData.invoiceKey);
          setInvoiceUrl(url);
        }
      }
    } finally {
      setLoading(false);
    }
  }, [id]);

  // In render, use invoiceUrl instead of expense.invoiceUri:
  {invoiceUrl ? (
    <TouchableOpacity
      style={styles.invoiceThumbnailContainer}
      onPress={handleViewInvoice}
      activeOpacity={0.7}
    >
      <Image
        source={{ uri: invoiceUrl }}
        style={styles.invoiceThumbnail}
      />
      <Text style={styles.viewReceiptText}>Tap to view</Text>
    </TouchableOpacity>
  ) : (
    <Text style={styles.noReceiptText}>No receipt attached</Text>
  )}
}
```

### 6.5 Update Invoice Viewer Screen

Update `ZuriSpend/app/invoice/[expenseId].tsx`:

```typescript
import { storageService } from '../../src/services/storageService';

export default function InvoiceViewerScreen() {
  const [invoiceUrl, setInvoiceUrl] = useState<string | null>(null);

  const loadExpense = async () => {
    if (!expenseId) return;
    setLoading(true);
    try {
      const data = await expenseService.getExpenseById(expenseId);
      setExpense(data);

      // Load signed URL for invoice
      if (data?.invoiceKey) {
        const url = await storageService.getInvoiceUrl(data.invoiceKey);
        setInvoiceUrl(url);
      }
    } finally {
      setLoading(false);
    }
  };

  // Use invoiceUrl in render:
  if (!expense || !invoiceUrl) {
    // ... not found state
  }

  <Image
    source={{ uri: invoiceUrl }}
    style={styles.invoiceImage}
    resizeMode="contain"
  />
}
```

### 6.6 Update ExpenseCard Component

Update `ZuriSpend/src/components/ExpenseCard.tsx`:

```typescript
interface ExpenseCardProps {
  expense: Expense;
  category?: Category;
  onPress: () => void;
}

export function ExpenseCard({ expense, category, onPress }: ExpenseCardProps) {
  // Use expense.categoryId if needed for any logic
  // The category prop is already resolved by the parent

  return (
    <TouchableOpacity style={styles.card} onPress={onPress}>
      <View style={styles.iconContainer}>
        <Text style={styles.icon}>{category?.icon ?? '📝'}</Text>
      </View>
      <View style={styles.content}>
        <Text style={styles.description} numberOfLines={1}>
          {expense.description}
        </Text>
        <Text style={styles.category}>{category?.name ?? 'Unknown'}</Text>
      </View>
      <View style={styles.amountContainer}>
        <Text style={styles.amount}>
          {expense.currency} {expense.amount.toFixed(2)}
        </Text>
        {expense.invoiceKey && (
          <Text style={styles.receiptIndicator}>📎</Text>
        )}
      </View>
    </TouchableOpacity>
  );
}
```

### 6.7 Add Sign Out Button

Add a sign out option to the app. Update `ZuriSpend/app/(tabs)/_layout.tsx`:

```typescript
import { Tabs } from 'expo-router';
import { TouchableOpacity, Text, Alert } from 'react-native';
import { useAuth } from '../../src/contexts/AuthContext';
import { colors } from '../../src/theme/constants';

export default function TabLayout() {
  const { signOut } = useAuth();

  const handleSignOut = () => {
    Alert.alert('Sign Out', 'Are you sure you want to sign out?', [
      { text: 'Cancel', style: 'cancel' },
      { text: 'Sign Out', style: 'destructive', onPress: signOut },
    ]);
  };

  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: colors.primary,
        headerRight: () => (
          <TouchableOpacity onPress={handleSignOut} style={{ marginRight: 16 }}>
            <Text style={{ color: colors.accent }}>Sign Out</Text>
          </TouchableOpacity>
        ),
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color }) => <Text style={{ color }}>🏠</Text>,
        }}
      />
      <Tabs.Screen
        name="stats"
        options={{
          title: 'Stats',
          tabBarIcon: ({ color }) => <Text style={{ color }}>📊</Text>,
        }}
      />
    </Tabs>
  );
}
```

## Field Mapping Summary

| Mock Field   | Amplify Field | Notes                    |
| ------------ | ------------- | ------------------------ |
| `category`   | `categoryId`  | References Category.id   |
| `invoiceUri` | `invoiceKey`  | S3 key, needs signed URL |
| -            | `owner`       | Auto-managed by Amplify  |
| -            | `createdAt`   | Auto-managed by Amplify  |
| -            | `updatedAt`   | Auto-managed by Amplify  |

## Verification

- [ ] Home screen displays expenses correctly
- [ ] Category filtering works
- [ ] Stats screen calculates correctly
- [ ] Add/Edit expense works
- [ ] Invoice upload and display works
- [ ] Sign out works
- [ ] No TypeScript errors

## Next Step

Proceed to `07-error-handling.md` to add proper error handling and offline support.
