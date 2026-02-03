# 📘 EXPO ROUTER – CORE NOTES (LAYOUT & ROUTING)

---

## I. Tổng quan Expo Router

- Expo Router là hệ thống **file-based routing** cho Expo / React Native
- Mỗi file trong thư mục `app/` tự động trở thành một route
- Không cần khai báo screen thủ công như React Navigation
- Hiểu **layout files** là chìa khóa để hiểu toàn bộ Expo Router
- Dựa trên React Navigation nhưng đơn giản hóa cách config
- Hỗ trợ:
  - **Deep linking** tự động
  - **Type-safe routing** với TypeScript
  - **SEO-friendly** cho web
  - **Code splitting** tự động

### Ví dụ so sánh:

**React Navigation (cũ):**
```tsx
// Phải khai báo thủ công
const Stack = createNativeStackNavigator();

<Stack.Navigator>
  <Stack.Screen name="Home" component={HomeScreen} />
  <Stack.Screen name="Profile" component={ProfileScreen} />
</Stack.Navigator>
```

**Expo Router (mới):**
```
app/
  index.tsx        // Tự động thành route "/"
  profile.tsx      // Tự động thành route "/profile"
```

### Lợi ích:
- ✅ Giảm boilerplate code
- ✅ Dễ refactor (di chuyển file = di chuyển route)
- ✅ Không bị lỗi typo khi navigate
- ✅ Auto-completion cho routes trong TypeScript

---

## II. Thư mục `app/`

- Mỗi project **bắt buộc** có thư mục `app/`
- `app/` chỉ chứa:
  - Screen files (`.tsx`, `.jsx`, `.js`)
  - `_layout.tsx` files (layout configuration)
  - `+html.tsx` (custom HTML cho web - optional)
  - `+not-found.tsx` (404 page - optional)
- `app/` có thể nằm:
  - Ở root project
  - Hoặc trong `src/` (đều được hỗ trợ)
- Thư mục ngoài `app/` dùng cho:
  - `components` - UI components
  - `utils` - Helper functions
  - `hooks` - Custom React hooks
  - `services` - API calls
  - `constants` - Config & constants
  - `types` - TypeScript types

### Ví dụ cấu trúc project hoàn chỉnh:

```
  - Đại diện cho route `/` (root)
- Không thể thay màn hình mở đầu bằng cách đổi file
- Cách duy nhất để mở screen khác lúc đầu:
  - Dùng `<Redirect />`
- Dù redirect, file `index.tsx` **vẫn phải tồn tại**
- `index.tsx` không nhất thiết phải nằm ở root `app/`
  - Có thể nằm trong grouping folders

### Ví dụ 1: Index Route cơ bản

```tsx
// app/index.tsx
import { View, Text } from 'react-native';
import { Link } from 'expo-router';

export default function Home() {
  return (
    <View>
      <Text>Welcome Home!</Text>
      <Link href="/about">Go to About</Link>
    </View>
  );
}
```

### Ví dụ 2: Redirect từ Index

```tsx
// app/index.tsx
import { Redirect } from 'expo-router';
import { useAuth } from '@/hooks/useAuth';

export default function Index() {
  const { isAuthenticated } = useAuth();
  
  // Nếu đã đăng nhập → vào home
  // Nếu chưa → vào login
  return isAuthenticated ? (
    <Redirect href="/(tabs)/home" />
  ) : (
    <Redirect href="/login" />
  );
}
```

### Ví dụ 3: Index trong nested folder
**Không xuất hiện trong URL** ⭐
  - Chỉ dùng để tổ chức code
  - Giúp tạo multiple layouts cho cùng một level
- Tên trong ngoặc:
  - Không có ý nghĩa kỹ thuật đặc biệt
  - `(tabs)`, `(auth)`, `(shop)` chỉ là convention
- Grouping folder quan trọng khi:
  - Cần nhiều layout khác nhau cùng level
  - Tránh route trùng tên
  - Tổ chức code rõ ràng hơn

### Ví dụ 1: Without grouping (có vấn đề)

```
app/
  ├── _layout.tsx          # Root layout (Stack)
  ├── index.tsx            # Home
  ├── home.tsx             # ❌ Conflict! Cùng tên "home"
  └── profile.tsx
```

### Ví dụ 2: With grouping (giải quyết vấn đề)

```
app/
  ├── _layout.tsx          # Root layout
  ├── (tabs)/              # 👈 Không xuất hiện trong URL
  │   ├── _layout.tsx      # Tabs layout
  │   ├── index.tsx        # route: "/" (not "/tabs")
  │   ├── home.tsx         # route: "/home"
  │   └── profile.tsx      # route: "/profile"
  └── (auth)/              # 👈 Không xuất hiện trong URL
      ├── _layout.tsx      # Auth layout (khác với tabs)
      ├── login.tsx        # route: "/login"
      └── register.tsx     # route: "/register"
