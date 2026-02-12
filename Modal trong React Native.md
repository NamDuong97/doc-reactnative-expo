# Hướng dẫn Modal trong React Native & Expo Router

## Tổng quan

**Modal** là một pattern điều hướng phổ biến, cho phép hiển thị màn hình đè lên nội dung ứng dụng hiện tại. Video này trình bày 3 cách khác nhau để hiển thị nội dung overlay:

1. **Alert** - Hộp thoại xác nhận nhanh
2. **React Native Modal** - Component modal tùy chỉnh hoàn toàn
3. **Expo Router Modal** - Hiển thị screen dạng modal với routing

---

## 1. Alert Component - Xác nhận nhanh

### Khi nào sử dụng?
- Cần xác nhận nhanh từ người dùng
- Không cần custom style phức tạp
- Ví dụ: "Bạn có chắc muốn xóa?", "Lưu thay đổi?"

### Ví dụ code:

```javascript
import { Alert, Pressable, Text } from 'react-native';

export default function HomePage() {
  const handleDelete = () => {
    Alert.alert(
      'Xác nhận xóa',           // Tiêu đề
      'Bạn có chắc muốn xóa?', // Nội dung
      [
        {
          text: 'Hủy',
          style: 'cancel'
        },
        {
          text: 'Xóa',
          style: 'destructive',  // Màu đỏ cảnh báo
          onPress: () => {
            console.log('Đã xóa!');
          }
        }
      ]
    );
  };

  return (
    <Pressable onPress={handleDelete}>
      <Text>Xóa tài khoản</Text>
    </Pressable>
  );
}
```

### Giao diện:

**iOS:**
```
┌─────────────────────┐
│   Xác nhận xóa      │
│                     │
│ Bạn có chắc muốn    │
│ xóa?                │
│                     │
│  [Hủy]   [Xóa]     │ <- Xóa màu đỏ
└─────────────────────┘
```

**Android:**
```
┌─────────────────────┐
│ Xác nhận xóa        │
│ Bạn có chắc muốn    │
│ xóa?                │
│        [HỦY]  [XÓA] │
└─────────────────────┘
```

**Ưu điểm:**
- Native UI, phù hợp với platform
- Dễ sử dụng, không cần state management
- Tự động xử lý accessibility

**Nhược điểm:**
- Không custom được style
- Giới hạn về layout

---

## 2. React Native Modal - Tùy chỉnh hoàn toàn

### Khi nào sử dụng?
- Cần custom style phức tạp
- Có form, input, hoặc UI phức tạp
- Không cần routing/navigation

### Ví dụ code:

```javascript
import { useState } from 'react';
import { Modal, View, Text, Pressable, StyleSheet } from 'react-native';

export default function CustomModalExample() {
  const [isVisible, setIsVisible] = useState(false);

  return (
    <View style={styles.container}>
      {/* Button mở modal */}
      <Pressable onPress={() => setIsVisible(true)}>
        <Text>Mở Modal</Text>
      </Pressable>

      {/* Modal component */}
      <Modal
        visible={isVisible}
        onRequestClose={() => setIsVisible(false)} // Android back button
        animationType="slide"    // 'none' | 'slide' | 'fade'
        transparent={true}        // Giữ background phía sau
        presentationStyle="pageSheet" // iOS only
      >
        {/* Outer view - Center content */}
        <View style={styles.modalOverlay}>
          {/* Inner view - Modal content */}
          <View style={styles.modalContent}>
            <Text style={styles.title}>Custom Modal</Text>
            <Text>Nội dung tùy chỉnh ở đây</Text>
            
            <Pressable 
              style={styles.closeButton}
              onPress={() => setIsVisible(false)}
            >
              <Text>Đóng</Text>
            </Pressable>
          </View>
        </View>
      </Modal>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  modalOverlay: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.5)', // Semi-transparent background
  },
  modalContent: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    width: '80%',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  closeButton: {
    marginTop: 15,
    padding: 10,
    backgroundColor: '#007AFF',
    borderRadius: 8,
    alignItems: 'center',
  },
});
```

### Animation Types:

**1. `animationType="fade"`** (Mặc định)
```
Opacity: 0 → 1
□□□□□□□□
░░░░░░░░  (Hiện dần)
████████
```

**2. `animationType="slide"`**
```
Bottom → Center
        ┌─────┐
        │     │
      ↑ │     │
        │     │
        └─────┘
```

**3. `presentationStyle="pageSheet"` (iOS)**
```
┌──────────────┐
│ ════════════ │ <- Background vẫn thấy
│ ┌──────────┐ │
│ │  Modal   │ │ <- Kéo xuống để đóng
│ │          │ │
│ │          │ │
```

