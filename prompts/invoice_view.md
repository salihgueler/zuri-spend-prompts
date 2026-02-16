Build the Invoice View feature for ZuriSpend. This feature allows users to attach receipt/invoice images to expenses and view them.

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

## Data Model Updates

1. Update the Expense type to include an optional invoice field:
   - `invoiceUri?: string` — local URI or remote URL of the invoice image

2. Update IExpenseService interface to include invoice methods:
   - `uploadInvoice(expenseId: string, imageUri: string): Promise<string>` — returns the stored URI
   - `deleteInvoice(expenseId: string): Promise<boolean>`
   - `getInvoiceUri(expenseId: string): Promise<string | null>`

3. MockExpenseService implementation:
   - Store invoice URIs in memory (simulating S3/cloud storage)
   - Return the same URI passed in for mock (real implementation would upload to S3)
   - Add mock invoices to a few existing expenses for testing

## Invoice Picker Component

Create a reusable InvoicePicker component:

1. Layout:
   - Container with dashed border (#E2E8F0), 12px radius
   - Height: 200px when empty, auto when image present
   - Centered content

2. Empty state:
   - Camera icon (📷) in slate color
   - "Add Receipt" text below in slate
   - Tappable to open image picker

3. With image:
   - Display image preview filling container
   - Overlay buttons at bottom:
     - "View" button (Alpine navy) — opens full-screen viewer
     - "Remove" button (Swiss red) — removes image with confirmation

4. Image picker options (ActionSheet):
   - "Take Photo" — opens camera
   - "Choose from Library" — opens photo library
   - "Cancel"

5. Props:
   - `imageUri?: string` — current image URI
   - `onImageSelected: (uri: string) => void`
   - `onImageRemoved: () => void`
   - `disabled?: boolean`

## Invoice Viewer Screen

Create a full-screen invoice viewer (modal):

Route: /invoice/[expenseId]

1. Header:
   - "Receipt" title
   - X button to close
   - Share button (top right) — native share sheet

2. Content:
   - Full-screen image with zoom/pan support (use react-native-gesture-handler or similar)
   - Pinch to zoom
   - Double-tap to zoom in/out
   - Pan when zoomed

3. Footer:
   - Expense description text
   - Amount in CHF

## Integration Points

1. AddExpense screen:
   - Add InvoicePicker below the date picker
   - Save invoice URI with expense

2. Expense Details screen:
   - Show invoice thumbnail if present (below date card)
   - Tap thumbnail to open Invoice Viewer
   - "No receipt attached" message if none

3. ExpenseCard component (optional enhancement):
   - Small receipt icon indicator if expense has invoice

## Dependencies

Use expo-image-picker for camera/library access:

- Request permissions appropriately
- Handle permission denied gracefully
- Support both camera and media library

## Mock Implementation Notes

The mock service should:

- Store URIs in a Map<expenseId, invoiceUri>
- Simulate upload delay (200ms)
- Pre-populate 2-3 expenses with mock invoice URIs (can use placeholder image URLs)

## Amplify Gen 2 Implementation

When implementing with AWS Amplify Gen 2:

- Use Amplify Storage (S3) for invoice image uploads
- Define storage resource in `amplify/storage/resource.ts`
- Configure access rules: authenticated users can read/write their own invoices
- Use `uploadData` from `aws-amplify/storage` for uploads
- Use `getUrl` for generating signed URLs to display images
- Store the S3 key (not full URL) in the expense record
- Handle upload progress for better UX
- Implement proper error handling for network failures

## Behavior

- Image picker respects device permissions
- Show loading indicator during "upload"
- Compress images before storage (quality: 0.8, max dimension: 1920px)
- Handle errors gracefully with user-friendly messages
- Invoice is optional — expenses can be saved without one
