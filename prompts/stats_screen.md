Build the Stats screen for ZuriSpend expense tracker.

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

Screen layout:

1. Header section:
   - "Statistics" title in Alpine navy (#1E3A5F)
   - Month/year selector with left/right arrows to navigate between months
   - Current month displayed in the center

2. Summary card:
   - White background, 12px radius, subtle shadow
   - Total spent this month — large, bold, Alpine navy
   - Comparison to previous month (e.g., "+12% vs last month" or "-8% vs last month")
   - Use Swiss red for increase, green (#10B981) for decrease

3. Category breakdown:
   - Horizontal bar chart or pie chart showing spending by category
   - Each category uses its designated color
   - Show percentage and CHF amount for each category
   - Categories sorted by amount (highest first)

4. Category list:
   - Below the chart, list each category with:
     - Category icon and name (left)
     - Amount in CHF (right, bold)
     - Percentage of total (right, slate color)
   - White card background, 12px radius

5. Daily spending trend:
   - Simple line chart or bar chart showing daily spending for the selected month
   - X-axis: days of the month
   - Y-axis: CHF amount
   - Use Alpine navy for the chart line/bars

6. Empty state:
   - If no expenses for selected month, show friendly message
   - "No expenses recorded for this month"
   - Subtle illustration or icon

Use the MockExpenseService to fetch and aggregate expense data. Calculate totals, percentages, and comparisons dynamically.
