# Step 8: Testing and Deployment

Final testing checklist and deployment instructions for the Amplify Gen 2 backend.

## Local Testing

### 8.1 Start Amplify Sandbox

```bash
cd ZuriSpend
npx ampx sandbox
```

This starts a local development environment connected to real AWS resources.

### 8.2 Start the Expo App

In a separate terminal:

```bash
cd ZuriSpend
npx expo start
```

### 8.3 Test Checklist

#### Authentication

- [ ] Sign up with new email
- [ ] Receive confirmation code via email
- [ ] Confirm sign up with code
- [ ] Sign in with credentials
- [ ] Sign out
- [ ] Protected routes redirect to sign in
- [ ] Authenticated routes accessible after sign in

#### Categories

- [ ] Default categories seeded for new users
- [ ] Categories load on home screen
- [ ] Category filter works correctly
- [ ] Categories display in add expense form

#### Expenses

- [ ] Create new expense
- [ ] View expense list
- [ ] Filter expenses by category
- [ ] View expense details
- [ ] Edit existing expense
- [ ] Delete expense with confirmation
- [ ] Expenses persist after app restart

#### Invoices/Receipts

- [ ] Take photo for receipt
- [ ] Choose photo from library
- [ ] Upload receipt to S3
- [ ] View receipt in expense details
- [ ] View receipt in full-screen viewer
- [ ] Remove receipt from expense
- [ ] Share receipt

#### Statistics

- [ ] Monthly totals calculate correctly
- [ ] Category breakdown displays
- [ ] Month navigation works
- [ ] Comparison to previous month shows

#### Error Handling

- [ ] Network errors display friendly message
- [ ] Retry functionality works
- [ ] Empty states display correctly
- [ ] Loading indicators show during operations

## Deployment

### 8.4 Deploy to AWS

When ready to deploy to production:

```bash
npx ampx pipeline-deploy --branch main
```

Or set up CI/CD with AWS Amplify Hosting.

### 8.5 Environment Configuration

Create environment-specific configurations:

#### Development

```bash
npx ampx sandbox
```

#### Production

```bash
npx ampx pipeline-deploy --branch main
```

### 8.6 Configure Amplify Hosting (Optional)

For full CI/CD deployment:

1. Connect your Git repository to AWS Amplify Console
2. Amplify will automatically detect the Gen 2 backend
3. Configure build settings for Expo/React Native web builds
4. Set up branch-based deployments (dev, staging, prod)

### 8.7 Mobile App Distribution

For iOS/Android distribution:

```bash
# Build for iOS
npx expo build:ios

# Build for Android
npx expo build:android

# Or use EAS Build
npx eas build --platform all
```

## Post-Deployment Checklist

- [ ] Verify Cognito User Pool in AWS Console
- [ ] Verify DynamoDB tables created
- [ ] Verify S3 bucket for invoices
- [ ] Test authentication flow in production
- [ ] Test data operations in production
- [ ] Test file uploads in production
- [ ] Monitor CloudWatch logs for errors

## Cleanup

### Remove Sandbox Resources

```bash
npx ampx sandbox delete
```

### Remove All Amplify Resources

```bash
npx ampx pipeline-deploy --branch main --delete
```

## Monitoring

### CloudWatch Logs

- Lambda function logs
- API Gateway logs
- Cognito authentication events

### CloudWatch Metrics

- API latency
- Error rates
- Storage usage

### Cost Monitoring

- Set up AWS Budgets
- Monitor DynamoDB capacity
- Monitor S3 storage costs

## Security Checklist

- [ ] Cognito password policy configured
- [ ] MFA enabled (optional)
- [ ] S3 bucket not publicly accessible
- [ ] DynamoDB tables use owner-based authorization
- [ ] API uses Cognito User Pool authentication
- [ ] Signed URLs expire appropriately

## Troubleshooting

### Common Issues

#### "User not authenticated" error

- Ensure Amplify is configured before accessing protected resources
- Check that the user session is valid

#### "Access Denied" on S3

- Verify storage access rules in `amplify/storage/resource.ts`
- Ensure the identity ID path matches the upload path

#### Data not syncing

- Check network connectivity
- Verify DynamoDB table permissions
- Check CloudWatch logs for errors

#### Categories not loading

- Ensure categories are seeded for the user
- Check the `useSeedCategories` hook is running

### Debug Mode

Enable verbose logging:

```typescript
import { Amplify } from "aws-amplify";

Amplify.configure(outputs, {
  // Enable debug logging
  ssr: false,
});

// Or use console logging in services
console.log("API Response:", data);
```

## Migration Complete! 🎉

The ZuriSpend app is now fully integrated with AWS Amplify Gen 2:

- ✅ Cognito authentication
- ✅ DynamoDB data storage
- ✅ S3 invoice storage
- ✅ Owner-based authorization
- ✅ Type-safe API client

The mock service layer has been replaced with real cloud infrastructure while maintaining the same interface, allowing for easy testing and development.
