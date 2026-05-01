---
name: mobile-development
description: Build cross-platform mobile apps with React Native and Expo — navigation, native APIs, styling, deployment, and performance
---

## What I do

I build cross-platform mobile applications:

- **React Native + Expo** — Project setup, development workflow, OTA updates
- **Navigation** — Stack, tab, and drawer navigation with Expo Router
- **Native APIs** — Camera, push notifications, geolocation, haptics
- **Styling** — StyleSheet, NativeWind, platform-specific code
- **Performance** — FlashList, memoization, image optimization
- **Deployment** — App Store, Google Play, EAS Build and Submit

## When to use me

Use this skill when:
- Building a new mobile app from scratch
- Adding mobile features to an existing React Native app
- Setting up push notifications or deep linking
- Optimizing mobile app performance
- Deploying to app stores
- Implementing native functionality (camera, sensors, etc.)

## Project setup

### Expo initialization

```bash
# Create new project
npx create-expo-app MyApp --template blank-typescript

# Or with navigation template
npx create-expo-app MyApp --template tabs

# Install dependencies
cd MyApp
npx expo install expo-router react-native-safe-area-context
n```

### Expo Router file structure

```
app/
├── _layout.tsx          # Root layout (providers, theme)
├── index.tsx            # Home screen (/)
├── (tabs)/
│   ├── _layout.tsx      # Tab layout
│   ├── index.tsx        # Tab 1 (/)
│   ├── explore.tsx      # Tab 2 (/explore)
│   └── profile.tsx      # Tab 3 (/profile)
├── (auth)/
│   ├── login.tsx        # (/login)
│   └── register.tsx     # (/register)
└── [id].tsx             # Dynamic route (/123)
```

### Root layout

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { StatusBar } from 'expo-status-bar';
import { ThemeProvider } from '@/contexts/ThemeContext';

export default function RootLayout() {
  return (
    <ThemeProvider>
      <Stack>
        <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
        <Stack.Screen name="(auth)" options={{ headerShown: false }} />
        <Stack.Screen name="[id]" options={{ title: 'Details' }} />
      </Stack>
      <StatusBar style="auto" />
    </ThemeProvider>
  );
}
```

## Navigation patterns

### Stack navigation

```tsx
// app/(tabs)/_layout.tsx
import { Stack } from 'expo-router';

export default function TabLayout() {
  return (
    <Stack
      screenOptions={{
        headerStyle: { backgroundColor: '#fff' },
        headerTintColor: '#000',
        headerTitleStyle: { fontWeight: 'bold' },
      }}
    >
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen
        name="details"
        options={({ route }) => ({ title: route.params?.title ?? 'Details' })}
      />
    </Stack>
  );
}
```

### Tab navigation

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName: string;

          if (route.name === 'index') {
            iconName = focused ? 'home' : 'home-outline';
          } else if (route.name === 'explore') {
            iconName = focused ? 'search' : 'search-outline';
          } else {
            iconName = focused ? 'person' : 'person-outline';
          }

          return <Ionicons name={iconName as any} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
      })}
    >
      <Tabs.Screen name="index" options={{ title: 'Home' }} />
      <Tabs.Screen name="explore" options={{ title: 'Explore' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile' }} />
    </Tabs>
  );
}
```

## Native APIs

### Push notifications

```tsx
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

async function registerForPushNotifications() {
  if (!Device.isDevice) {
    alert('Push notifications require a physical device');
    return;
  }

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    alert('Failed to get push token for push notification!');
    return;
  }

  const token = (await Notifications.getExpoPushTokenAsync()).data;

  // Send token to server
  await fetch('/api/push-token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ token }),
  });
}

// Handle incoming notifications
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});
```

### Camera

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';
import { useState, useRef } from 'react';