```

### Ví dụ 3: Multiple layouts cùng level

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen name="index" />
      <Tabs.Screen name="profile" />
    </Tabs>
  );
}

// app/(auth)/_layout.tsx
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="login" />
      <Stack.Screen name="register" />
    </Stack>
  );
}
```

### Kết quả:
- `/` → Tabs layout (index.tsx trong (tabs))
- `/profile` → Tabs layout
- `/login` → Auth layout (Stack, no header)
- `/register` → Auth layout

### ⚠️ Lưu ý:
- Grouping folders **CHỈ ẢNH HƯỞNG LAYOUT**, không ảnh hưởng URL
- Có thể có nhiều grouping folders cùng level
  - Là "wrapper" cho tất cả routes con
  - Quyết định kiểu navigation (Stack, Tabs, Drawer, etc.)
- Layout file được áp dụng dựa trên vị trí trong folder tree
- Screen sẽ được quản lý bởi **layout gần nhất**
- Layout chỉ render **1 lần**, các screen con render nhiều lần khi navigate

### Ví dụ 1: Root Layout (bắt buộc)

```tsx
#### a) Stack Navigator
```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack
      screenOptions={{
        headerStyle: { backgroundColor: '#f4511e' },
        headerTintColor: '#fff',
        headerTitleStyle: { fontWeight: 'bold' },
      }}
    >
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="details" options={{ title: 'Details' }} />
      <Stack.Screen 
        name="modal" 
        options={{ 
          presentation: 'modal',
          headerShown: false 
        }} 
      />
    </Stack>
  );
}
```

#### b) Tabs Navigator
- **Slot** là navigator không có style/animation
- Slot dùng để:
  - Cho phép screen con tiếp tục render
  - Wrap screens với shared components
  - Pass qua layout layer mà không apply navigation
- Slot = "cho đi tiếp" (pass-through)
- Không Slot (hoặc null) = "chặn lại" (block)

### Ví dụ 1: Shared Header với Slot

```tsx
// app/_layout.tsx
import { Slot } from 'expo-router';
import { View, Text, StyleSheet } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';

export default function Layout() {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      {/* Header hiển thị trên TẤT CẢ screens */}
      <View style={styles.header}>
        <Text style={styles.headerText}>My App</Text>
      </View>
      
      {/* Screen con render ở đây */}
      <Slot />
      
      {/* Footer hiển thị trên TẤT CẢ screens */}
      <View style={styles.footer}>
        <Text>© 2026 My Company</Text>
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  header: {
    padding: 16,
    backgroundColor: '#007AFF',
    alignItems: 'center',
  },
  headerText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  footer: {
    padding: 16,
    backgroundColor: '#f0f0f0',
    alignItems: 'center',
  },
});
```

### Ví dụ 2: Context Provider với Slot

```tsx
// app/_layout.tsx
import { Slot } from 'expo-router';
import { AuthProvider } from '@/contexts/AuthContext';
import { ThemeProvider } from '@/contexts/ThemeContext';
import { QueryClientProvider } from '@tanstack/react-query';
import { queryClient } from '@/lib/queryClient';

export default function RootLayout() {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <ThemeProvider>
          {/* Tất cả screens đều có access tới contexts */}
          <Slot />
        </ThemeProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
}
```

### Ví dụ 3: Analytics Tracking với Slot

```tsx
// app/_layout.tsx
import { Slot, usePathname } from 'expo-router';
import { useEffect } from 'react';
import * as Analytics from '@/lib/analytics';

