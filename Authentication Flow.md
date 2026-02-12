# Authentication Flow - Hướng dẫn Ngắn Gọn

## Vấn đề cần giải quyết

Ứng dụng có đăng nhập sẽ có **2 trạng thái**:

```
┌─────────────────────────────────────┐
│  Chưa đăng nhập → Màn hình Login   │
│  Đã đăng nhập   → Màn hình App     │
└─────────────────────────────────────┘
```

**Mục tiêu:** Tự động chuyển đúng màn hình dựa trên trạng thái đăng nhập.

---

## Giải pháp: Sử dụng Redirect

### ❌ KHÔNG làm thế này (như React Navigation)

```typescript
// Không hoạt động với Expo Router
function RootLayout() {
  const isLoggedIn = checkAuth();
  return isLoggedIn ? <AppScreen /> : <LoginScreen />;
}
```

### ✅ Phải làm thế này (với Expo Router)

```typescript
function ProtectedLayout() {
  const isLoggedIn = checkAuth();
  
  if (!isLoggedIn) {
    return <Redirect href="/login" />;
  }
  
  return <Stack />;
}
```

---

## Bước 1: Tạo cấu trúc thư mục

### Trước khi có Auth

```
app/
├── _layout.tsx
├── index.tsx
├── profile.tsx
└── settings.tsx
```

**Vấn đề:** Không có nơi check auth!

### Sau khi thêm Auth

```
app/
├── _layout.tsx              ← Root (không redirect)
│
├── (protected)/             ← Nhóm các screen cần đăng nhập
│   ├── _layout.tsx         ← CHECK AUTH Ở ĐÂY!
│   ├── index.tsx
│   ├── profile.tsx
│   └── settings.tsx
│
└── login.tsx               ← Public screen
```

**Tại sao?**
- Root layout KHÔNG thể dùng redirect
- `(protected)/_layout.tsx` chạy TRƯỚC KHI các screen bên trong được render
- Nếu chưa đăng nhập → redirect về login
- Nếu đã đăng nhập → cho phép vào app

---

## Bước 2: Code từng file

### File 1: Root Layout

**`app/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      {/* Protected screens - ẩn header */}
      <Stack.Screen 
        name="(protected)" 
        options={{ 
          headerShown: false,
          animation: 'none' // Không animation
        }} 
      />
      
      {/* Login screen */}
      <Stack.Screen 
        name="login" 
        options={{ 
          headerShown: false,
          animation: 'none'
        }} 
      />
    </Stack>
  );
}
```

### File 2: Protected Layout

**`app/(protected)/_layout.tsx`**

```typescript
import { Stack } from 'expo-router';
import { Redirect } from 'expo-router';

export default function ProtectedLayout() {
  const isLoggedIn = false; // Tạm thời hardcode
  
  // Chưa đăng nhập → quay về login
  if (!isLoggedIn) {
    return <Redirect href="/login" />;
  }
  
  // Đã đăng nhập → cho vào app
  return <Stack />;
}
```

**Test:**
- `isLoggedIn = false` → Mở app → Thấy màn login
- `isLoggedIn = true` → Mở app → Thấy màn app

### File 3: Login Screen

**`app/login.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';

export default function LoginScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Đăng nhập</Text>
      
      <Pressable style={styles.button}>
        <Text style={styles.buttonText}>Đăng nhập</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 30,
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

## Bước 3: Quản lý State với Context

### Tại sao cần Context?

```
login.tsx cần SET trạng thái
         ↓
    ┌─────────┐
    │ Context │ ← State chung
    └─────────┘
         ↑
(protected)/_layout.tsx cần READ trạng thái
```

### Tạo Auth Context

**`utils/auth-context.tsx`**

```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. Định nghĩa kiểu dữ liệu
interface AuthState {
  isLoggedIn: boolean;
  login: () => void;
  logout: () => void;
}

// 2. Tạo Context
const AuthContext = createContext<AuthState>({
  isLoggedIn: false,
  login: () => {},
  logout: () => {},
});

