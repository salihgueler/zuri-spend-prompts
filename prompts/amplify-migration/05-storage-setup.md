# Step 5: Configure S3 Storage for Invoices

Set up Amplify Storage (S3) for invoice image uploads.

## Tasks

### 5.1 Define Storage Resource

Create `amplify/storage/resource.ts`:

```typescript
import { defineStorage } from "@aws-amplify/backend";

export const storage = defineStorage({
  name: "zurispend-invoices",
  access: (allow) => ({
    "invoices/{entity_id}/*": [
      allow.entity("identity").to(["read", "write", "delete"]),
    ],
  }),
});
```

### 5.2 Update Backend Definition

Update `amplify/backend.ts`:

```typescript
import { defineBackend } from "@aws-amplify/backend";
import { auth } from "./auth/resource";
import { data } from "./data/resource";
import { storage } from "./storage/resource";

defineBackend({
  auth,
  data,
  storage,
});
```

### 5.3 Create Storage Service

Create `ZuriSpend/src/services/storageService.ts`:

```typescript
import { uploadData, getUrl, remove } from "aws-amplify/storage";
import { fetchAuthSession } from "aws-amplify/auth";
import * as FileSystem from "expo-file-system";

export interface IStorageService {
  uploadInvoice(expenseId: string, localUri: string): Promise<string>;
  getInvoiceUrl(key: string): Promise<string>;
  deleteInvoice(key: string): Promise<void>;
}

export class AmplifyStorageService implements IStorageService {
  async uploadInvoice(expenseId: string, localUri: string): Promise<string> {
    const session = await fetchAuthSession();
    const identityId = session.identityId;

    if (!identityId) {
      throw new Error("User not authenticated");
    }

    // Read file as base64
    const base64 = await FileSystem.readAsStringAsync(localUri, {
      encoding: FileSystem.EncodingType.Base64,
    });

    // Convert to blob
    const response = await fetch(`data:image/jpeg;base64,${base64}`);
    const blob = await response.blob();

    // Generate unique key
    const timestamp = Date.now();
    const key = `invoices/${identityId}/${expenseId}_${timestamp}.jpg`;

    // Upload to S3
    const result = await uploadData({
      key,
      data: blob,
      options: {
        contentType: "image/jpeg",
        onProgress: ({ transferredBytes, totalBytes }) => {
          if (totalBytes) {
            const progress = Math.round((transferredBytes / totalBytes) * 100);
            console.log(`Upload progress: ${progress}%`);
          }
        },
      },
    }).result;

    return result.key;
  }

  async getInvoiceUrl(key: string): Promise<string> {
    const result = await getUrl({
      key,
      options: {
        expiresIn: 3600, // URL valid for 1 hour
      },
    });

    return result.url.toString();
  }

  async deleteInvoice(key: string): Promise<void> {
    await remove({ key });
  }
}

export const storageService: IStorageService = new AmplifyStorageService();
```

### 5.4 Update Amplify Expense Service with Storage

Update `ZuriSpend/src/services/amplifyExpenseService.ts`:

```typescript
import { client } from "./amplifyClient";
import { storageService } from "./storageService";
import {
  Expense,
  Category,
  CreateExpenseInput,
  UpdateExpenseInput,
} from "../types";
import { IExpenseService } from "./expenseService";

export class AmplifyExpenseService implements IExpenseService {
  // ... existing methods ...

  async uploadInvoice(expenseId: string, imageUri: string): Promise<string> {
    // Upload image to S3
    const key = await storageService.uploadInvoice(expenseId, imageUri);

    // Update expense with invoice key
    await client.models.Expense.update({
      id: expenseId,
      invoiceKey: key,
    });

    return key;
  }

  async deleteInvoice(expenseId: string): Promise<boolean> {
    // Get expense to find invoice key
    const expense = await this.getExpenseById(expenseId);

    if (!expense?.invoiceKey) {
      return false;
    }

    // Delete from S3
    await storageService.deleteInvoice(expense.invoiceKey);

    // Update expense to remove invoice key
    await client.models.Expense.update({
      id: expenseId,
      invoiceKey: null,
    });

    return true;
  }

  async getInvoiceUri(expenseId: string): Promise<string | null> {
    const expense = await this.getExpenseById(expenseId);

    if (!expense?.invoiceKey) {
      return null;
    }

    // Get signed URL from S3
    return storageService.getInvoiceUrl(expense.invoiceKey);
  }
}
```

### 5.5 Update InvoicePicker Component

Update `ZuriSpend/src/components/InvoicePicker.tsx` to handle S3 URLs:

```typescript
import { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
} from 'react-native';
import { Image } from 'expo-image';
import * as ImagePicker from 'expo-image-picker';
import { colors, typography, spacing, borderRadius } from '../theme/constants';

interface InvoicePickerProps {
  imageUri?: string;
  invoiceKey?: string; // S3 key for existing invoice
  onImageSelected: (uri: string) => void;
  onImageRemoved: () => void;
  disabled?: boolean;
  getSignedUrl?: (key: string) => Promise<string>; // Function to get signed URL
}

export function InvoicePicker({
  imageUri,
  invoiceKey,
  onImageSelected,
  onImageRemoved,
  disabled = false,
  getSignedUrl,
}: InvoicePickerProps) {
  const [displayUri, setDisplayUri] = useState<string | undefined>(imageUri);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (imageUri) {
      setDisplayUri(imageUri);
    } else if (invoiceKey && getSignedUrl) {
      loadSignedUrl();
    } else {
      setDisplayUri(undefined);
    }
  }, [imageUri, invoiceKey]);

  const loadSignedUrl = async () => {
    if (!invoiceKey || !getSignedUrl) return;

    setLoading(true);
    try {
      const url = await getSignedUrl(invoiceKey);
      setDisplayUri(url);
    } catch (error) {
      console.error('Failed to load invoice URL:', error);
    } finally {
      setLoading(false);
    }
  };

  const pickImage = async () => {
    if (disabled) return;

    const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert('Permission needed', 'Please grant camera roll permissions');
      return;
    }

    const result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      allowsEditing: true,
      quality: 0.8,
    });

    if (!result.canceled && result.assets[0]) {
      onImageSelected(result.assets[0].uri);
      setDisplayUri(result.assets[0].uri);
    }
  };

  const takePhoto = async () => {
    if (disabled) return;

    const { status } = await ImagePicker.requestCameraPermissionsAsync();
    if (status !== 'granted') {
      Alert.alert('Permission needed', 'Please grant camera permissions');
      return;
    }

    const result = await ImagePicker.launchCameraAsync({
      allowsEditing: true,
      quality: 0.8,
    });

    if (!result.canceled && result.assets[0]) {
      onImageSelected(result.assets[0].uri);
      setDisplayUri(result.assets[0].uri);
    }
  };

  const showOptions = () => {
    Alert.alert('Add Receipt', 'Choose an option', [
      { text: 'Take Photo', onPress: takePhoto },
      { text: 'Choose from Library', onPress: pickImage },
      { text: 'Cancel', style: 'cancel' },
    ]);
  };

  const handleRemove = () => {
    Alert.alert('Remove Receipt', 'Are you sure?', [
      { text: 'Cancel', style: 'cancel' },
      {
        text: 'Remove',
        style: 'destructive',
        onPress: () => {
          onImageRemoved();
          setDisplayUri(undefined);
        },
      },
    ]);
  };

  if (loading) {
    return (
      <View style={styles.container}>
        <ActivityIndicator size="large" color={colors.primary} />
      </View>
    );
  }

  if (displayUri) {
    return (
      <View style={styles.imageContainer}>
        <Image source={{ uri: displayUri }} style={styles.image} />
        <TouchableOpacity
          style={styles.removeButton}
          onPress={handleRemove}
          disabled={disabled}
        >
          <Text style={styles.removeButtonText}>Remove</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <TouchableOpacity
      style={styles.container}
      onPress={showOptions}
      disabled={disabled}
    >
      <Text style={styles.icon}>📷</Text>
      <Text style={styles.text}>Add Receipt</Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  container: {
    height: 200,
    borderWidth: 2,
    borderStyle: 'dashed',
    borderColor: colors.border,
    borderRadius: borderRadius.md,
    alignItems: 'center',
    justifyContent: 'center',
  },
  icon: {
    fontSize: 48,
    marginBottom: spacing.sm,
  },
  text: {
    fontSize: typography.sizes.md,
    color: colors.textSecondary,
  },
  imageContainer: {
    borderRadius: borderRadius.md,
    overflow: 'hidden',
  },
  image: {
    width: '100%',
    height: 200,
    borderRadius: borderRadius.md,
  },
  removeButton: {
    position: 'absolute',
    bottom: spacing.sm,
    right: spacing.sm,
    backgroundColor: colors.accent,
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.sm,
    borderRadius: borderRadius.sm,
  },
  removeButtonText: {
    color: colors.surface,
    fontSize: typography.sizes.sm,
    fontWeight: typography.weights.medium,
  },
});
```

### 5.6 Install Required Dependencies

```bash
cd ZuriSpend
npm install expo-file-system
```

## Storage Access Rules

The storage configuration uses identity-based access:

- Each user can only access files in their own `invoices/{identity_id}/` folder
- Files are automatically scoped to the authenticated user
- Signed URLs expire after 1 hour for security

## Verification

- [ ] Storage resource deploys successfully
- [ ] Images upload to S3
- [ ] Signed URLs are generated correctly
- [ ] Images display in the app
- [ ] Images can be deleted
- [ ] Access is properly scoped to user

## Next Step

Proceed to `06-ui-updates.md` to update UI components for the new data structure.
