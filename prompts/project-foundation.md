Create a React Native Expo app called "ZuriSpend" — an expense tracker with Swiss-inspired minimal design.

Design specs for ZuriSpend:

- Primary: #1E3A5F (Alpine navy)
- Accent: #E63946 (Swiss red) — used for FAB, CTAs, delete actions
- Background: #FAFAFA (snow white)
- Surface: #FFFFFF (cards)
- Text Primary: #1E293B
- Text Secondary: #64748B (slate)
- Category colors: Food #F59E0B, Transport #3B82F6, Lodging #8B5CF6, Activities #10B981, Shopping #EC4899
- Border radius: 12px for cards, 8px for buttons, 24px for pills
- Shadows: subtle, rgba(0,0,0,0.08)
- Font: System default (San Francisco on iOS, Roboto on Android)
- Currency: CHF (Swiss Francs)

Setup requirements:

- TypeScript strict mode
- Expo Router with bottom tab navigation (Home, Stats tabs)
- Folder structure: src/components, src/screens, src/services, src/types, src/hooks, src/theme

Create:

1. theme/constants.ts — all colors, spacing (4, 8, 12, 16, 24, 32), border radius values
2. types/index.ts — Expense {id, amount, category, description, date, currency}, Category {id, name, icon, color}
3. services/expenseService.ts — interface IExpenseService with methods: getExpenses(), getExpenseById(id), addExpense(expense), updateExpense(expense), deleteExpense(id), getCategories(). Create MockExpenseService implementing this with 12 sample expenses in CHF.
4. Shell screens (Home, Stats, AddExpense, ExpenseDetail) with just the background color and screen title

The service layer must be abstract — it will be replaced with AWS Amplify Gen 2 later.
