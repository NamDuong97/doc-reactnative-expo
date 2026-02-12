# Protected Routes - Expo Router SDK 53

## Giới thiệu

**Expo SDK 53** giới thiệu cách mới để quản lý auth flows: **Protected Routes**.

### So sánh với cách cũ

**Cách cũ (Redirect-based):**
```typescript
if (!isLoggedIn) {
  return ;
}
return ;
```

**Cách mới (Protected Routes):**
```typescript
<Stack.Protected guard={isLoggedIn}>
  <Stack.Screen name="Home" />
  <Stack.Screen name="(Settings)" />
</Stack.Protected>
```

**Lưu ý:** Cách cũ vẫn hoạt động bình thường! Bạn có thể dùng cả 2.

---

## Cách hoạt động của Protected Routes

### Khái niệm cốt lõi

**Protected Routes** = Tạo "rào chắn" xung quanh nhóm screens với điều kiện:

```
┌──────────────────────────────────┐
│ Stack.Protected                  │
│ guard={isLoggedIn}              │
│                                  │
│  ┌────────────────────────────┐ │
│  │ Screen chỉ truy cập được   │ │
│  │ khi guard = true           │ │
│  └────────────────────────────┘ │
└──────────────────────────────────┘
```

### Ví dụ đơn giản

```typescript
<Stack>
 {/* Chỉ truy cập được khi isLoggedIn = true */}
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" />
  </Stack.Protected>
  
  {/* Chỉ truy cập được khi isLoggedIn = false */}
  <Stack.Protected guard={!isLoggedIn}>
    <Stack.Screen name="signin" />
  </Stack.Protected>
</Stack>
```

---

## Bước 1: Setup cơ bản

### Cấu trúc thư mục

```
app/
├── _layout.tsx           ← Root layout
├── (tabs)/               ← Logged-in view
│   ├── _layout.tsx
│   ├── index.tsx        (Home)
│   └── settings.tsx
└── signin.tsx           ← Logged-out view
```

### Root Layout với Protected Routes

**File: `app/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';

export default function RootLayout() {
  const isLoggedIn = false; // Tạm thời hardcode
  
  return (
    <Stack>
      {/* Protected: Chỉ khi đã đăng nhập */}
      <Stack.Protected guard={isLoggedIn}>
        <Stack.Screen 
          name="(tabs)" 
          options={{ headerShown: false }} 
        />
      </Stack.Protected>
      
      {/* Protected: Chỉ khi chưa đăng nhập */}
      <Stack.Protected guard={!isLoggedIn}>
        <Stack.Screen 
          name="signin" 
          options={{ headerShown: false }} 
        />
      </Stack.Protected>
    </Stack>
  );
}
```

**Cách hoạt động:**

```
isLoggedIn = false
  ↓
App khởi động → Tìm index route (/)
  ↓
Index route nằm trong (tabs) → guard = false
  ↓
Không truy cập được!
  ↓
Tìm route tiếp theo không bị guard
  ↓
signin → guard = true
  ↓
✅ Hiển thị signin screen
```

```
isLoggedIn = true
  ↓
App khởi động → Tìm index route (/)
  ↓
Index route nằm trong (tabs) → guard = true
  ↓
✅ Hiển thị tabs layout
```

---

## Bước 2: Tạo Tabs Layout

**File: `app/(tabs)/_layout.tsx`**

```typescript
import { Tabs } from 'expo-router';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen 
        name="index" 
        options={{ title: 'Home' }} 
      />
      <Tabs.Screen 
        name="settings" 
        options={{ title: 'Settings' }} 
      />
    </Tabs>
  );
}
```

**File: `app/(tabs)/index.tsx`**

```typescript
import { View, Text, StyleSheet } from 'react-native';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Home Screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    fontSize: 24,
    fontWeight: 'bold',
  },
});
```

**File: `app/(tabs)/settings.tsx`**

```typescript
import { View, Text, StyleSheet } from 'react-native';

export default function SettingsScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Settings Screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    fontSize: 24,
    fontWeight: 'bold',
  },
});
```

---

