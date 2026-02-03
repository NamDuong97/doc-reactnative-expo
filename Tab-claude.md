# BOTTOM TABS NAVIGATOR

## **1. TABS NAVIGATOR LÀ GÌ?**

Tabs Navigator là layout điều hướng phổ biến nhất trên mobile apps, hiển thị các tab ở dưới cùng màn hình (bottom tabs). Rất hiếm thấy trên web nhưng cực kỳ phổ biến trên mobile.

**Ví dụ thực tế:**
- Instagram: Home, Search, Reels, Shop, Profile
- Facebook: Home, Watch, Marketplace, Gaming, Menu
- Twitter: Home, Search, Notifications, Messages

---

## **2. TẠO BOTTOM TABS CƠ BẢN**

### **Chuyển từ Stack sang Tabs**

**Trước (Stack Navigator):**
```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return <Stack />;
}
```

**Sau (Tabs Navigator):**
```tsx
// app/_layout.tsx
import { Tabs } from 'expo-router';

export default function Layout() {
  return <Tabs />;
}
```

**Cấu trúc file:**
```
app/
  _layout.tsx      ← Tabs navigator
  index.tsx        ← Tab 1
  second.tsx       ← Tab 2
  third.tsx        ← Tab 3
  fourth.tsx       ← Tab 4
```

**Kết quả:** Tất cả 4 screens tự động hiển thị như bottom tabs (không cần khai báo).

---

## **3. CẤU HÌNH TABS**

### **A) Thứ tự và Screen Options**

```tsx
import { Tabs } from 'expo-router';

export default function Layout() {
  return (
    <Tabs>
      <Tabs.Screen name="index" />
      <Tabs.Screen name="second" />
      <Tabs.Screen name="third" />
      <Tabs.Screen name="fourth" />
    </Tabs>
  );
}
```

**Lợi ích của việc khai báo:**
- Kiểm soát thứ tự tabs
- Truyền screen options
- Cấu hình từng tab riêng biệt

### **B) Thêm Icons**

**Bước 1: Cài đặt**
```bash
npx expo install @expo/vector-icons
```

**Bước 2: Import và sử dụng**
```tsx
import { Tabs } from 'expo-router';
import { MaterialIcons } from '@expo/vector-icons';

export default function Layout() {
  return (
    <Tabs>
      <Tabs.Screen 
        name="index"
        options={{
          tabBarIcon: ({ color, size, focused }) => (
            <MaterialIcons 
              name="home" 
              size={size} 
              color={color} 
            />
          )
        }}
      />
      
      <Tabs.Screen 
        name="second"
        options={{
          tabBarIcon: ({ color, size }) => (
            <MaterialIcons 
              name="search" 
              size={size} 
              color={color} 
            />
          )
        }}
      />
    </Tabs>
  );
}
```

**Props của tabBarIcon:**
- `color`: Màu tự động (active/inactive)
- `size`: Kích thước icon
- `focused`: Boolean cho biết tab có đang active không

### **C) Cấu hình Màu Sắc**

**Cấu hình toàn bộ Tabs:**
```tsx
<Tabs
  screenOptions={{
    tabBarActiveTintColor: 'teal',      // Màu khi active
    tabBarInactiveTintColor: 'gray',    // Màu khi inactive
  }}
>
  {/* screens */}
</Tabs>
```

**Cấu hình từng Tab riêng:**
```tsx
<Tabs.Screen 
  name="index"
  options={{
    tabBarActiveTintColor: 'red',  // Override màu riêng
  }}
/>
```

### **D) Cấu hình Labels**

```tsx
<Tabs.Screen 
  name="index"
  options={{
    title: 'Home',              // Dùng cho cả header và tab label
    tabBarLabel: 'Home Tab',    // Override chỉ tab label
    headerTitle: 'Home Screen', // Override chỉ header title
  }}
/>

// Ẩn label hoàn toàn
<Tabs.Screen 
  name="second"
  options={{
    tabBarShowLabel: false,
  }}
/>
```

### **E) Ẩn Tab khỏi Tab Bar**

```tsx
<Tabs.Screen 
  name="secret"
  options={{
    href: null,  // Tab không hiển thị trong tab bar
  }}
/>
```

**Use case:** Màn hình detail hoặc settings nằm trong tab nhưng không muốn hiện icon dưới tab bar.

### **F) Badges (Thông báo)**

```tsx
<Tabs.Screen 
  name="notifications"
  options={{
    tabBarBadge: 3,  // Hiển thị số 3
    tabBarBadgeStyle: {
      backgroundColor: '#FF6B6B',  // Màu badge nhẹ hơn
    }
  }}
/>
```