export default function Layout() {
### 1. Link Component (Declarative)

```tsx
import { Link } from 'expo-router';

// ✅ Link cơ bản
<Link href="/about">Go to About</Link>

// ✅ Link với params
<Link href="/user/123">View User</Link>

// ✅ Link với query params
<Link href={{ pathname: "/search", params: { q: "expo" } }}>
  Search Expo
</Link>

// ✅ Link với custom style
<Link href="/profile" style={{ color: 'blue', fontSize: 16 }}>
  Profile
</Link>
```

### 2. useRouter Hook (Imperative)

```tsx
import { useRouter } from 'expo-router';

function MyComponent() {
  const router = useRouter();
  
  // ✅ Push (thêm vào stack)
  router.push('/details');
  
  // ✅ Replace (thay thế current screen)
  router.replace('/login');
  
  // ✅ Back
  router.back();
  
  // ✅ Navigate với params
  router.push({
    pathname: '/user/[id]',
    params: { id: '123', name: 'John' }
  });
  
  // ✅ Dismiss modal
  router.dismiss();
  
  // ✅ Navigate tới route cụ thể
  router.navigate('/home');
  
  // ✅ Set params cho current screen
  router.setParams({ filter: 'new' });
  
  return (
    <Button title="Go" onPress={() => router.push('/about')} />
  );
}
```

### 3. Link với asChild (Important!)

```tsx
import { Link } from 'expo-router';
import { Pressable, Text } from 'react-native';

// ❌ SAI: Custom component không nhận press event
<Link href="/about">
  <Pressable>
    <Text>Go to About</Text>
  </Pressable>
</Link>

// ✅ ĐÚNG: Dùng asChild
<Link href="/about" asChild>
  <Pressable>
    <Text>Go to About</Text>
  </Pressable>
</Link>

// ✅ Với custom button component
<Link href="/profile" asChild>
  <CustomButton title="View Profile" />
</Link>
```

### 4. Navigation với Type Safety

```tsx
import { Link } from 'expo-router';

// ✅ TypeScript sẽ check href có tồn tại không
<Link href="/about" />          // ✅ Nếu file tồn tại
<Link href="/invalid" />         // ❌ TypeScript error

// ✅ Autocomplete cho routes
<Link href="/user/" />           // TypeScript suggest các routes
```

### 5. So sánh push vs replace vs navigate

```tsx
const router = useRouter();

// push: Thêm vào stack → có thể back
router.push('/details');         // Stack: [Home, Details]
router.back();                   // → về Home

// replace: Thay thế → không thể back về screen cũ
router.replace('/login');        // Stack: [Login] (Home bị xóa)

// navigate: Smart navigation
router.navigate('/home');        // Nếu home đã trong stack → back về home
                                 // Nếu chưa → push mới
```

### 6. Ví dụ thực tế: Navigate sau khi submit form

```tsx
import { useRouter } from 'expo-router';
import { useState } from 'react';

function CreatePostScreen() {
  const router = useRouter();
  const [title, setTitle] = useState('');
  
  async function handleSubmit() {
    try {
      const post = await createPost({ title });
      
      // Replace để user không back về form
      router.replace({
        pathname: '/post/[id]',
        params: { id: post.id }
      });
    } catch (error) {
      alert('Error creating post');
    }
  }
  
  return (
    <View>
      <TextInput value={title} onChangeText={setTitle} />
      <Button title="Create" onPress={handleSubmit} />
    </View>
  );
}
```

### 7. Ví dụ: Passing complex data

```tsx
// ❌ SAI: Không nên pass object phức tạp qua params
router.push({
  pathname: '/details',
  params: { user: { name: 'John', age: 30 } } // ❌ Object sẽ bị stringify
});

// ✅ ĐÚNG: Chỉ pass primitive values (string, number)
router.push({
  pathname: '/details',
  params: { userId: '123', userName: 'John' }
});

// Hoặc fetch data ở destination screen
router.push('/details/123');

// details/[id].tsx
function DetailsScreen() {
  const { id } = useLocalSearchParams();
  const { data: user } = useQuery(['user', id], () => fetchUser(id));
  
  return <Text>{user.name}</Text>;
}
```

### 8. Conditional Navigation

```tsx
import { useRouter } from 'expo-router';
import { useAuth } from '@/hooks/useAuth';

function ActionButton() {
  const router = useRouter();
  const { isAuthenticated } = useAuth();
  
  function handlePress() {
    if (isAuthenticated) {
      router.push('/dashboard');
    } else {
      router.push('/login');
    }
  }
  
  return <Button title="Continue" onPress={handlePress} />;
}
```

### Tóm tắt:
| Method | Use Case | Back Enabled? |
|--------|----------|---------------|
| `<Link>` | Khi có touchable element | ✅ Yes |
| `router.push()` | Programmatic navigation | ✅ Yes |
| `router.replace()` | Login, success screens | ❌ No |
| `router.back()` | Custom back button | - |
| `router.navigate()` | Smart navigation | ✅ Yes |
| `router.dismiss()` | Close modal | - |

### ⚠️ Lưu ý:
- Luôn dùng `asChild` với custom components trong `<Link>`
- Không pass objects phức tạp qua params
- Dùng `replace()` khi không muốn user back về màn hình cũ
- TypeScript sẽ giúp check routes tồn tại hay không
}
```

### Ví dụ 4: Conditional Slot

```tsx
// app/(protected)/_layout.tsx
import { Slot, Redirect } from 'expo-router';
import { useAuth } from '@/hooks/useAuth';

export default function ProtectedLayout() {
  const { isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    // ❌ Không render Slot → redirect
    return <Redirect href="/login" />;
  }
  
  // ✅ Render Slot → screens có thể access
  return <Slot />;
}
```

### So sánh Slot vs Stack:

```tsx
// ❌ Với Stack: Có navigation animation, header, back button
<Stack>
  <Stack.Screen name="home" />
  <Stack.Screen name="profile" />
</Stack>

// ✅ Với Slot: Không có gì, chỉ render screen thôi
<Slot />
```

### Khi nào dùng Slot?
- ✅ Wrap screens với shared UI (header, footer)
- ✅ Inject contexts/providers cho screens con
- ✅ Setup global logic (analytics, error boundary)
- ✅ Pass-through layout layer mà không thêm navigation

### Khi nào KHÔNG dùng Slot?
- ❌ Cần navigation với animation
- ❌ Cần back button
- ❌ Cần header/tabs
- ❌ Cần screen transitions

→ Dùng Stack, Tabs, Drawer thay vì Slot
export default function TabsLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: 'blue',
        tabBarStyle: { height: 60 },
      }}
    >
      <Tabs.Screen 
        name="home" 
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen 
        name="search" 
        options={{
          title: 'Search',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="search" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

#### c) Drawer Navigator
```tsx
// app/_layout.tsx
import { Drawer } from 'expo-router/drawer';

export default function DrawerLayout() {
  return (
    <Drawer>
      <Drawer.Screen 
        name="index" 
        options={{ 
          drawerLabel: 'Home',
          title: 'Home' 
        }} 
      />
      <Drawer.Screen 
        name="settings" 
        options={{ 
          drawerLabel: 'Settings',
          title: 'Settings' 
        }} 
      />
    </Drawer>
  );
}
```

#### d) Slot Navigator (unstyled)
```tsx
// app/_layout.tsx
import { Slot } from 'expo-router';

export default function Layout() {
  return (
    <>
      {/* Shared header cho tất cả screens */}
      <View style={{ padding: 20, backgroundColor: 'blue' }}>
        <Text style={{ color: 'white' }}>My App Header</Text>
      </View>
      
      {/* Render screen con (không có navigation animation) */}
      <Slot />
    </>
  );
}
```

---

### 2. Redirect

- Redirect sẽ:
  - **Không render screen con**
  - Chuyển hướng sang route khác ngay lập tức
- Redirect:
  - Không thể dùng ở root layout
  - Dùng nhiều trong **auth flow** và **conditional routing**

#### Ví dụ: Auth Flow
```tsx
// app/(app)/_layout.tsx
import { Redirect, Stack } from 'expo-router';
import { useAuth } from '@/hooks/useAuth';

export default function AppLayout() {
  const { isAuthenticated, isLoading } = useAuth();
  
  // Đang check auth
  if (isLoading) {
    return <LoadingScreen />;
  }
  
  // Chưa login → redirect sang auth
  if (!isAuthenticated) {
    return <Redirect href="/login" />;
  }
  
  // Đã login → render screens bình thường
  return (
    <Stack>
      <Stack.Screen name="home" />
      <Stack.Screen name="profile" />
    </Stack>
  );
}
```

#### Ví dụ: Onboarding Flow
```tsx
// app/_layout.tsx
import { Redirect, Slot } from 'expo-router';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useState, useEffect } from 'react';

export default function Layout() {
  const [hasSeenOnboarding, setHasSeenOnboarding] = useState(null);
  
  useEffect(() => {
    AsyncStorage.getItem('hasSeenOnboarding').then(value => {
      setHasSeenOnboarding(value === 'true');
    });
  }, []);
  
  if (hasSeenOnboarding === null) {
    return <LoadingScreen />;
  }
  
  // Lần đầu mở app → show onboarding
  if (!hasSeenOnboarding) {
    return <Redirect href="/onboarding" />;
  }
  
  return <Slot />;
}
```

---

### 3. Component thường hoặc `null`

- Nếu layout không return navigator:
  - Screen con sẽ **không render**
  - Screen trở nên **inaccessible**
- Use case: Blocking routes tạm thời

#### Ví dụ: Maintenance Mode
```tsx
// app/(shop)/_layout.tsx
import { View, Text } from 'react-native';

export default function ShopLayout() {
  const isUnderMaintenance = true;
  
  if (isUnderMaintenance) {
    // ❌ Không return navigator → screen con không render
    return (
      <View>
        <Text>Shop is under maintenance</Text>
      </View>
    );
  }
  
  // ✅ Return navigator → screens render bình thường
  return <Stack />;
}
```

#### Ví dụ: Feature Flag
```tsx
// app/(beta)/_layout.tsx
import { Slot } from 'expo-router';
import { useFeatureFlag } from '@/hooks/useFeatureFlag';

export default function BetaLayout() {
  const isBetaEnabled = useFeatureFlag('beta-features');
  
  // Feature chưa enable → block access
  if (!isBetaEnabled) {
    return null; // ❌ Screen con không accessible
  }
  
  return <Slot />; // ✅ Cho phép access
}
```

### Tóm tắt:
| Return Type | Screen Con Render? | Use Case |
|-------------|-------------------|----------|
| **Navigator** (Stack, Tabs, Drawer) | ✅ Có | Navigation thông thường |
| **Slot** | ✅ Có (no style) | Custom layout, shared components |
| **Redirect** | ❌ Không | Auth flow, conditional routing |
| **Component / null** | ❌ Không | Maintenance mode, feature flags |
```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen 
        name="index" 
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen 
        name="profile" 
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

### Ví dụ 3: Layout với shared components

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { AuthProvider } from '@/contexts/AuthContext';
import { StatusBar } from 'expo-status-bar';

export default function RootLayout() {
  return (
    <AuthProvider>
      {/* StatusBar render cho tất cả screens */}
      <StatusBar style="auto" />
      
      <Stack>
        <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
        <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
      </Stack>
    </AuthProvider>
  );
}
```

### Flow khi navigate:

```
User mở app/profile.tsx
    ↓
1. Root _layout.tsx render      (Stack)
    ↓
2. (tabs)/_layout.tsx render    (Tabs)
    ↓
3. profile.tsx render           (Screen)
```

### ⚠️ Lưu ý:
- **Bắt buộc** phải có `app/_layout.tsx` (root layout)
- Layout không cần import screens thủ công (auto-discover)
- Mỗi folder có thể có 1 `_layout.tsx`
- Layout càng gần screen càng có priority cao
```tsx
// app/shop/index.tsx
export default function ShopHome() {
  return <Text>Shop Home Page</Text>;
}
```

### ⚠️ Lưu ý:
- `index.tsx` và `home.tsx` là 2 route khác nhau
- `index.tsx` → `/`
- `home.tsx` → `/home`roup
│       ├── _layout.tsx       # Tabs layout
│       ├── home.tsx          # Tab 1
│       └── profile.tsx       # Tab 2
├── components/               # ✅ Reusable components
│   ├── Button.tsx
│   └── Card.tsx
├── hooks/                    # ✅ Custom hooks
│   └── useAuth.tsx
├── utils/                    # ✅ Helper functions
│   └── formatDate.ts
├── services/                 # ✅ API services
│   └── api.ts
└── package.json
```

### ⚠️ Lưu ý:
- Không đặt components trong `app/` → sẽ bị hiểu nhầm là route
- Tên file bắt đầu bằng `_` hoặc `.` sẽ bị ignore (trừ `_layout.tsx`)
- Tên file không được chứa khoảng trắng

---

## III. Index Route

- Mỗi project **bắt buộc phải có index route**
- File `index.tsx` là:
  - Màn hình mở đầu khi app launch
- Không thể thay màn hình mở đầu bằng cách đổi file
- Cách duy nhất để mở screen khác lúc đầu:
  - Dùng `<Redirect />`
- Dù redirect, file `index.tsx` **vẫn phải tồn tại**
- `index.tsx` không nhất thiết phải nằm ở root `app/`
  - Có thể nằm trong grouping folders

---

## IV. Grouping Folders `( )`

- Folder có tên trong ngoặc tròn là **grouping folder**
- Grouping folder:
  - Không xuất hiện trong URL
  - Chỉ dùng để tổ chức code
- Tên trong ngoặc:
  - Không có ý nghĩa kỹ thuật đặc biệt
  - `(tabs)` chỉ là convention, không có quyền năng riêng
- Grouping folder chỉ quan trọng khi:
  - Có nhiều route trùng tên

---

## V. Layout Files `_layout.tsx`

- Một project thường có **nhiều layout files**
- Mỗi layout file:
  - Định nghĩa layout & navigation cho các screen cùng cấp
- Layout file được áp dụng dựa trên vị trí trong folder tree
- Screen sẽ được quản lý bởi **layout gần nhất**

---

## VI. Layout File có thể return gì?

### 1. Navigator (trường hợp phổ biến nhất)

- Stack
- Tabs
- Slot (unstyled navigator)

> Navigator quyết định cách các screen con được điều hướng

---

### 2. Redirect

- Redirect sẽ:
  - Không render screen con
  - Chuyển hướng sang route khác ngay
- Redirect:
  - Không thể dùng ở root layout (hiện tại)
  - Sẽ dùng nhiều trong auth flow

---

### 3. Component thường hoặc `null`

- Nếu layout không return navigator:
  - Screen con sẽ không render
  - Screen trở nên **inaccessible**

---

## VII. Slot Navigator

- Slot là navigator không style
- Slot dùng để:
  - Cho phép screen con tiếp tục render
- Slot = “cho đi tiếp”
- Không slot = “chặn lại”

---

## VIII. Sự khác biệt cốt lõi với React Navigation

- Không cần liệt kê screens trong layout
- Expo Router:
  - Tự suy ra routes từ file system
- File system chính là router config

---

## IX. Điều hướng giữa các screen

- Có 2 cách chính:
  - Dùng `<Link />`
  - Dùng `useRouter().push`
- Hành vi của `Link` và `router.push` là tương đương
- Khi dùng custom component trong `Link`:
  - Bắt buộc dùng `asChild`
- Nếu không dùng `asChild`:
  - Component con sẽ không nhận press event

---

## X. Cách xác định Route Path

- Route được tạo từ:
  - Tên folder
  - Tên file
- `index.tsx` đại diện cho route gốc của folder
- Grouping folders không ảnh hưởng URL
- Route path bắt đầu tính từ thư mục `app/`