## Bước 3: Tạo SignIn Screen

**File: `app/signin.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';

export default function SignInScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sign In</Text>
      
      <Pressable style={styles.button}>
        <Text style={styles.buttonText}>Login</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 40,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 40,
    paddingVertical: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

---

## Bước 4: State Management với Zustand

### Tại sao dùng Zustand?

- Nhẹ hơn Redux
- API đơn giản hơn Context
- Tích hợp dễ dàng với persist

### Install dependencies

```bash
npx expo install zustand expo-secure-store
```

### Tạo Auth Store

**File: `utils/auth-store.ts`**

```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import * as SecureStore from 'expo-secure-store';

// Define interface
interface AuthState {
  // State
  isLoggedIn: boolean;
  
  // Actions
  login: () => void;
  logout: () => void;
}

// Custom storage adapter cho Expo SecureStore
const secureStorage = {
  getItem: async (name: string): Promise => {
    return await SecureStore.getItemAsync(name);
  },
  setItem: async (name: string, value: string): Promise => {
    await SecureStore.setItemAsync(name, value);
  },
  removeItem: async (name: string): Promise => {
    await SecureStore.deleteItemAsync(name);
  },
};

// Create store
export const useAuthStore = create()(
  persist(
    (set) => ({
      // Initial state
      isLoggedIn: false,
      
      // Actions
      login: () => set({ isLoggedIn: true }),
      logout: () => set({ isLoggedIn: false }),
    }),
    {
      name: 'auth-store', // Key trong SecureStore
      storage: createJSONStorage(() => secureStorage),
    }
  )
);
```

**Giải thích:**

```typescript
persist(
  (set) => ({
    isLoggedIn: false,  // ← State
    login: () => set({ isLoggedIn: true }),  // ← Action
  }),
  {
    name: 'auth-store',  // ← Tên key lưu trong SecureStore
    storage: createJSONStorage(() => secureStorage),  // ← Adapter
  }
)
```

**So sánh với AsyncStorage:**

| | AsyncStorage | SecureStore |
|---|--------------|-------------|
| **Bảo mật** | Không mã hóa | Mã hóa (iOS Keychain, Android Keystore) |
| **Use case** | Preferences, cache | Tokens, passwords, sensitive data |
| **Performance** | Nhanh hơn | Chậm hơn chút |

---

## Bước 5: Sử dụng Zustand Store

### Update Root Layout

**File: `app/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';
import { useAuthStore } from '@/utils/auth-store';

export default function RootLayout() {
  const isLoggedIn = useAuthStore((state) => state.isLoggedIn);
  
  return (
    <Stack>
      <Stack.Protected guard={isLoggedIn}>
        <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
      </Stack.Protected>
      
      <Stack.Protected guard={!isLoggedIn}>
        <Stack.Screen name="signin" options={{ headerShown: false }} />
      </Stack.Protected>
    </Stack>
  );
}
```

### Update SignIn Screen

**File: `app/signin.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useAuthStore } from '@/utils/auth-store';