function CameraComponent() {
  const [permission, requestPermission] = useCameraPermissions();
  const [photo, setPhoto] = useState<string | null>(null);
  const cameraRef = useRef<CameraView>(null);

  if (!permission?.granted) {
    return <Button onPress={requestPermission} title="Grant permission" />;
  }

  const takePhoto = async () => {
    const photo = await cameraRef.current?.takePictureAsync({
      quality: 0.8,
      base64: true,
    });
    setPhoto(photo?.uri ?? null);
  };

  return (
    <View style={styles.container}>
      <CameraView style={styles.camera} ref={cameraRef}>
        <Button onPress={takePhoto} title="Take Photo" />
      </CameraView>
    </View>
  );
}
```

## Styling

### StyleSheet patterns

```tsx
import { StyleSheet, Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: '#fff',
    ...Platform.select({
      ios: { paddingTop: 50 },
      android: { paddingTop: 20 },
    }),
  },
  card: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
      },
      android: {
        elevation: 4,
      },
    }),
  },
  text: {
    fontSize: 16,
    lineHeight: 24,
    color: '#1a1a1a',
  },
});
```

### NativeWind (Tailwind for React Native)

```tsx
// tailwind.config.js
module.exports = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  theme: {
    extend: {},
  },
};

// Component
function Card({ title, description }: { title: string; description: string }) {
  return (
    <View className="bg-white rounded-xl p-4 shadow-md">
      <Text className="text-lg font-bold text-gray-900">{title}</Text>
      <Text className="text-sm text-gray-600 mt-1">{description}</Text>
    </View>
  );
}
```

## Performance

### FlashList for large lists

```tsx
import { FlashList } from '@shopify/flash-list';

function UserList({ users }: { users: User[] }) {
  return (
    <FlashList
      data={users}
      renderItem={({ item }) => <UserCard user={item} />}
      estimatedItemSize={80}
      keyExtractor={item => item.id}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
    />
  );
}

// Memoize list items
const UserCard = memo(function UserCard({ user }: { user: User }) {
  return (
    <View style={styles.card}>
      <Image source={{ uri: user.avatar }} style={styles.avatar} />
      <Text>{user.name}</Text>
    </View>
  );
});
```

### Image optimization

```tsx
import { Image } from 'expo-image';

function OptimizedImage({ source }: { source: string }) {
  return (
    <Image
      source={{ uri: source }}
      style={{ width: 200, height: 200 }}
      contentFit="cover"
      transition={200}
      cachePolicy="memory-disk"
    />
  );
}
```

## Deployment

### EAS Build

```bash
# Install EAS CLI
npm install -g eas-cli

# Configure project
eas login
eas build:configure

# Build for production
eas build --platform ios --profile production
eas build --platform android --profile production

# OTA updates
npx expo export --platform ios
npx eas update --branch production --message "Bug fixes"
```

### eas.json

```json
{
  "cli": {
    "version": ">= 7.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "autoIncrement": true
    }
  }
}
```

## Quality checklist

- [ ] Request permissions before using native APIs (camera, location, notifications)
- [ ] Handle permission denials gracefully
- [ ] Test on both iOS and Android physical devices
- [ ] Use FlashList instead of FlatList for large lists
- [ ] Memoize heavy components to prevent unnecessary re-renders
- [ ] Optimize images with expo-image or react-native-fast-image
- [ ] Implement proper error boundaries
- [ ] Use platform-specific code where needed (`Platform.OS`)
- [ ] Test offline behavior and network failures
- [ ] Implement deep linking for all routes
- [ ] Use EAS Update for critical bug fixes

## Anti-patterns I avoid

- Using `ScrollView` for long lists — use `FlatList` or `FlashList`
- Not handling permission denials — app crashes or hangs
- Inline styles in render — use `StyleSheet` or NativeWind
- Not testing on real devices — simulators miss many issues
- Blocking the main thread with heavy computations
- Not optimizing images — large unoptimized images kill performance
- Using `console.log` in production — impacts performance
- Hardcoding dimensions without considering different screen sizes
- Not handling keyboard avoiding for input fields
- Using `setState` in loops or rapid events without debouncing