---

## XI. Router Sitemap

- Có thể dùng sitemap để:
  - Xem toàn bộ routes tồn tại trong app
- Sitemap phản ánh chính xác cấu trúc router hiện tại

---

## XII. Route Lồng Sâu (Nested Routes)

- Folder lồng sâu chỉ ảnh hưởng:
  - Đường dẫn URL
- Việc lồng folder **không tự tạo layout**
- Không phải folder nào cũng cần layout file

---

## XIII. Cách Expo Router tìm Layout (RẤT QUAN TRỌNG)

- Khi render một screen:
  - Router tìm layout trong cùng folder
  - Nếu không có → tìm folder cha
  - Tiếp tục cho đến root layout
- Layout đầu tiên được tìm thấy sẽ quyết định navigator

---

## XIV. Thứ tự thực thi Layout

- Layout được thực thi:
  - Từ ngoài vào trong
- Flow:
  1. Root layout
  2. Layout cha
  3. Layout gần screen nhất
- Nếu layout ở giữa:
  - Không return navigator → dừng render
  - Redirect → chuyển hướng ngay

---

## XV. Redirect trong Layout

- Redirect ở layout gần screen:
  - Có thể chặn screen render
  - Ép điều hướng sang route khác
- Redirect luôn có hiệu lực trước khi screen render

---

## XVI. Nguyên tắc vàng cần nhớ

- File system = Router
- Layout quyết định số phận screen
- Slot cho phép render tiếp
- Không navigator = không screen
- Layout gần screen nhất có quyền cao nhất
- Router luôn chạy từ root → xuống dưới
- Grouping folders `()` không ảnh hưởng URL
- `index.tsx` = route gốc của folder
- Dùng `asChild` khi Link wrap custom component
- `push` vs `replace`: push có thể back, replace không thể

---

## XVII. Dynamic Routes

Dynamic routes cho phép tạo routes với params động (như `/user/123`, `/post/abc`).

### Cú pháp:
- `[param]` - Single dynamic segment
- `[...param]` - Catch-all segment (match multiple segments)

### Ví dụ 1: Single Dynamic Param

```
app/
  └── user/
      └── [id].tsx        → matches /user/123, /user/abc
```

```tsx
// app/user/[id].tsx
import { useLocalSearchParams } from 'expo-router';
import { View, Text } from 'react-native';

export default function UserScreen() {
  const { id } = useLocalSearchParams();
  
  return (
    <View>
      <Text>User ID: {id}</Text>
    </View>
  );
}
```

### Ví dụ 2: Navigate tới Dynamic Route

```tsx
import { Link, useRouter } from 'expo-router';

// Với Link
<Link href="/user/123">View User</Link>
<Link href={{ pathname: "/user/[id]", params: { id: "123" } }}>
  View User
</Link>

// Với router
const router = useRouter();
router.push('/user/123');
router.push({ pathname: '/user/[id]', params: { id: '123' } });
```

### Ví dụ 3: Multiple Params

```
app/
  └── shop/
      └── [category]/
          └── [productId].tsx    → /shop/electronics/phone-123
```

```tsx
// app/shop/[category]/[productId].tsx
import { useLocalSearchParams } from 'expo-router';

export default function ProductScreen() {
  const { category, productId } = useLocalSearchParams();
  
  return (
    <View>
      <Text>Category: {category}</Text>
      <Text>Product: {productId}</Text>
    </View>
  );
}

// Navigate: /shop/electronics/phone-123
<Link href="/shop/electronics/phone-123" />
```

### Ví dụ 4: Catch-all Routes `[...]`

```
app/
  └── blog/
      └── [...slug].tsx    → matches /blog/a, /blog/a/b, /blog/a/b/c
```

```tsx
// app/blog/[...slug].tsx
import { useLocalSearchParams } from 'expo-router';

export default function BlogPost() {
  const { slug } = useLocalSearchParams();
  
  // slug là array khi match nhiều segments
  // /blog/2024/january/post-1 → slug = ["2024", "january", "post-1"]
  
  const path = Array.isArray(slug) ? slug.join('/') : slug;
  
  return <Text>Blog Path: {path}</Text>;
}
```

### Ví dụ 5: Query Params

```tsx
// Navigate với query params
<Link href={{ pathname: "/search", params: { q: "expo", filter: "new" } }}>
  Search
</Link>
// URL: /search?q=expo&filter=new

// Đọc query params
function SearchScreen() {
  const { q, filter } = useLocalSearchParams();
  
  return (
    <View>
      <Text>Search: {q}</Text>
      <Text>Filter: {filter}</Text>
    </View>
  );
}
```

---

## XVIII. Modal Presentation

Modal là screen hiển thị overlay, có thể dismiss bằng cách swipe down.

### Ví dụ 1: Basic Modal