export default function SignInScreen() {
  const login = useAuthStore((state) => state.login);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sign In</Text>
      
      <Pressable onPress={login} style={styles.button}>
        <Text style={styles.buttonText}>Login</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 40,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 40,
    paddingVertical: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

### Update Settings Screen (Logout)

**File: `app/(tabs)/settings.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useAuthStore } from '@/utils/auth-store';

export default function SettingsScreen() {
  const logout = useAuthStore((state) => state.logout);
  
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Settings Screen</Text>
      
      <Pressable onPress={logout} style={styles.logoutButton}>
        <Text style={styles.buttonText}>Logout</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  text: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 30,
  },
  logoutButton: {
    backgroundColor: '#FF3B30',
    paddingHorizontal: 40,
    paddingVertical: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

---

## Multiple Public Routes

### Kịch bản: Có nhiều màn hình public

```
app/
├── (tabs)/         ← Logged in
└── signin.tsx      ← Not logged in
└── signup.tsx      ← Not logged in (MỚI)
```

### Cách 1: List từng screen

**File: `app/_layout.tsx`**

```typescript
<Stack>
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
  </Stack.Protected>
  
  <Stack.Protected guard={!isLoggedIn}>
    <Stack.Screen name="signin" />
    <Stack.Screen name="signup" />
  </Stack.Protected>
</Stack>
```

### Cách 2: Dùng grouping folder

```
app/
├── (tabs)/         ← Logged in
└── (auth)/         ← Not logged in (GROUP)
    ├── _layout.tsx
    ├── signin.tsx
    └── signup.tsx
```

**File: `app/_layout.tsx`**

```typescript
<Stack>
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
  </Stack.Protected>
  
  <Stack.Protected guard={!isLoggedIn}>
    <Stack.Screen name="(auth)" options={{ headerShown: false }} />
  </Stack.Protected>
</Stack>
```

**File: `app/(auth)/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack>
      <Stack.Screen name="signin" options={{ title: 'Sign In' }} />
      <Stack.Screen name="signup" options={{ title: 'Sign Up' }} />
    </Stack>
  );
}
```

### Thứ tự ưu tiên route

**Router sẽ mở route đầu tiên KHÔNG bị guard:**

```typescript
<Stack>
  <Stack.Protected guard={false}>
    <Stack.Screen name="(tabs)" />      {/* ❌ Bị chặn */}
  </Stack.Protected>
  
  <Stack.Protected guard={true}>
    <Stack.Screen name="signup" />      {/* ✅ Mở đầu tiên */}
    <Stack.Screen name="signin" />      {/* Mở thứ 2 nếu signup bị guard */}
  </Stack.Protected>
</Stack>
```

**Muốn signin luôn mở đầu tiên?**

```typescript
// Cách 1: List signin trước
<Stack.Screen name="signin" />
<Stack.Screen name="signup" />

// Cách 2: Redirect từ signup
export default function SignUpScreen() {
  const needsSignup = false; // Check logic
  
  if (!needsSignup) {
    return <Redirect href="/signin" />;
  }
  
  return <View>...</View>;
}
```

---

## Nested Protected Routes

### Kịch bản: Signup chỉ hiện nếu cần tạo tài khoản

```typescript
<Stack>
  {/* Logged in view */}
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" />
  </Stack.Protected>
  
  {/* Not logged in view */}
  <Stack.Protected guard={!isLoggedIn}>
    {/* Signup - chỉ nếu shouldCreateAccount = true */}
    <Stack.Protected guard={shouldCreateAccount}>
      <Stack.Screen name="signup" />
    </Stack.Protected>
    
    {/* Signin - chỉ nếu shouldCreateAccount = false */}
    <Stack.Protected guard={!shouldCreateAccount}>
      <Stack.Screen name="signin" />
    </Stack.Protected>
  </Stack.Protected>
</Stack>
```

**Zustand store:**

```typescript
interface AuthState {
  isLoggedIn: boolean;
  shouldCreateAccount: boolean;
  
  login: () => void;
  logout: () => void;
  setShouldCreateAccount: (value: boolean) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      isLoggedIn: false,
      shouldCreateAccount: false,
      
      login: () => set({ isLoggedIn: true }),
      logout: () => set({ isLoggedIn: false }),
      setShouldCreateAccount: (value) => set({ shouldCreateAccount: value }),
    }),
    // ... persist config
  )
);
```

**Diagram:**

```
isLoggedIn = false, shouldCreateAccount = true
  ↓
Chỉ có thể vào: signup

isLoggedIn = false, shouldCreateAccount = false
  ↓
Chỉ có thể vào: signin

isLoggedIn = true
  ↓
Chỉ có thể vào: (tabs)
```

---

## Protected Modals

### Vấn đề với cách cũ

**Trước SDK 53:**

```
app/
├── (protected)/
│   ├── _layout.tsx      ← Redirect check
│   ├── (tabs)/
│   └── modal.tsx        ← Muốn modal chỉ cho logged-in users
```

**Phải tạo grouping folder phức tạp!**

### Giải pháp với Protected Routes

```typescript
<Stack>
  {/* Tabs - logged in only */}
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
    
    {/* Modal - CŨNG logged in only */}
    <Stack.Screen 
      name="modal" 
      options={{ presentation: 'modal' }} 
    />
  </Stack.Protected>
  
  {/* Auth screens */}
  <Stack.Protected guard={!isLoggedIn}>
    <Stack.Screen name="signin" />
  </Stack.Protected>