### So sánh Presentation Styles:

| Style | Mô tả | Khi nào dùng |
|-------|-------|--------------|
| `fullScreen` | Toàn màn hình | Form dài, cần nhiều không gian |
| `pageSheet` | Card từ dưới lên (iOS) | Sheet action, filter |
| `formSheet` | Card nhỏ hơn (iPad) | Form ngắn |
| `overFullScreen` | Toàn màn, giữ background | Custom overlay |

---

## 3. Expo Router Modal - Screen Navigation

### Khi nào sử dụng?
- Modal cần là một route trong app
- Cần deep linking vào modal
- Modal có nested navigation
- Cần share URL đến modal

### Cấu trúc thư mục:

#### **Bước 1: Cấu trúc ban đầu (Trước khi thêm modal)**

```
app/
├── _layout.tsx          (Root Stack)
├── (tabs)/              (Bottom Tabs Group)
│   ├── _layout.tsx      (Tab Navigator)
│   ├── index.tsx        (Home tab)
│   ├── profile.tsx      (Profile tab)
│   ├── (feed)/          (Feed Stack)
│   │   ├── _layout.tsx
│   │   └── index.tsx
│   └── (settings)/      (Settings Stack)
│       ├── _layout.tsx
│       └── index.tsx
```

**Vấn đề:** Modal không thể nằm cùng level với tabs!

#### **Bước 2: Restructure với Modal**

```
app/
├── _layout.tsx              ← ROOT STACK (chứa tabs + modals)
├── (tabs)/                  ← TAB GROUP
│   ├── _layout.tsx          ← Tab Navigator
│   ├── index.tsx
│   ├── profile.tsx
│   ├── (feed)/
│   │   ├── _layout.tsx
│   │   └── index.tsx
│   └── (settings)/
│       ├── _layout.tsx
│       └── index.tsx
│
├── modal.tsx                ← MODAL SCREEN (cùng level với tabs)
│
└── (modal-with-stack)/      ← MODAL với NESTED STACK
    ├── _layout.tsx
    ├── index.tsx
    └── nested.tsx
```

### Code Implementation:

#### **1. Root Layout (`app/_layout.tsx`)**

```typescript
import { Stack } from 'expo-router';

export const unstable_settings = {
  // Đảm bảo tabs luôn render trước, kể cả khi deep link vào modal
  initialRouteName: '(tabs)',
};

export default function RootLayout() {
  return (
    <Stack>
      {/* Main tabs - không show header */}
      <Stack.Screen 
        name="(tabs)" 
        options={{ headerShown: false }} 
      />
      
      {/* Simple Modal */}
      <Stack.Screen 
        name="modal" 
        options={{ 
          presentation: 'modal',  // Hiệu ứng modal
          title: 'My Modal'
        }} 
      />
      
      {/* Modal with nested stack */}
      <Stack.Screen 
        name="(modal-with-stack)" 
        options={{ 
          presentation: 'modal',
          headerShown: false  // Ẩn header của root, dùng header của nested stack
        }} 
      />
    </Stack>
  );
}
```

#### **2. Tabs Layout (`app/(tabs)/_layout.tsx`)**

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
        name="profile" 
        options={{ title: 'Profile' }} 
      />
      <Tabs.Screen 
        name="(feed)" 
        options={{ title: 'Feed' }} 
      />
      <Tabs.Screen 
        name="(settings)" 
        options={{ title: 'Settings' }} 
      />
    </Tabs>
  );
}
```

#### **3. Home Screen - Mở Modal (`app/(tabs)/index.tsx`)**

```typescript
import { Link } from 'expo-router';
import { View, Text } from 'react-native';

export default function HomePage() {
  return (
    <View>
      <Text>Home Screen</Text>
      
      {/* Link đến modal đơn giản */}
      <Link href="/modal">
        <Text>Open Simple Modal</Text>
      </Link>
      
      {/* Link đến modal có stack */}
      <Link href="/(modal-with-stack)">
        <Text>Open Modal with Stack</Text>
      </Link>
    </View>
  );
}
```

#### **4. Modal Screen (`app/modal.tsx`)**

```typescript
import { View, Text } from 'react-native';
import { router } from 'expo-router';

export default function ModalScreen() {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>This is a Modal!</Text>
      <Text onPress={() => router.back()}>Close</Text>
    </View>
  );
}
```

#### **5. Modal with Stack Layout (`app/(modal-with-stack)/_layout.tsx`)**

```typescript
import { Stack } from 'expo-router';