```
app/
  ├── _layout.tsx
  ├── index.tsx
  └── modal.tsx           → Modal screen
```

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen 
        name="modal" 
        options={{ 
          presentation: 'modal',        // ⭐ Quan trọng
          title: 'My Modal'
        }} 
      />
    </Stack>
  );
}
```

```tsx
// app/index.tsx
import { Link } from 'expo-router';

export default function Home() {
  return (
    <View>
      <Link href="/modal">Open Modal</Link>
    </View>
  );
}
```

```tsx
// app/modal.tsx
import { View, Text, Button } from 'react-native';
import { useRouter } from 'expo-router';

export default function Modal() {
  const router = useRouter();
  
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>This is a Modal</Text>
      
      {/* Dismiss modal */}
      <Button title="Close" onPress={() => router.back()} />
    </View>
  );
}
```

### Ví dụ 2: Full Screen Modal

```tsx
<Stack.Screen 
  name="fullscreen-modal" 
  options={{ 
    presentation: 'fullScreenModal',  // Full screen
    headerShown: false
  }} 
/>
```

### Ví dụ 3: Transparent Modal

```tsx
<Stack.Screen 
  name="transparent-modal" 
  options={{ 
    presentation: 'transparentModal',
    animation: 'fade',
    headerShown: false
  }} 
/>
```

---

## XIX. Shared Routes / Nesting Navigators

Shared routes là routes có thể access từ nhiều navigators.

### Ví dụ: Details screen từ nhiều tabs

```
app/
  ├── _layout.tsx           # Root Stack
  ├── details.tsx           # 🔥 Shared route (ngoài tabs)
  └── (tabs)/               # Tabs group
      ├── _layout.tsx       # Tabs layout
      ├── home.tsx          # Tab 1
      └── profile.tsx       # Tab 2
```

```tsx
// app/_layout.tsx (Root Stack)
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
      <Stack.Screen name="details" options={{ title: 'Details' }} />
    </Stack>
  );
}

// app/(tabs)/_layout.tsx (Tabs)
import { Tabs } from 'expo-router';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen name="home" options={{ title: 'Home' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile' }} />
    </Tabs>
  );
}
```

Bây giờ cả `home.tsx` và `profile.tsx` đều có thể navigate tới `/details`:

```tsx
// app/(tabs)/home.tsx
<Link href="/details">Go to Details</Link>

// app/(tabs)/profile.tsx
<Link href="/details">Go to Details</Link>
```

Khi navigate tới `/details`, nó sẽ **push lên Stack** (không phải tab), và khi back sẽ về tab trước đó.

---

## XX. Not Found (404) Screen

```tsx
// app/+not-found.tsx
import { Link, Stack } from 'expo-router';
import { View, Text, StyleSheet } from 'react-native';

export default function NotFoundScreen() {
  return (
    <>
      <Stack.Screen options={{ title: "Oops! Not Found" }} />
      <View style={styles.container}>
        <Text style={styles.title}>404</Text>
        <Text>This screen doesn't exist.</Text>
        <Link href="/" style={styles.link}>
          Go back to Home
        </Link>
      </View>
    </>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 20,
  },
  title: {
    fontSize: 64,
    fontWeight: 'bold',
  },
  link: {
    marginTop: 20,
    paddingVertical: 15,
    color: 'blue',
  },
});
```

---

## XXI. Head / SEO (Web Only)

Thay đổi `<title>`, meta tags cho web.

```tsx
// app/about.tsx
import { View, Text } from 'react-native';
import Head from 'expo-router/head';

export default function About() {
  return (
    <>
      <Head>
        <title>About Us - My App</title>
        <meta name="description" content="Learn more about our company" />
        <meta property="og:title" content="About Us" />
      </Head>
      
      <View>
        <Text>About Screen</Text>
      </View>
    </>
  );
}
```

---

## XXII. Deep Linking

Expo Router tự động support deep linking cho tất cả routes.

### Ví dụ:
```
File: app/user/[id].tsx
↓
Deep link: myapp://user/123
Web URL: https://myapp.com/user/123
```

### Config (app.json):

```json
{
  "expo": {
    "scheme": "myapp",
    "web": {
      "bundler": "metro"
    }
  }
}
```

Bây giờ có thể mở app bằng:
- `myapp://` (mobile)
- `https://myapp.com/` (web)

Tất cả routes tự động work với deep links!

---

## XXIII. Ví dụ App Hoàn Chỉnh

```
app/
  ├── _layout.tsx                    # Root Stack
  ├── index.tsx                      # Redirect based on auth
  ├── +not-found.tsx                 # 404 page
  ├── (auth)/                        # Auth group
  │   ├── _layout.tsx                # Auth Stack (no header)
  │   ├── login.tsx                  # /login
  │   └── register.tsx               # /register
  └── (app)/                         # Main app group
      ├── _layout.tsx                # Check auth + setup tabs
      ├── (tabs)/                    # Tabs group
      │   ├── _layout.tsx            # Tabs navigator
      │   ├── index.tsx              # / (Home tab)
      │   ├── search.tsx             # /search
      │   └── profile.tsx            # /profile
      ├── settings.tsx               # /settings (shared, outside tabs)
      ├── post/
      │   └── [id].tsx               # /post/123
      └── modal.tsx                  # /modal (modal presentation)
```