</Stack>
```

**Ví dụ đầy đủ:**

**File: `app/modal.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { router } from 'expo-router';

export default function ModalScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Protected Modal</Text>
      <Text style={styles.subtitle}>Chỉ logged-in users mới thấy được</Text>
      
      <Pressable onPress={() => router.back()} style={styles.button}>
        <Text style={styles.buttonText}>Close</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    marginBottom: 30,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
});
```

**File: `app/(tabs)/index.tsx`** (mở modal)

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { Link } from 'expo-router';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Home Screen</Text>
      
      <Link href="/modal" asChild>
        <Pressable style={styles.button}>
          <Text style={styles.buttonText}>Open Modal</Text>
        </Pressable>
      </Link>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
  },
});
```

**Kết quả:**

```
Logged out:
- Thử navigate to /modal → KHÔNG hiển thị (bị guard)

Logged in:
- Navigate to /modal → ✅ Hiển thị modal
```

---

## Onboarding Flow với Protected Routes

### Cấu trúc

```
app/
├── _layout.tsx
│
├── (onboarding)/        ← Onboarding flow
│   ├── _layout.tsx
│   ├── index.tsx       (Bước 1)
│   └── final.tsx       (Bước cuối)
│
├── (auth)/              ← Auth flow
│   ├── signin.tsx
│   └── signup.tsx
│
└── (tabs)/              ← Main app
    ├── index.tsx
    └── settings.tsx
```

### Zustand Store

**File: `utils/auth-store.ts`** (updated)

```typescript
interface AuthState {
  isLoggedIn: boolean;
  hasCompletedOnboarding: boolean;
  
  login: () => void;
  logout: () => void;
  completeOnboarding: () => void;
  resetOnboarding: () => void; // For debugging
}

export const useAuthStore = create()(
  persist(
    (set) => ({
      isLoggedIn: false,
      hasCompletedOnboarding: false,
      
      login: () => set({ isLoggedIn: true }),
      logout: () => set({ isLoggedIn: false }),
      completeOnboarding: () => set({ hasCompletedOnboarding: true }),
      resetOnboarding: () => set({ hasCompletedOnboarding: false }),
    }),
    {
      name: 'auth-store',
      storage: createJSONStorage(() => secureStorage),
    }
  )
);
```

### Root Layout với Onboarding

**File: `app/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';
import { useAuthStore } from '@/utils/auth-store';

export default function RootLayout() {
  const isLoggedIn = useAuthStore((state) => state.isLoggedIn);
  const hasCompletedOnboarding = useAuthStore((state) => state.hasCompletedOnboarding);
  
  return (
    <Stack>
      {/* 1. Onboarding - chỉ nếu chưa hoàn thành */}
      <Stack.Protected guard={!hasCompletedOnboarding}>
        <Stack.Screen 
          name="(onboarding)" 
          options={{ headerShown: false }} 
        />
      </Stack.Protected>
      
      {/* 2. Auth - chỉ nếu đã onboarding NHƯNG chưa login */}
      <Stack.Protected guard={hasCompletedOnboarding && !isLoggedIn}>
        <Stack.Screen 
          name="signin" 
          options={{ headerShown: false }} 
        />
      </Stack.Protected>
      
      {/* 3. Main App - chỉ nếu đã login */}
      <Stack.Protected guard={isLoggedIn}>
        <Stack.Screen 
          name="(tabs)" 
          options={{ headerShown: false }} 
        />
      </Stack.Protected>
    </Stack>
  );
}
```

### Onboarding Screens

**File: `app/(onboarding)/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';

export default function OnboardingLayout() {
  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="index" />
      <Stack.Screen name="final" />
    </Stack>
  );
}
```

