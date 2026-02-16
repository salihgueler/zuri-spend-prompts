Build the Home screen for ZuriSpend expense tracker.

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

The screen should have:

1. Header section:
   - "ZuriSpend" title in Alpine navy (#1E3A5F)
   - Current month/year below it in slate (#64748B)
   - Total spent this month — large, bold, Alpine navy

2. Category filter:
   - Horizontal scroll of pills (All + each category)
   - Unselected: white background, slate border
   - Selected: category's own color as background, white text
   - Border radius 24px, padding 8px 16px

3. Expense list:
   - Grouped by date ("Today", "Yesterday", or "Mon, Jan 15")
   - Each ExpenseCard: white background, 12px radius, subtle shadow
   - Card shows: category icon (left), description + category name (middle), amount in CHF (right, bold)
   - Tapping card navigates to ExpenseDetail

4. Pull-to-refresh functionality

5. Floating Action Button:
   - Bottom right, 56px, Swiss red (#E63946)
   - White "+" icon
   - Navigates to AddExpense

Use the MockExpenseService. Filter expenses when category pill is tapped.