```tsx
// app/_layout.tsx (Root)
import { Slot } from 'expo-router';
import { AuthProvider } from '@/contexts/AuthContext';

export default function RootLayout() {
  return (
    <AuthProvider>
      <Slot />
    </AuthProvider>
  );
}

// app/index.tsx (Initial redirect)
import { Redirect } from 'expo-router';
import { useAuth } from '@/contexts/AuthContext';

export default function Index() {
  const { isAuthenticated } = useAuth();
  
  return isAuthenticated ? (
    <Redirect href="/(app)/(tabs)" />
  ) : (
    <Redirect href="/(auth)/login" />
  );
}

// app/(auth)/_layout.tsx
import { Stack } from 'expo-router';

export default function AuthLayout() {
  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="login" />
      <Stack.Screen name="register" />
    </Stack>
  );
}

// app/(app)/_layout.tsx
import { Redirect, Stack } from 'expo-router';
import { useAuth } from '@/contexts/AuthContext';

export default function AppLayout() {
  const { isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    return <Redirect href="/(auth)/login" />;
  }
  
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
      <Stack.Screen name="settings" options={{ title: 'Settings' }} />
      <Stack.Screen name="post/[id]" options={{ title: 'Post' }} />
      <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
    </Stack>
  );
}

// app/(app)/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen 
        name="index" 
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen 
        name="search" 
        options={{
          title: 'Search',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="search" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen 
        name="profile" 
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

---

## XXIV. Troubleshooting / Common Mistakes

### ❌ Mistake 1: Quên return Navigator trong Layout
```tsx
// ❌ SAI
export default function Layout() {
  // Không return gì → screens không render
}

// ✅ ĐÚNG
export default function Layout() {
  return <Stack />;
}
```

### ❌ Mistake 2: Quên asChild với custom component
```tsx
// ❌ SAI: Button không nhận press event
<Link href="/about">
  <Button title="Go" />
</Link>

// ✅ ĐÚNG
<Link href="/about" asChild>
  <Button title="Go" />
</Link>
```

### ❌ Mistake 3: Đặt components trong app/
```
app/
  ├── Button.tsx          ❌ Sẽ bị hiểu là route /Button
  └── index.tsx
```
→ Đặt components ngoài `app/`

### ❌ Mistake 4: Nhầm lẫn push vs replace
```tsx
// Login thành công
router.push('/home');     // ❌ User có thể back về login

router.replace('/home');  // ✅ User không thể back về login
```

### ❌ Mistake 5: Pass object qua params
```tsx
// ❌ SAI
router.push({ 
  pathname: '/details', 
  params: { user: { name: 'John' } }  // Object sẽ bị stringify
});

// ✅ ĐÚNG
router.push({ 
  pathname: '/details', 
  params: { userId: '123' }  // Pass ID, fetch data ở destination
});
```

---

## XXV. Best Practices

### ✅ 1. Tổ chức file rõ ràng
```
app/
  ├── (auth)/              # Group cho auth screens
  ├── (app)/               # Group cho main app
  └── (onboarding)/        # Group cho onboarding
```

### ✅ 2. Dùng TypeScript
```tsx
// Expo Router tự generate types
import { Link } from 'expo-router';

<Link href="/about" />  // ✅ TypeScript check route tồn tại
<Link href="/invalid" /> // ❌ TypeScript error
```

### ✅ 3. Shared components ngoài app/
```
src/
  ├── app/                # Routes only
  ├── components/         # Shared components
  ├── hooks/              # Custom hooks
  └── utils/              # Helpers
```

### ✅ 4. Use Stack.Screen để config từng screen
```tsx
<Stack>
  <Stack.Screen 
    name="details" 
    options={{ 
      title: 'Details',
      headerBackTitle: 'Back',
      headerStyle: { backgroundColor: 'blue' }
    }} 
  />
</Stack>
```

### ✅ 5. Loading states khi check auth
```tsx
export default function Layout() {
  const { isAuthenticated, isLoading } = useAuth();
  
  if (isLoading) {
    return <LoadingScreen />;  // ✅ Show loading
  }
  
  if (!isAuthenticated) {
    return <Redirect href="/login" />;
  }
  
  return <Slot />;
}
```

### ✅ 6. Error boundaries
```tsx
// app/_layout.tsx
import { ErrorBoundary } from 'expo-router';

export function ErrorBoundary({ error }) {
  return (
    <View>
      <Text>Error: {error.message}</Text>
    </View>
  );
}
```

---

**🎉 THE END - Đã đủ để bắt đầu với Expo Router!**