**File: `app/(onboarding)/index.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { Link } from 'expo-router';

export default function OnboardingFirstScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome! 👋</Text>
      <Text style={styles.subtitle}>Bước 1/2</Text>
      
      <Link href="/(onboarding)/final" asChild>
        <Pressable style={styles.button}>
          <Text style={styles.buttonText}>Tiếp theo</Text>
        </Pressable>
      </Link>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#007AFF',
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 18,
    color: 'rgba(255,255,255,0.8)',
    marginBottom: 40,
  },
  button: {
    backgroundColor: 'white',
    paddingHorizontal: 40,
    paddingVertical: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: '#007AFF',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

**File: `app/(onboarding)/final.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useAuthStore } from '@/utils/auth-store';

export default function OnboardingFinalScreen() {
  const completeOnboarding = useAuthStore((state) => state.completeOnboarding);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Bắt đầu thôi! 🚀</Text>
      <Text style={styles.subtitle}>Bước 2/2</Text>
      
      <Pressable onPress={completeOnboarding} style={styles.button}>
        <Text style={styles.buttonText}>Hoàn tất</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#34C759',
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 18,
    color: 'rgba(255,255,255,0.8)',
    marginBottom: 40,
  },
  button: {
    backgroundColor: 'white',
    paddingHorizontal: 40,
    paddingVertical: 15,
    borderRadius: 10,
  },
  buttonText: {
    color: '#34C759',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

### Flow hoàn chỉnh

```
Lần 1: Mở app
  ↓
hasCompletedOnboarding = false
  ↓
Hiển thị onboarding
  ↓
User nhấn "Hoàn tất"
  ↓
completeOnboarding() → hasCompletedOnboarding = true
  ↓
Tự động chuyển sang signin (vì isLoggedIn = false)

Lần 2: Mở app
  ↓
hasCompletedOnboarding = true, isLoggedIn = false
  ↓
Hiển thị signin (bỏ qua onboarding)
```

---

## Conditional Bottom Tabs

### Kịch bản: Tab VIP chỉ cho premium users

```
app/(tabs)/
├── index.tsx       ← Mọi user
├── settings.tsx    ← Mọi user
└── vip.tsx         ← Chỉ VIP users
```

### Zustand Store

**File: `utils/auth-store.ts`** (updated)

```typescript
interface AuthState {
  isLoggedIn: boolean;
  isVIP: boolean;
  
  login: () => void;
  loginAsVIP: () => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      isLoggedIn: false,
      isVIP: false,
      
      login: () => set({ isLoggedIn: true, isVIP: false }),
      loginAsVIP: () => set({ isLoggedIn: true, isVIP: true }),
      logout: () => set({ isLoggedIn: false, isVIP: false }),
    }),
    {
      name: 'auth-store',
      storage: createJSONStorage(() => secureStorage),
    }
  )
);
```

### Tabs Layout với Protected Tab

**File: `app/(tabs)/_layout.tsx`**

```typescript
import { Tabs } from 'expo-router';
import { useAuthStore } from '@/utils/auth-store';

export default function TabsLayout() {
  const isVIP = useAuthStore((state) => state.isVIP);
  
  return (
    <Tabs>
      {/* Tab thường - mọi user */}
      <Tabs.Screen 
        name="index" 
        options={{ title: 'Home' }} 
      />
      
      {/* Tab VIP - chỉ VIP users */}
      <Tabs.Protected guard={isVIP}>
        <Tabs.Screen 
          name="vip" 
          options={{ 
            title: 'VIP',
            tabBarIcon: ({ color }) => <Text style={{ color }}>👑</Text>
          }} 
        />
      </Tabs.Protected>
      
      {/* Tab settings - mọi user */}
      <Tabs.Screen 
        name="settings" 
        options={{ title: 'Settings' }} 
      />
    </Tabs>
  );
}
```

**File: `app/(tabs)/vip.tsx`**

```typescript
import { View, Text, StyleSheet } from 'react-native';

export default function VIPScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.icon}>👑</Text>
      <Text style={styles.title}>VIP Lounge</Text>
      <Text style={styles.subtitle}>Chào mừng thành viên VIP!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#FFD700',
  },
  icon: {
    fontSize: 80,
    marginBottom: 20,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#000',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#333',
  },
});
```

### SignIn Screen với 2 loại login

**File: `app/signin.tsx`** (updated)

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useAuthStore } from '@/utils/auth-store';