export default function ModalStackLayout() {
  return (
    <Stack>
      <Stack.Screen 
        name="index" 
        options={{ title: 'Modal Home' }} 
      />
      <Stack.Screen 
        name="nested" 
        options={{ title: 'Nested Screen' }} 
      />
    </Stack>
  );
}
```

### Hiệu ứng Modal trên các Platform:

**iOS:**
```
┌─────────────────┐
│ ← Back   Title  │  ← Header với swipe gesture
├─────────────────┤
│                 │
│   Modal Content │  ← Slide từ dưới lên
│                 │
│                 │  ← Kéo xuống để dismiss
└─────────────────┘
     ↑ Tabs ở dưới vẫn thấy
```

**Android:**
```
┌─────────────────┐
│ [X]      Title  │  ← Header
├─────────────────┤
│                 │
│   Modal Content │  ← Fade in
│                 │
│                 │  ← Back button dismiss
└─────────────────┘
```

---

## Deep Linking vào Modal

### Vấn đề:

Khi deep link trực tiếp vào modal, tabs không được render:

```
Deep link: myapp://modal

Stack render:
┌──────────┐
│  Modal   │  ← Modal được render
└──────────┘
   (Tabs KHÔNG được render - BUG!)
```

### Giải pháp: `unstable_settings`

```typescript
// app/_layout.tsx
export const unstable_settings = {
  initialRouteName: '(tabs)',
};
```

**Cách hoạt động:**

```
Deep link: myapp://modal

1. Router check: "Modal là route đầu tiên?"
2. unstable_settings: "initialRouteName = (tabs)"
3. Stack render:
   ┌──────────┐
   │  Tabs    │  ← Render TRƯỚC (initial)
   ├──────────┤
   │  Modal   │  ← Render SAU (đè lên tabs)
   └──────────┘
```

**Tại sao tên là "unstable"?**
- Không hoạt động với async routes (routes động)
- Tên sẽ đổi thành `anchor` trong phiên bản tương lai
- **An toàn sử dụng** trong production nếu không dùng async routes

---

## So sánh 3 phương pháp

| Tiêu chí | Alert | React Native Modal | Expo Router Modal |
|----------|-------|-------------------|-------------------|
| **Độ phức tạp** | Đơn giản nhất | Trung bình | Phức tạp nhất |
| **Custom UI** | ❌ Không | ✅ Hoàn toàn | ✅ Hoàn toàn |
| **Navigation** | ❌ Không | ❌ Không | ✅ Có |
| **Deep linking** | ❌ Không | ❌ Không | ✅ Có |
| **State management** | ❌ Không cần | ✅ Cần useState | ✅ Tự động (routing) |
| **Nested screens** | ❌ Không | ❌ Không | ✅ Có |
| **Platform native** | ✅ Tự động | ⚠️ Tùy config | ✅ Tự động |
| **Use case** | Xác nhận nhanh | Component overlay | Screen/Feature modal |

---

## Workflow chọn phương pháp

```
Cần hiển thị overlay?
│
├─ Chỉ cần xác nhận đơn giản?
│  └─ ✅ Dùng Alert
│
├─ Cần custom UI nhưng KHÔNG cần routing?
│  └─ ✅ Dùng React Native Modal
│
└─ Cần routing/navigation/deep linking?
   └─ ✅ Dùng Expo Router Modal
```

---

## Best Practices

### 1. Alert

```typescript
// ✅ GOOD: Rõ ràng về hành động
Alert.alert(
  'Xóa ảnh',
  'Bạn có chắc muốn xóa ảnh này? Hành động này không thể hoàn tác.',
  [
    { text: 'Hủy', style: 'cancel' },
    { text: 'Xóa', style: 'destructive', onPress: handleDelete }
  ]
);

// ❌ BAD: Không rõ ràng
Alert.alert('Thông báo', 'OK?');
```

### 2. React Native Modal

```typescript
// ✅ GOOD: Cleanup listener
useEffect(() => {
  const backHandler = BackHandler.addEventListener(
    'hardwareBackPress',
    () => {
      if (isModalVisible) {
        setModalVisible(false);
        return true; // Prevent default behavior
      }
      return false;
    }
  );

  return () => backHandler.remove();
}, [isModalVisible]);

// ✅ GOOD: Accessible
<Modal
  accessible={true}
  accessibilityLabel="Settings modal"
  onRequestClose={closeModal} // Quan trọng cho Android!
>
```

### 3. Expo Router Modal

```typescript
// ✅ GOOD: Explicit presentation
<Stack.Screen 
  name="modal" 
  options={{ 
    presentation: 'modal',
    gestureEnabled: true,  // Cho phép swipe dismiss
    fullScreenGestureEnabled: true
  }} 
