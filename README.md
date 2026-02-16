# ZuriSpend

A React Native Expo expense tracker with Swiss-inspired minimal design, powered by AWS Amplify Gen 2.

## Demo

<video src="https://github.com/salihgueler/zuri-spend-prompts/blob/main/assets/zurispend.mov?raw=true" controls width="300"></video>

## Prerequisites

- Node.js 18+
- Expo CLI (`npm install -g expo-cli`)
- AWS Account with appropriate permissions (for Amplify backend)
- AWS CLI configured (optional, for debugging)

### iOS (macOS only)

- Xcode (latest version from the Mac App Store)
- Xcode Command Line Tools (`xcode-select --install`)
- iOS Simulator (installed via Xcode → Settings → Platforms → iOS)
- CocoaPods (`sudo gem install cocoapods`)

### Android

- Android Studio (latest stable)
- Android SDK (installed via Android Studio → SDK Manager)
  - Android SDK Platform (API 34 or latest)
  - Android SDK Build-Tools
  - Android Emulator
  - Android SDK Platform-Tools
- Java Development Kit (JDK) 17 (`brew install openjdk@17` on macOS)
- `ANDROID_HOME` environment variable pointing to your SDK location (typically `~/Library/Android/sdk` on macOS)
- An Android Virtual Device created via Android Studio → Device Manager

## What the Prompts Build

The `prompts/` folder contains step-by-step instructions to build ZuriSpend from scratch:

1. **Project Foundation** (`project-foundation.md`) — Expo app scaffolding, theme, types, abstract service layer with mock data, and shell screens.
2. **Home Screen** (`home_screen.md`) — Expense list grouped by date, category filter pills, pull-to-refresh, and a floating action button.
3. **Add/Edit Expense** (`expense_screen.md`) — Modal form with amount input, category picker, description, and date selection.
4. **Expense Details** (`expense_details_screen.md`) — Full expense view with edit and delete actions.
5. **Invoice/Receipt View** (`invoice_view.md`) — Image picker for attaching receipts, full-screen viewer with zoom, and storage service integration.
6. **Statistics** (`stats_screen.md`) — Monthly spending breakdown by category with charts and month-to-month comparison.

### Amplify Gen 2 Migration (`prompts/amplify-migration/`)

Migrates the app from mock data to a real AWS backend:

| Step | Description                                                 |
| ---- | ----------------------------------------------------------- |
| 01   | Initialize Amplify Gen 2 project                            |
| 02   | Cognito authentication (sign up, sign in, protected routes) |
| 03   | DynamoDB data model for expenses and categories             |
| 04   | Amplify expense service replacing the mock                  |
| 05   | S3 storage for receipt/invoice images                       |
| 06   | UI updates for the new data structure                       |
| 07   | Error boundary, loading states, empty states                |
| 08   | Testing checklist and production deployment                 |

## Tech Stack

- React Native + Expo + Expo Router
- TypeScript (strict mode)
- AWS Amplify Gen 2 (Cognito, DynamoDB, S3, AppSync)