export default function SignInScreen() {
  const login = useAuthStore((state) => state.login);
  const loginAsVIP = useAuthStore((state) => state.loginAsVIP);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Sign In</Text>
      
      {/* Regular login */}
      <Pressable onPress={login} style={styles.button}>
        <Text style={styles.buttonText}>Login (Regular)</Text>
      </Pressable>
      
      {/* VIP login */}
      <Pressable onPress={loginAsVIP} style={[styles.button, styles.vipButton]}>
        <Text style={styles.buttonText}>👑 Login (VIP)</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 40,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 40,
    paddingVertical: 15,
    borderRadius: 10,
    marginBottom: 15,
    width: '80%',
    alignItems: 'center',
  },
  vipButton: {
    backgroundColor: '#FFD700',
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

### Kết quả

```
Login Regular:
┌───────────────────────────┐
│ [Home] [Settings]         │ ← 2 tabs
└───────────────────────────┘

Login VIP:
┌───────────────────────────┐
│ [Home] [👑 VIP] [Settings]│ ← 3 tabs
└───────────────────────────┘
```

---

## So sánh: Redirect vs Protected Routes

### Redirect-based (Cách cũ)

```typescript
// app/(protected)/_layout.tsx
export default function ProtectedLayout() {
  const { isLoggedIn, isReady } = useAuth();
  
  if (!isReady) return null;
  
  if (!isLoggedIn) {
    return ;
  }
  
  return ;
}
```

**Ưu điểm:**
- Kiểm soát chi tiết
- Có thể thêm loading state
- Có thể custom redirect logic

**Nhược điểm:**
- Cần grouping folders
- Code dài hơn
- Khó quản lý nhiều điều kiện

### Protected Routes (Cách mới)

```typescript
// app/_layout.tsx
<Stack.Protected guard={isLoggedIn}>
  <Stack.Screen name="(tabs)" />
</Stack.Protected>

<Stack.Protected guard={!isLoggedIn}>
  <Stack.Screen name="signin" />
</Stack.Protected>
```

**Ưu điểm:**
- Code ngắn gọn
- Dễ đọc, dễ maintain
- Không cần grouping folders cho modals
- Nested protection dễ dàng
- Conditional tabs

**Nhược điểm:**
- Ít kiểm soát hơn
- Không có loading state built-in

---

## Best Practices

### 1. Luôn persist state

```typescript
// ❌ BAD: State không persist
const [isLoggedIn, setIsLoggedIn] = useState(false);

// ✅ GOOD: Dùng Zustand với persist
export const useAuthStore = create()(
  persist(
    (set) => ({
      isLoggedIn: false,
      login: () => set({ isLoggedIn: true }),
    }),
    { name: 'auth-store', storage: secureStorage }
  )
);
```

### 2. Dùng SecureStore cho sensitive data

```typescript
// ❌ BAD: Dùng AsyncStorage cho tokens
await AsyncStorage.setItem('token', token);

// ✅ GOOD: Dùng SecureStore
await SecureStore.setItemAsync('token', token);
```

### 3. Clear state khi logout

```typescript
logout: () => set({ 
  isLoggedIn: false,
  isVIP: false,
  user: null,
  // Reset tất cả state liên quan
})
```

### 4. Thêm debug actions

```typescript
// Development only
resetOnboarding: () => set({ hasCompletedOnboarding: false }),
clearAuth: () => set({ isLoggedIn: false, isVIP: false }),
```

### 5. Type safety

```typescript
interface AuthState {
  // State
  isLoggedIn: boolean;
  isVIP: boolean;
  
  // Actions - đầy đủ type
  login: () => void;
  logout: () => void;
}
```

---

## Migration từ Redirect sang Protected Routes

### Before (Redirect)

```typescript
// app/_layout.tsx
<Stack>
  <Stack.Screen name="(protected)" options={{ headerShown: false }} />
  <Stack.Screen name="signin" />
</Stack>

// app/(protected)/_layout.tsx
export default function ProtectedLayout() {
  const { isLoggedIn } = useAuth();
  
  if (!isLoggedIn) {
    return <Redirect href="/signin" />;
  }
  
  return <Stack />;
}
```

### After (Protected Routes)

```typescript
// app/_layout.tsx
<Stack>
  <Stack.Protected guard={isLoggedIn}>
    <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
  </Stack.Protected>
  
  <Stack.Protected guard={!isLoggedIn}>
    <Stack.Screen name="signin" />
  </Stack.Protected>
</Stack>

// Xóa app/(protected)/_layout.tsx - không cần nữa!
```

**Lợi ích:**
- Bỏ được 1 folder level
- Code ngắn hơn
- Dễ thêm điều kiện mới

---

## Troubleshooting

### Vấn đề 1: Màn hình trắng khi khởi động

**Nguyên nhân:** State chưa load xong từ SecureStore

**Giải pháp:** Thêm loading check

```typescript
export default function RootLayout() {
  const isLoggedIn = useAuthStore((state) => state.isLoggedIn);
  const [isReady, setIsReady] = useState(false);
  
  useEffect(() => {
    // Wait for state to hydrate
    const unsubscribe = useAuthStore.persist.onFinishHydration(() => {
      setIsReady(true);
    });
    
    return unsubscribe;
  }, []);
  
  if (!isReady) {
    return null; // Or 
  }
  
  return (
    
      {/* ... */}
    
  );
}
```

### Vấn đề 2: Guards không update

**Nguyên nhân:** Component không re-render khi state thay đổi

**Giải pháp:** Đảm bảo selector đúng

```typescript
// ❌ BAD: Lấy cả object
const store = useAuthStore();


// ✅ GOOD: Chỉ lấy giá trị cần
const isLoggedIn = useAuthStore((state) => state.isLoggedIn);

```

### Vấn đề 3: SecureStore error trên web

**Nguyên nhân:** SecureStore không hoạt động trên web

**Giải pháp:** Fallback sang AsyncStorage trên web

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';
import * as SecureStore from 'expo-secure-store';
import { Platform } from 'react-native';

const storage = Platform.OS === 'web' 
  ? {
      getItem: AsyncStorage.getItem,
      setItem: AsyncStorage.setItem,
      removeItem: AsyncStorage.removeItem,
    }
  : {
      getItem: SecureStore.getItemAsync,
      setItem: SecureStore.setItemAsync,
      removeItem: SecureStore.deleteItemAsync,
    };
```

---

## Tổng kết

### Khi nào dùng Protected Routes?

✅ **Dùng Protected Routes khi:**
- App mới (SDK 53+)
- Nhiều conditional routes
- Có conditional tabs
- Muốn code ngắn gọn

✅ **Dùng Redirect khi:**
- Cần kiểm soát chi tiết
- Có custom loading state
- App đã dùng redirect (không bắt buộc migrate)

### Checklist

- [ ] Install Zustand & SecureStore
- [ ] Tạo auth store với persist
- [ ] Wrap screens trong Stack.Protected
- [ ] Set guard conditions
- [ ] Test các flows (login, logout, onboarding)
- [ ] Handle loading state
- [ ] Clear sensitive data khi logout

### Key Takeaways

1. **Protected Routes** = guards xung quanh groups of screens
2. **guard={condition}** = điều kiện để truy cập
3. **Zustand + SecureStore** = state management + encrypted storage
4. **Nested protected routes** = điều kiện lồng nhau
5. **Tabs.Protected** = conditional bottom tabs
6. **Không bắt buộc migrate** từ redirect

---

## Resources

- [Expo Router Protected Routes Docs](https://docs.expo.dev/router/reference/protected-routes/)
- [Zustand](https://github.com/pmndrs/zustand)
- [Expo SecureStore](https://docs.expo.dev/versions/latest/sdk/securestore/)

Beta
0 / 0
used queries
1