/>

// ✅ GOOD: Safe navigation
const router = useRouter();
const closeModal = () => {
  if (router.canGoBack()) {
    router.back();
  } else {
    router.replace('/'); // Fallback
  }
};
```

---

## Ví dụ thực tế

### Ví dụ 1: Confirmation Dialog (Alert)

```typescript
// Xóa sản phẩm khỏi giỏ hàng
const confirmRemoveItem = (itemId: string) => {
  Alert.alert(
    'Xóa sản phẩm',
    'Bạn có muốn xóa sản phẩm này khỏi giỏ hàng?',
    [
      { text: 'Không', style: 'cancel' },
      { 
        text: 'Xóa', 
        style: 'destructive',
        onPress: () => removeFromCart(itemId)
      }
    ]
  );
};
```

### Ví dụ 2: Filter Sheet (React Native Modal)

```typescript
function FilterModal({ visible, onClose, onApply }) {
  const [priceRange, setPriceRange] = useState([0, 1000]);
  const [category, setCategory] = useState('all');

  return (
    <Modal
      visible={visible}
      animationType="slide"
      transparent
      presentationStyle="pageSheet"
    >
      <View style={styles.modalContainer}>
        <View style={styles.content}>
          <Text style={styles.title}>Bộ lọc</Text>
          
          {/* Price range slider */}
          <PriceRangeSlider 
            value={priceRange} 
            onChange={setPriceRange} 
          />
          
          {/* Category picker */}
          <CategoryPicker 
            value={category} 
            onChange={setCategory} 
          />
          
          <View style={styles.actions}>
            <Button title="Hủy" onPress={onClose} />
            <Button 
              title="Áp dụng" 
              onPress={() => {
                onApply({ priceRange, category });
                onClose();
              }} 
            />
          </View>
        </View>
      </View>
    </Modal>
  );
}
```

### Ví dụ 3: Checkout Flow (Expo Router Modal)

```
app/
├── _layout.tsx
├── (tabs)/
│   └── cart.tsx          (Giỏ hàng)
│
└── (checkout)/           ← Modal stack
    ├── _layout.tsx
    ├── shipping.tsx      (Bước 1: Địa chỉ)
    ├── payment.tsx       (Bước 2: Thanh toán)
    └── confirmation.tsx  (Bước 3: Xác nhận)
```

```typescript
// app/(tabs)/cart.tsx
import { Link } from 'expo-router';

export default function CartScreen() {
  return (
    <View>
      <CartItems />
      <Link href="/(checkout)/shipping" asChild>
        <Pressable>
          <Text>Thanh toán</Text>
        </Pressable>
      </Link>
    </View>
  );
}

// app/(checkout)/_layout.tsx
import { Stack } from 'expo-router';

export default function CheckoutLayout() {
  return (
    <Stack>
      <Stack.Screen name="shipping" options={{ title: 'Địa chỉ giao hàng' }} />
      <Stack.Screen name="payment" options={{ title: 'Thanh toán' }} />
      <Stack.Screen name="confirmation" options={{ title: 'Xác nhận' }} />
    </Stack>
  );
}

// app/_layout.tsx
<Stack.Screen 
  name="(checkout)" 
  options={{ 
    presentation: 'modal',
    headerShown: false 
  }} 
/>
```

---

## Tổng kết

### Checklist chọn phương pháp:

**Dùng Alert nếu:**
- [ ] Chỉ cần Yes/No, OK/Cancel
- [ ] Không cần custom style
- [ ] Nội dung ngắn gọn (1-2 dòng)

**Dùng React Native Modal nếu:**
- [ ] Cần custom UI phức tạp
- [ ] Có form, input fields
- [ ] Không cần navigation/routing
- [ ] Modal đơn giản, không có sub-screens

**Dùng Expo Router Modal nếu:**
- [ ] Cần deep linking
- [ ] Có nhiều bước (multi-step flow)
- [ ] Cần share URL
- [ ] Modal là một phần của navigation flow

### Lưu ý quan trọng:

1. **Alert**: Platform-specific, không thể custom
2. **Modal**: Nhớ handle `onRequestClose` cho Android back button
3. **Router Modal**: PHẢI set `initialRouteName` để deep linking hoạt động đúng
4. **Performance**: Modal không unmount background screen → Tốt cho UX nhưng tốn memory

---

## Tài liệu tham khảo

- [React Native Modal](https://reactnative.dev/docs/modal)
- [Expo Router Modals](https://docs.expo.dev/router/advanced/modals/)
- [React Native Alert](https://reactnative.dev/docs/alert)