**Ví dụ thực tế:**
```tsx
// Hiển thị số tin nhắn chưa đọc
const [unreadCount, setUnreadCount] = useState(5);

<Tabs.Screen 
  name="messages"
  options={{
    tabBarBadge: unreadCount > 0 ? unreadCount : undefined,
  }}
/>
```

---

## **4. NESTED NAVIGATORS TRONG TABS**

### **Vấn đề:** 
Mobile apps thường có stack navigator BÊN TRONG mỗi tab (ví dụ: Instagram Home tab có thể vào profile, vào post detail, etc.)

### **A) Tạo Stack Trong Tab**

**Cấu trúc file:**
```
app/
  _layout.tsx              ← Tabs navigator
  index.tsx                ← Tab 1 (single screen)
  second/                  ← Tab 2 (stack)
    _layout.tsx            ← Stack navigator cho tab này
    index.tsx              ← Screen đầu tiên trong stack
    nested.tsx             ← Screen thứ 2
    also-nested.tsx        ← Screen thứ 3
  third.tsx                ← Tab 3 (single screen)
```

**Code trong second/_layout.tsx:**
```tsx
import { Stack } from 'expo-router';

export default function SecondLayout() {
  return <Stack />;
}
```

**Code trong second/index.tsx:**
```tsx
import { Link } from 'expo-router';

export default function SecondHome() {
  return (
    <View>
      <Text>Second Tab Home</Text>
      <Link href="/second/nested" push>
        Go to Nested
      </Link>
    </View>
  );
}
```

**Kết quả:** Tab "second" giờ có thể navigate qua lại giữa các screens như một stack.

### **B) Vấn đề: Double Headers**

Khi có nested navigator, sẽ xuất hiện **2 headers**:
- Header từ Tabs navigator (ngoài)
- Header từ Stack navigator (trong)

**Ví dụ trực quan:**
```
┌─────────────────────┐
│  Second Tab         │ ← Header từ Tabs
├─────────────────────┤
│  Nested Screen      │ ← Header từ Stack
├─────────────────────┤
│                     │
│  Content here       │
│                     │
└─────────────────────┘
```

**Giải pháp: Ẩn header ngoài**
```tsx
// app/_layout.tsx (root layout)
<Tabs>
  <Tabs.Screen 
    name="second"
    options={{
      headerShown: false,  // Ẩn header của Tabs
    }}
  />
</Tabs>
```

**Sau khi ẩn:**
```
┌─────────────────────┐
│  Nested Screen      │ ← Chỉ còn header từ Stack
├─────────────────────┤
│                     │
│  Content here       │
│                     │
└─────────────────────┘
```

### **C) Cấu hình Stack Nested**

```tsx
// app/second/_layout.tsx
import { Stack } from 'expo-router';

export default function SecondLayout() {
  return (
    <Stack>
      <Stack.Screen 
        name="index"
        options={{ title: 'Second Home' }}
      />
      <Stack.Screen 
        name="nested"
        options={{ title: 'Nested Screen' }}
      />
      <Stack.Screen 
        name="also-nested"
        options={{ title: 'Deep Nested' }}
      />
    </Stack>
  );
}
```

**Lưu ý:** `name` ở đây là **relative** so với layout file này (không bao gồm "second/").

---

## **5. INDEX ROUTE VÀ GROUPING FOLDERS**

### **Vấn đề đặc biệt với Index Route**

**Index route là route đặc biệt:**
- Luôn render đầu tiên khi app khởi động
- Phải tồn tại ở root level (hoặc trong grouping folder)

**Ví dụ vấn đề:**
```
app/
  _layout.tsx
  home/              ← Folder bình thường
    index.tsx        ← KHÔNG hoạt động!
```

**Lỗi:** App sẽ hiện "Not Found" khi khởi động vì không tìm thấy route `/`.

### **Giải pháp: Grouping Folder**

```
app/
  _layout.tsx
  (home)/            ← Grouping folder (có dấu ngoặc)
    index.tsx        ← Hoạt động bình thường!
    nested.tsx
```

**Routes:**
```
/           → (home)/index.tsx
/nested     → (home)/nested.tsx
```

Tên `(home)` bị bỏ qua trong route!

### **Tạo Stack trong Home Tab với Grouping Folder**

**Cấu trúc:**
```
app/
  _layout.tsx              ← Tabs
  (home)/                  ← Grouping folder
    _layout.tsx            ← Stack
    index.tsx              ← Home screen
    nested.tsx             ← Nested screen
  second/
    _layout.tsx
    index.tsx
```

