# PrimeSlot SMS Lab - Native Implementation Guide

This guide explains how to implement the three advanced features that require native Android code.

## Overview

The PrimeSlot SMS Lab app includes three main features that require native Android implementation:

1. **Native SMS Interception** - Capture incoming SMS messages
2. **Background Service** - Monitor SMS when app is closed
3. **Push Notifications** - Alert users of filtered messages

## Architecture

```
┌─────────────────────────────────────────┐
│   React Native / Expo App               │
│   (lib/sms-service.ts)                  │
│   (lib/notification-service.ts)         │
└────────────┬────────────────────────────┘
             │
             ├─────────────────────────────────────┐
             │                                     │
    ┌────────▼──────────┐          ┌──────────────▼─────┐
    │  SMSReceiver      │          │  SMSMonitoringService
    │  (BroadcastReceiver)         │  (Foreground Service)
    │  - Intercepts SMS │          │  - Keeps monitoring
    │  - Filters (IVACBD)          │  - Runs in background
    │  - Forwards to API│          │  - Shows notification
    └────────┬──────────┘          └──────────────┬─────┘
             │                                     │
             └────────────────┬────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │  API Endpoint      │
                    │  prime-slot.wuaze  │
                    │  /otp_server_multi │
                    └────────────────────┘
```

## 1. Native SMS Interception (SMSReceiver)

### File: `android/app/src/main/java/com/primeslot/SMSReceiver.kt`

**What it does:**
- Listens for incoming SMS using BroadcastReceiver
- Filters messages containing "(IVACBD)"
- Forwards filtered messages to the API endpoint
- Sends push notifications

**Key methods:**
- `onReceive()` - Called when SMS arrives
- `forwardMessageToAPI()` - Sends message to API
- `sendNotification()` - Triggers notification

**Implementation steps:**

1. Copy the SMSReceiver.kt file to your Android project
2. Update the package name to match your app
3. Implement HTTP client (OkHttp recommended):

```kotlin
// Add to build.gradle
dependencies {
    implementation 'com.squareup.okhttp3:okhttp:4.11.0'
}
```

4. Complete the `forwardMessageToAPI()` method:

```kotlin
private fun forwardMessageToAPI(context: Context?, phoneNumber: String, messageBody: String, timestamp: Long) {
    val client = OkHttpClient()
    val json = JSONObject().apply {
        put("message", messageBody)
        put("phoneNumber", phoneNumber)
        put("timestamp", System.currentTimeMillis())
        put("deviceId", getDeviceId(context))
    }
    
    val body = RequestBody.create(MediaType.parse("application/json"), json.toString())
    val request = Request.Builder()
        .url(API_ENDPOINT)
        .post(body)
        .build()
    
    client.newCall(request).enqueue(object : Callback {
        override fun onFailure(call: Call, e: IOException) {
            Log.e(TAG, "Failed to forward message", e)
        }
        
        override fun onResponse(call: Call, response: Response) {
            Log.d(TAG, "Message forwarded: ${response.code}")
        }
    })
}
```

## 2. Background Service (SMSMonitoringService)

### File: `android/app/src/main/java/com/primeslot/SMSMonitoringService.kt`

**What it does:**
- Runs as a Foreground Service
- Keeps SMS receiver active even when app is closed
- Shows persistent notification
- Prevents system from killing the service

**Key methods:**
- `onStartCommand()` - Starts the service
- `createNotificationChannel()` - Creates notification channel (Android 8+)
- `createNotification()` - Creates foreground notification

**Implementation steps:**

1. Copy the SMSMonitoringService.kt file to your Android project
2. Update the package name
3. The service is already properly configured

## 3. Android Manifest Configuration

### File: `android/app/src/main/AndroidManifest.xml.config`

**Required permissions:**
```xml
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.RECEIVE_SMS" />
<uses-permission android:name="android.permission.SEND_SMS" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

**Receiver configuration:**
```xml
<receiver
    android:name=".SMSReceiver"
    android:exported="true"
    android:permission="android.permission.BROADCAST_SMS">
    <intent-filter>
        <action android:name="android.provider.Telephony.SMS_RECEIVED" />
    </intent-filter>
</receiver>
```

**Service configuration:**
```xml
<service
    android:name=".SMSMonitoringService"
    android:foregroundServiceType="phone"
    android:exported="false" />
```

## 4. Starting the Service from React Native

Add this to your app's initialization:

```typescript
// In app/_layout.tsx or app/(tabs)/index.tsx

import { useEffect } from 'react';
import { Platform } from 'react-native';

export default function App() {
  useEffect(() => {
    if (Platform.OS === 'android') {
      // Start SMS monitoring service
      const startService = async () => {
        try {
          // This would require a native module to start the service
          // For now, the service starts automatically on app install
          console.log('SMS monitoring service should be running');
        } catch (error) {
          console.error('Error starting SMS service:', error);
        }
      };
      
      startService();
    }
  }, []);
  
  return (
    // ... your app content
  );
}
```

## 5. Permission Handling

Users need to grant permissions for the app to work:

```typescript
// In a settings or setup screen
import * as Permissions from 'expo-permissions';

const requestSMSPermissions = async () => {
  try {
    const { status } = await Permissions.askAsync(
      Permissions.SMS,
      Permissions.NOTIFICATIONS
    );
    
    if (status === 'granted') {
      console.log('Permissions granted');
    }
  } catch (error) {
    console.error('Error requesting permissions:', error);
  }
};
```

## 6. Testing

### On Android Device/Emulator:

1. **Test SMS Interception:**
   - Send SMS with "(IVACBD)" keyword
   - Check if message appears in app
   - Verify notification is shown

2. **Test Background Service:**
   - Close the app completely
   - Send SMS with "(IVACBD)" keyword
   - Check if notification appears
   - Verify message was forwarded to API

3. **Test API Forwarding:**
   - Monitor API logs at https://prime-slot.wuaze.com/otp_server_multi.php
   - Verify messages are being received

### Debugging:

```bash
# View Android logs
adb logcat | grep "SMSReceiver\|SMSMonitoringService"

# Test SMS sending
adb emu sms send <phone_number> "Test message (IVACBD)"
```

## 7. Building for Production

### Generate APK:

```bash
# Build APK
eas build --platform android --local

# Or using Expo CLI
expo build:android
```

### Important Notes:

1. **Permissions**: Users must grant SMS and notification permissions
2. **Battery**: Foreground service will use battery - show persistent notification
3. **API Key**: Ensure API endpoint is accessible from device
4. **Testing**: Test on real device, not just emulator
5. **Compliance**: Inform users about SMS monitoring in privacy policy

## 8. Troubleshooting

| Issue | Solution |
|-------|----------|
| SMS not intercepted | Check permissions, verify BroadcastReceiver is registered |
| Service stops | Ensure it's a Foreground Service with notification |
| Notifications not showing | Check notification permissions for Android 13+ |
| API calls failing | Verify network connectivity, check API endpoint |
| App crashes | Check logcat for errors, verify package names |

## 9. JavaScript Layer Integration

The JavaScript layer (`lib/sms-service.ts` and `lib/notification-service.ts`) handles:
- Message filtering
- Local storage
- Notification scheduling
- API communication

These work alongside the native code to provide complete SMS monitoring functionality.

## 10. Future Enhancements

- [ ] Add SMS reply functionality
- [ ] Implement message scheduling
- [ ] Add custom filtering rules
- [ ] Support multiple API endpoints
- [ ] Add offline message queue
- [ ] Implement end-to-end encryption