// 3. Provider để wrap app
export function AuthProvider({ children }: { children: ReactNode }) {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  
  const login = () => {
    console.log('✅ Đăng nhập thành công');
    setIsLoggedIn(true);
  };
  
  const logout = () => {
    console.log('👋 Đã đăng xuất');
    setIsLoggedIn(false);
  };
  
  return (
    <AuthContext.Provider value={{ isLoggedIn, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// 4. Hook để dùng Context
export function useAuth() {
  return useContext(AuthContext);
}
```

### Wrap App với Provider

**`app/_layout.tsx`** (updated)

```typescript
import { Stack } from 'expo-router';
import { AuthProvider } from '@/utils/auth-context';

export default function RootLayout() {
  return (
    <AuthProvider>  {/* ← Wrap toàn bộ */}
      <Stack>
        <Stack.Screen name="(protected)" options={{ headerShown: false, animation: 'none' }} />
        <Stack.Screen name="login" options={{ headerShown: false, animation: 'none' }} />
      </Stack>
    </AuthProvider>
  );
}
```

### Dùng Context ở Protected Layout

**`app/(protected)/_layout.tsx`** (updated)

```typescript
import { Stack } from 'expo-router';
import { Redirect } from 'expo-router';
import { useAuth } from '@/utils/auth-context';

export default function ProtectedLayout() {
  const { isLoggedIn } = useAuth(); // ← Đọc từ Context
  
  if (!isLoggedIn) {
    return <Redirect href="/login" />;
  }
  
  return <Stack />;
}
```

### Dùng Context ở Login Screen

**`app/login.tsx`** (updated)

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useRouter } from 'expo-router';
import { useAuth } from '@/utils/auth-context';

export default function LoginScreen() {
  const router = useRouter();
  const { login } = useAuth();
  
  const handleLogin = () => {
    login(); // Set isLoggedIn = true
    router.replace('/'); // Chuyển về home (thay thế login screen)
  };
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Đăng nhập</Text>
      
      <Pressable onPress={handleLogin} style={styles.button}>
        <Text style={styles.buttonText}>Đăng nhập</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 30,
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

### Thêm Logout

**`app/(protected)/settings.tsx`**

```typescript
import { View, Text, Pressable, StyleSheet } from 'react-native';
import { useRouter } from 'expo-router';
import { useAuth } from '@/utils/auth-context';

export default function SettingsScreen() {
  const router = useRouter();
  const { logout } = useAuth();
  
  const handleLogout = () => {
    logout(); // Set isLoggedIn = false
    router.replace('/login'); // Về login
  };
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Cài đặt</Text>
      
      <Pressable onPress={handleLogout} style={styles.button}>
        <Text style={styles.buttonText}>Đăng xuất</Text>
      </Pressable>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#FF3B30',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: '600',
  },
});
```

---

## Bước 4: Lưu trạng thái với AsyncStorage

### Vấn đề

```
User đăng nhập → Đóng app → Mở lại app
                           ↓
                    Phải đăng nhập lại 😓
```

**Nguyên nhân:** State lưu trong RAM → Mất khi tắt app

**Giải pháp:** Lưu vào AsyncStorage (giống localStorage)

### Install AsyncStorage

```bash
npx expo install @react-native-async-storage/async-storage
```

### Update Auth Context

**`utils/auth-context.tsx`** (updated)

```typescript
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

const AUTH_KEY = '@auth_state';

interface AuthState {
  isLoggedIn: boolean;
  isReady: boolean; // ← MỚI: Đã load xong chưa?
  login: () => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthState>({
  isLoggedIn: false,
  isReady: false,
  login: async () => {},
  logout: async () => {},
});

export function AuthProvider({ children }: { children: ReactNode }) {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [isReady, setIsReady] = useState(false);
  
  // Load state khi app khởi động
  useEffect(() => {
    loadAuthState();
  }, []);
  
  const loadAuthState = async () => {
    try {
      const value = await AsyncStorage.getItem(AUTH_KEY);
      
      if (value) {
        const state = JSON.parse(value);
        setIsLoggedIn(state.isLoggedIn);
      }
    } catch (error) {
      console.error('Load failed:', error);
    } finally {
      setIsReady(true); // Đánh dấu đã load xong
    }
  };
  
  const login = async () => {
    setIsLoggedIn(true);
    
    try {
      await AsyncStorage.setItem(AUTH_KEY, JSON.stringify({ isLoggedIn: true }));
    } catch (error) {
      console.error('Save failed:', error);
    }
  };
  
  const logout = async () => {
    setIsLoggedIn(false);
    
    try {
      await AsyncStorage.setItem(AUTH_KEY, JSON.stringify({ isLoggedIn: false }));
    } catch (error) {
      console.error('Save failed:', error);
    }
  };
  
  return (
    <AuthContext.Provider value={{ isLoggedIn, isReady, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

### Update Protected Layout

**`app/(protected)/_layout.tsx`** (updated)

```typescript
import { Stack } from 'expo-router';
import { Redirect } from 'expo-router';
import { useAuth } from '@/utils/auth-context';

export default function ProtectedLayout() {
  const { isLoggedIn, isReady } = useAuth();
  
  // ⏳ Chưa load xong → Đợi
  if (!isReady) {
    return null;
  }
  
  // ❌ Chưa đăng nhập → Redirect
  if (!isLoggedIn) {
    return <Redirect href="/login" />;
  }
  
  // ✅ Đã đăng nhập → Vào app
  return <Stack />;
}
```

**Tại sao cần `isReady`?**

```
Không có isReady:
─────────────────
1. App khởi động
   → isLoggedIn = false (default)
   
2. Protected layout render
   → if (!isLoggedIn) → Redirect ❌ (Sai!)
   
3. AsyncStorage trả về (100ms sau)
   → isLoggedIn = true
   → Nhưng đã redirect rồi!

Có isReady:
───────────
1. App khởi động
   → isReady = false
   
2. Protected layout render
   → if (!isReady) → return null ⏳ (Đợi)
   
3. AsyncStorage trả về
   → isLoggedIn = true
   → isReady = true
   
4. Re-render
   → Vào app ✅
```

---

## Bước 5: Giữ Splash Screen khi loading

### Vấn đề

```
App khởi động
  ↓
Splash biến mất
  ↓
⬜ Màn trắng (đang load AsyncStorage)
  ↓
App hiển thị
```

### Giải pháp

```
App khởi động
  ↓
🎨 Splash (đang load)
  ↓
App hiển thị
```

### Code

**`utils/auth-context.tsx`** (updated)

```typescript
import * as SplashScreen from 'expo-splash-screen';

// Ngăn splash tự ẩn
SplashScreen.preventAutoHideAsync();

export function AuthProvider({ children }: { children: ReactNode }) {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [isReady, setIsReady] = useState(false);
  
  // Load state
  useEffect(() => {
    loadAuthState();
  }, []);
  
  // Ẩn splash khi ready
  useEffect(() => {
    if (isReady) {
      SplashScreen.hideAsync();
    }
  }, [isReady]);
  
  // ... rest of code
}
```

**Timeline:**

```
0ms    → SplashScreen.preventAutoHideAsync()
         🎨 Splash hiển thị
         
100ms  → Đang load AsyncStorage...
         🎨 Splash vẫn hiển thị
         
1000ms → Load xong
         → setIsReady(true)
         → SplashScreen.hideAsync()
         ✅ App hiển thị
```

---

## Ví dụ hoàn chỉnh

### Kịch bản 1: Lần đầu mở app

```
1. User mở app
   ↓
2. AuthProvider load state
   → AsyncStorage.getItem() → null (lần đầu)
   → isLoggedIn = false
   → isReady = true
   ↓
3. Protected layout render
   → if (!isLoggedIn) → Redirect to /login
   ↓
4. Hiển thị màn login
```

### Kịch bản 2: User đăng nhập

```
1. User nhấn "Đăng nhập"
   ↓
2. handleLogin() gọi
   → login() trong context
   → setIsLoggedIn(true)
   → AsyncStorage.setItem({ isLoggedIn: true })
   ↓
3. router.replace('/')
   ↓
4. Protected layout re-render
   → isLoggedIn = true
   → return <Stack />
   ↓
5. Hiển thị app
```

### Kịch bản 3: Mở lại app (đã đăng nhập)

```
1. User mở app
   ↓
2. AuthProvider load state
   → AsyncStorage.getItem()
   → { isLoggedIn: true }
   → setIsLoggedIn(true)
   → setIsReady(true)
   ↓
3. Protected layout render
   → isReady = true
   → isLoggedIn = true
   → return <Stack />
   ↓
4. Hiển thị app (KHÔNG cần login lại)
```

### Kịch bản 4: User đăng xuất

```
1. User nhấn "Đăng xuất"
   ↓
2. handleLogout() gọi
   → logout() trong context
   → setIsLoggedIn(false)
   → AsyncStorage.setItem({ isLoggedIn: false })
   ↓
3. router.replace('/login')
   ↓
4. Protected layout re-render
   → isLoggedIn = false
   → Redirect to /login
   ↓
5. Hiển thị màn login
```

---

## Tips quan trọng

### 1. Dùng `replace` thay vì `push`

```typescript
// ❌ SAI
router.push('/'); // User có thể swipe back về login

// ✅ ĐÚNG
router.replace('/'); // Thay thế login screen
```

### 2. Tắt animation cho auth flow

```typescript
<Stack.Screen 
  name="login" 
  options={{ animation: 'none' }} // ← Không animation
/>
```

### 3. Luôn check `isReady` trước `isLoggedIn`

```typescript
if (!isReady) return null; // ← TRƯỚC

if (!isLoggedIn) return <Redirect />; // ← SAU
```

### 4. Dùng try-catch với AsyncStorage

```typescript
try {
  await AsyncStorage.setItem(key, value);
} catch (error) {
  console.error('Storage error:', error);
}
```

---

## Mở rộng: Login với API

```typescript
const login = async (email: string, password: string) => {
  try {
    // Gọi API
    const response = await fetch('https://api.example.com/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });
    
    const data = await response.json();
    
    if (response.ok) {
      // Lưu token
      setIsLoggedIn(true);
      await AsyncStorage.setItem(AUTH_KEY, JSON.stringify({
        isLoggedIn: true,
        token: data.token,
        user: data.user,
      }));
      
      return true;
    } else {
      throw new Error(data.message);
    }
  } catch (error) {
    console.error('Login failed:', error);
    return false;
  }
};
```

### Dùng trong Login Screen

```typescript
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');
const [loading, setLoading] = useState(false);

const handleLogin = async () => {
  setLoading(true);
  
  const success = await login(email, password);
  
  setLoading(false);
  
  if (success) {
    router.replace('/');
  } else {
    Alert.alert('Lỗi', 'Đăng nhập thất bại');
  }
};
```

---

## Tổng kết

### Checklist

- [ ] Tạo folder `(protected)/` cho routes cần auth
- [ ] Tạo `AuthContext` với `isLoggedIn`, `isReady`
- [ ] Wrap app với `<AuthProvider>`
- [ ] Check auth trong `(protected)/_layout.tsx`
- [ ] Lưu state vào `AsyncStorage`
- [ ] Giữ splash screen khi loading
- [ ] Dùng `router.replace()` khi login/logout
- [ ] Tắt animation cho auth screens

### Key Points

1. **Redirect > Conditional Rendering** trong Expo Router
2. **Protected Layout** = nơi check auth
3. **isReady** = đợi load state xong
4. **AsyncStorage** = lưu state vĩnh viễn
5. **SplashScreen** = UX mượt mà

---

## Debug

### Xem AsyncStorage

```typescript
const viewStorage = async () => {
  const value = await AsyncStorage.getItem('@auth_state');
  console.log('Storage:', value);
};
```

### Xóa Storage (test)

```typescript
const clearStorage = async () => {
  await AsyncStorage.clear();
  console.log('Đã xóa storage');
};
```

### Console log flow

```typescript
console.log('[ProtectedLayout]', { isLoggedIn, isReady });
```

---

## Pattern tương tự: Onboarding

Dùng **CÙNG pattern** cho onboarding!

```
app/
├── (onboarding)/
│   ├── _layout.tsx      ← Check: Đã xem onboarding chưa?
│   ├── step1.tsx
│   └── step2.tsx
│
└── (protected)/
    ├── _layout.tsx      ← Check: Đã đăng nhập chưa?
    └── ...
```

Flow:

```
Lần đầu → Onboarding → Login → App
Lần sau → App (skip onboarding & login)
```