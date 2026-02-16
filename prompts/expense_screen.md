Build the AddExpense screen for ZuriSpend. It should also handle editing when an expense ID is passed via route params.

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

Screen layout (modal presentation, slides up):

1. Header:
   - "Add Expense" or "Edit Expense" title
   - X button (top left) to close
   - Save button (top right) in Swiss red (#E63946)

2. Amount input:
   - Large, centered, 48px font size
   - "CHF" prefix in slate (#64748B)
   - Numeric keyboard

3. Category picker:
   - Grid of category options (2 rows, scrollable)
   - Each shows icon + name
   - Unselected: white bg, slate border
   - Selected: category color bg, white text
   - Use category colors from design specs

4. Description input:
   - Single line text input
   - Placeholder:
