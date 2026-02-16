Build the Expense Details screen for ZuriSpend. This screen displays full details of a single expense and allows editing or deleting it.

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

Screen layout (stack navigation, pushed from Home):

1. Header:
   - Use native Stack header from Expo Router
   - Title: "Expense Details"
   - Back button with "Back" label (not route name)

2. Category badge (centered, top of content):
   - Pill shape with category color background
   - Category icon + name in white text
   - Border radius 24px

3. Amount display:
   - Large, centered, 48px font size, bold
   - Format: "CHF 45.50"
   - Alpine navy color (#1E3A5F)

4. Detail cards:
   - Description card:
     - White background, 12px radius, 1px border (#E2E8F0)
     - "DESCRIPTION" label in slate, uppercase, small
     - Description text below in primary text color
   - Date card:
     - Same styling as description card
     - "DATE" label
     - Formatted date: "Monday, January 20, 2026"

5. Action buttons (bottom of content, side by side):
   - Edit button:
     - Alpine navy background (#1E3A5F)
     - White text "Edit"
     - Navigates to AddExpense with expense ID param
   - Delete button:
     - Swiss red background (#E63946)
     - White text "Delete"
     - Shows confirmation alert before deleting
     - On confirm: delete expense and navigate back

6. States:
   - Loading: centered ActivityIndicator
   - Not found: centered "Expense not found" message

7. Behavior:
   - Fetch expense by ID from route params using expenseService
   - Use useFocusEffect to refresh data when returning from edit
   - Delete confirmation: "Delete Expense" title, warning message, Cancel/Delete buttons

Route: /expense/[id] (dynamic route with expense ID)