**app/_layout.tsx:**
```tsx
<Tabs>
  <Tabs.Screen 
    name="(home)"          // Target grouping folder
    options={{
      headerShown: false,
      title: 'Home',
      tabBarIcon: ({ color, size }) => (
        <MaterialIcons name="home" size={size} color={color} />
      )
    }}
  />
  <Tabs.Screen name="second" options={{ headerShown: false }} />
</Tabs>
```

**app/(home)/_layout.tsx:**
```tsx
import { Stack } from 'expo-router';

export default function HomeLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="nested" options={{ title: 'Details' }} />
    </Stack>
  );
}
```

---

## **6. NAVIGATION BEHAVIOR VÀ HISTORY**

### **A) Tab Bar Click Behavior (Mặc định)**

**Hành vi mặc định:**
```
1. User ở tab Second, navigate: Index → Nested → Deep Nested
2. User click icon tab Second lần 1 → Reset về Index
3. User navigate lại: Index → Nested → Deep Nested
4. User switch sang tab Third rồi quay lại Second → Vẫn ở Deep Nested
5. User click icon tab Second lần 2 → Reset về Index
```

**Giải thích:**
- Click vào tab đang active → Reset về screen đầu tiên
- Switch tab rồi quay lại → Giữ nguyên state (không reset)

### **B) Reset State Khi Switch Tab**

**Thêm option `popToTopOnBlur`:**
```tsx
// app/_layout.tsx
<Tabs>
  <Tabs.Screen 
    name="second"
    options={{
      headerShown: false,
      // Reset về top khi rời khỏi tab này
      unmountOnBlur: false,  // Giữ component mounted
    }}
    listeners={({ navigation }) => ({
      blur: () => navigation.popToTop(),  // Reset khi blur
    })}
  />
</Tabs>
```

**Hoặc trong Stack layout:**
```tsx
// app/second/_layout.tsx
import { Stack } from 'expo-router';

export default function SecondLayout() {
  return (
    <Stack
      screenOptions={{
        // Có thể cấu hình ở đây
      }}
    />
  );
}
```

**Với `popToTopOnBlur: true`:**
```
1. Navigate: Index → Nested → Deep Nested
2. Switch sang tab Third
3. Quay lại tab Second → Reset về Index (có animation)
```

### **C) Tắt Animation Khi Reset**

**Vấn đề:** Animation khi reset có thể gây khó chịu.

**Giải pháp: Conditional animation:**
```tsx
// app/second/_layout.tsx
import { Stack } from 'expo-router';
import { usePathname } from 'expo-router';

export default function SecondLayout() {
  const pathname = usePathname();
  
  return (
    <Stack
      screenOptions={{
        animation: pathname.startsWith('/second') 
          ? 'default'  // Animation bình thường khi trong tab
          : 'none'     // Không animation khi switch tab
      }}
    />
  );
}
```

**Cách hoạt động:**
- Khi navigate trong tab (`/second` → `/second/nested`): Animation bình thường
- Khi switch từ tab khác về: Không animation

---

## **7. BACK BUTTON BEHAVIOR**

### **A) Back Button Cơ Bản**

```tsx
import { router } from 'expo-router';

export default function Screen() {
  return (
    <Button 
      title="Back" 
      onPress={() => router.back()}
    />
  );
}
```

**Lỗi trên Index Screen:**
```
Warning: The action 'GO_BACK' was not handled
```
→ Không có screen nào để back về.

### **B) Check canGoBack()**

```tsx
import { router } from 'expo-router';

export default function Screen() {
  const canGoBack = router.canGoBack();
  
  return canGoBack ? (
    <Button title="Back" onPress={() => router.back()} />
  ) : null;
}
```

### **C) Back Behavior trong Tabs**

**Mặc định: Reset về Tab đầu tiên**
```
Navigate: Index → Second → Third → Fourth
Press Back → Quay về Index (tab đầu tiên)
```

**Nếu reorder tabs:**
```tsx
<Tabs>
  <Tabs.Screen name="second" />  {/* Tab đầu tiên */}
  <Tabs.Screen name="index" />
  <Tabs.Screen name="third" />
  <Tabs.Screen name="fourth" />
</Tabs>

// Navigate: Index → Second → Third → Fourth
// Press Back → Quay về Second (tab đầu tiên bây giờ)
```

### **D) Change Back Behavior sang "order"**

**Để back theo thứ tự đã navigate:**
```tsx
<Tabs
  screenOptions={{
    // Không có backBehavior prop trực tiếp trong Expo Router
    // Cần dùng React Navigation config
  }}
>
```

**Workaround với manual back:**
```tsx
import { router, useNavigationState } from 'expo-router';

const history = useNavigationState(state => state.history);

const goBackInOrder = () => {
  if (history.length > 1) {
    router.back();
  }
};
```

**Behavior với "order":**
```
Navigate: Index → Second → Third → Fourth
Press Back → Third
Press Back → Second
Press Back → Index
```

---

## **8. TREE STRUCTURE CỦA NAVIGATION**

**Khác với Stack (linear), Tabs tạo tree structure:**

```
Root
├── Tab 1 (Home)
│   ├── Screen 1
│   ├── Screen 2
│   └── Screen 3
├── Tab 2 (Search)
│   └── Screen 1
├── Tab 3 (Profile)
│   ├── Screen 1
│   └── Screen 2
└── Tab 4 (Settings)
    └── Screen 1
```

**Ý nghĩa:**
- Mỗi tab có state riêng
- Navigate trong tab không ảnh hưởng tab khác
- Switch tab giữ state (trừ khi dùng `unmountOnBlur`)

---

## **9. TIPS & BEST PRACTICES**

### **A) Debug Navigation**

```bash
# Xem sitemap
npx expo-router sitemap

# Check routes
npx expo-router --check
```

### **B) TypeScript với Tabs**

```tsx
import { Tabs } from 'expo-router';
import { ComponentProps } from 'react';

type TabsScreenProps = ComponentProps<typeof Tabs.Screen>;

const tabConfig: TabsScreenProps = {
  name: 'home',
  options: {
    title: 'Home',
    // TypeScript sẽ autocomplete tất cả options
  }
};
```

### **C) Dynamic Tab Badge**

```tsx
import { useState, useEffect } from 'react';
import { Tabs } from 'expo-router';

export default function Layout() {
  const [notificationCount, setNotificationCount] = useState(0);
  
  useEffect(() => {
    // Fetch notification count
    fetchNotifications().then(count => setNotificationCount(count));
  }, []);
  
  return (
    <Tabs>
      <Tabs.Screen 
        name="notifications"
        options={{
          tabBarBadge: notificationCount > 0 ? notificationCount : undefined,
        }}
      />
    </Tabs>
  );
}
```

### **D) Conditional Tab Visibility**

```tsx
const { user } = useAuth();

<Tabs>
  <Tabs.Screen name="home" />
  <Tabs.Screen 
    name="admin"
    options={{
      href: user?.isAdmin ? '/admin' : null,  // Ẩn nếu không phải admin
    }}
  />
</Tabs>
```

### **E) Custom Tab Bar**

```tsx
<Tabs
  tabBar={(props) => <CustomTabBar {...props} />}
>
  {/* screens */}
</Tabs>
```

---

## **10. TÓM TẮT QUAN TRỌNG**

### **Tabs Navigator**
- Thay `<Stack />` bằng `<Tabs />` trong layout
- Tất cả screens tự động thành tabs
- Khai báo để control thứ tự và options

### **Icons & Styling**
- Dùng `tabBarIcon` với props `color`, `size`, `focused`
- Cấu hình màu với `tabBarActiveTintColor`
- Ẩn tab với `href: null`
- Thêm badge với `tabBarBadge`

### **Nested Navigators**
- Tạo folder → thêm `_layout.tsx` với `<Stack />`
- Ẩn outer header với `headerShown: false`
- Screen names là relative trong nested layout

### **Index Route**
- Cần grouping folder `(name)` để nest index route
- Index route luôn render đầu tiên
- Grouping folder name không ảnh hưởng route

### **Navigation Behavior**
- Click tab đang active → Reset
- Switch tab → Giữ state (mặc định)
- `popToTopOnBlur` để reset khi switch
- Back button mặc định về tab đầu tiên

### **Tree Structure**
- Mỗi tab = branch riêng
- State của tab được preserve
- Navigate trong tab không ảnh hưởng tab khác

---

## **CÔNG CỤ HỮU ÍCH**

### **Debug Commands**
```bash
# Xem tất cả routes
npx expo-router sitemap

# Clear cache và restart
npx expo start -c
```

### **TypeScript Tips**
```tsx
// Type cho params
type RouteParams = {
  id: string;
  category?: string;
};

const params = useLocalSearchParams<RouteParams>();

// Type cho href
import { Href } from 'expo-router';

const myHref: Href = '/profile/123';
```

---

## **BEST PRACTICES**

1. **Luôn dùng TypeScript cho params** để tránh lỗi typo
2. **Đặt screen options trong component** khi cần access data
3. **Dùng replace thay vì push** cho auth flows (để user không back về login)
4. **Kiểm tra not found case** khi dùng dynamic routes
5. **Dùng grouping folders** để tổ chức code rõ ràng
6. **Hard refresh** khi thêm file mới và gặp lỗi not found
7. **Serialize data** trước khi truyền qua params (JSON.stringify nếu cần)
8. **Convert string params** sang number/boolean nếu cần (vì params luôn là string)