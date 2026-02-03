
# Stack Navigator trong Expo Router

## **1. STACK NAVIGATOR LÀ GÌ?**

Stack Navigator là một trong những Navigator chính để di chuyển giữa các màn hình trên mobile. Hoạt động theo nguyên lý **LIFO** (Last In First Out) - giống như chồng đĩa.

**Ví dụ minh họa:**
```
Stack: [Index]
Push Second → [Index, Second]
Push Third → [Index, Second, Third]
Go Back → [Index, Second]
Go Back → [Index]
```

---

## **2. CÁC PHƯƠNG THỨC ĐIỀU HƯỚNG**

### **A) PUSH**
Luôn đẩy màn hình mới lên đỉnh stack.

**Ví dụ:**
```tsx
// Từ Index
<Link href="/second" push>Go to Second</Link>

// Kết quả:
Index → push Second → [Index, Second]
Second → push Third → [Index, Second, Third]
Third → push Index → [Index, Second, Third, Index]
Second → push Third → [Index, Second, Third, Index, Second, Third]
```

**Đặc điểm:** Có thể có nhiều instances cùng màn hình trên stack.

### **B) DISMISS (dismissTo)**
Loại bỏ tất cả màn hình cho đến khi gặp màn hình đích.

**Ví dụ 1: Dismiss đơn giản**
```tsx
router.dismissTo('/');

// Stack hiện tại: [Index, Second, Third]
// Sau dismissTo('/'): [Index]
// → Đã xóa Second và Third
```

**Ví dụ 2: Dismiss với multiple instances**
```tsx
// Stack: [Index, Second, Third, Index, Second, Third]
router.dismissTo('/');

// Kết quả: [Index, Second, Third, Index]
// → Chỉ dismiss đến instance INDEX GẦN NHẤT (topmost)
```

**Quan trọng:** `dismissTo` dismiss đến **instance trên cùng** của màn hình đó trong stack.

### **C) REPLACE**
Thay thế màn hình hiện tại bằng màn hình mới.

**Ví dụ:**
```tsx
// Stack hiện tại: [Index, Second, Third]
router.replace('/second');

// Stack sau replace: [Index, Second, Second]
// → Third bị thay bằng Second
```

**Use case thực tế:**
```tsx
// Màn hình login thành công → replace bằng home
// Để user không thể back về login
router.replace('/home');
```

### **D) NAVIGATE (mặc định của Link)**
**Lưu ý quan trọng về phiên bản:**

**Trước React Navigation 7:**
```tsx
// Stack: [Index, Second]
navigate('/second');
// → Dismiss về Second có sẵn, không push mới
// Kết quả: [Index, Second]
```

**Từ React Navigation 7 trở đi:**
```tsx
// Stack: [Index, Second]
navigate('/second');
// → Hoạt động giống PUSH
// Kết quả: [Index, Second, Second]
```

**Kết luận:** Hiện tại `navigate` ≈ `push`, nhưng tùy phiên bản router.

---

## **3. TRUYỀN DỮ LIỆU GIỮA CÁC MÀN HÌNH**

### **A) Sử dụng Link Component với Object**

**Cách 1: String interpolation**
```tsx
<Link href="/second?name=John">
  Go to Second
</Link>
```

**Cách 2: Object syntax (khuyến nghị)**
```tsx
<Link href={{
  pathname: '/second',
  params: { 
    name: 'John',
    age: 25 
  }
}}>
  Go to Second
</Link>
```

### **B) Sử dụng useRouter Hook**

```tsx
const router = useRouter();

router.push({
  pathname: '/second',
  params: { 
    name: 'Alice',
    message: 'Hello World' 
  }
});
```

### **C) Nhận dữ liệu với useLocalSearchParams**

**Không có TypeScript:**
```tsx
import { useLocalSearchParams } from 'expo-router';

export default function SecondScreen() {
  const { name, age } = useLocalSearchParams();
  
  return (
    <View>
      <Text>Hello {name}</Text>
      <Text>Age: {age}</Text>
    </View>
  );
}
```

**Có TypeScript (type-safe):**
```tsx
import { useLocalSearchParams } from 'expo-router';

type Params = {
  name: string;
  age: string; // Params luôn là string!
};

export default function SecondScreen() {
  const { name, age } = useLocalSearchParams<Params>();
  
  // name và age có type checking
  const greeting = name ? `Hello ${name}` : 'Hello Guest';
  
  return <Text>{greeting}</Text>;
}
```

**Lưu ý quan trọng:**
- Params phải là **serializable object** (không truyền functions, classes)
- Tất cả params đều là **string** khi nhận được

---

## **4. DYNAMIC ROUTES**

### **A) Dynamic Segment Đơn**

**Cấu trúc file:**
```
app/
  proverbs/
    [id].tsx    ← Square brackets = dynamic
```

**Route tương ứng:**
```
/proverbs/1      → id = "1"
/proverbs/abc    → id = "abc"
/proverbs/999    → id = "999"
```

**Code trong [id].tsx:**
```tsx
import { useLocalSearchParams } from 'expo-router';

type Params = {
  id: string;
};

export default function ProverbScreen() {
  const { id } = useLocalSearchParams<Params>();
  
  const proverbs = [
    { id: '1', text: 'Actions speak louder than words', source: 'English' },
    { id: '2', text: 'A journey of 1000 miles...', source: 'Chinese' }
  ];
  
  const proverb = proverbs.find(p => p.id === id);
  
  if (!proverb) {
    return <Text>Not Found</Text>;
  }
  
  return (
    <View>
      <Text>{proverb.text}</Text>
      <Text>Source: {proverb.source}</Text>
    </View>
  );
}
```

**Link đến dynamic route:**

Cách 1:
```tsx
<Link href="/proverbs/1">Proverb 1</Link>
```

Cách 2 (explicit):
```tsx
<Link href={{
  pathname: '/proverbs/[id]',
  params: { id: '1' }
}}>
  Proverb 1
</Link>
```

### **B) Multiple Dynamic Segments**

**Cấu trúc file:**
```
app/
  products/
    [category]/
      [productId].tsx
```

**Routes:**
```
/products/shoes/1234     → category="shoes", productId="1234"
/products/phones/5678    → category="phones", productId="5678"
/products/books/abc      → category="books", productId="abc"
```

**Code:**
```tsx
type Params = {
  category: string;
  productId: string;
};

export default function ProductScreen() {
  const { category, productId } = useLocalSearchParams<Params>();
  
  return (
    <View>
      <Text>Category: {category}</Text>
      <Text>Product ID: {productId}</Text>
    </View>
  );
}
```

**Link:**
```tsx
<Link href="/products/shoes/1234">Nike Air Max</Link>

// hoặc
<Link href={{
  pathname: '/products/[category]/[productId]',
  params: { 
    category: 'shoes',
    productId: '1234'
  }
}}>
  Nike Air Max
</Link>
```

### **C) Dynamic Segments Ở Giữa Route**

**Ví dụ:**
```
app/
  users/
    [userId]/
      posts/
        [postId].tsx

→ Route: /users/123/posts/456
→ userId="123", postId="456"
```

---

## **5. CẤU HÌNH SCREENS VỚI SCREEN OPTIONS**

### **A) Cấu hình trong Layout File**

Expo Router không bắt buộc khai báo <Stack.Screen />
Nhưng khai báo để:
Set title
Set animation
Set header style

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack>
      <Stack.Screen 
        name="proverbs/[id]"  // Tên = pathname (không có /)
        options={{
          title: 'Proverb',
          headerStyle: { backgroundColor: '#f4511e' },
          headerTintColor: '#fff',
          headerTitleStyle: { fontWeight: 'bold' }
        }}
      />
    </Stack>
  );
}
```

**Lưu ý:** `name` là pathname **không có** dấu `/` ở đầu.

### **B) Dynamic Title Dựa Trên Route**

```tsx
<Stack.Screen 
  name="proverbs/[id]"
  options={({ route }) => ({
    title: `Proverb #${route.params.id}`
  })}
/>
```

**Vấn đề:** Không truy cập được data bên trong component (như proverb.source).

### **C) Cấu hình Trong Screen Component (Khuyến nghị)**

```tsx
// app/proverbs/[id].tsx
import { Stack } from 'expo-router';

export default function ProverbScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const proverb = proverbs.find(p => p.id === id);
  
  return (
    <>
      <Stack.Screen 
        options={{
          title: proverb?.source || 'Proverb'
        }}
      />
      
      <View>
        <Text>{proverb?.text}</Text>
      </View>
    </>
  );
}
```

**Ưu điểm:**
- Truy cập được data của component
- Screen options override layout options (nếu trùng)

### **D) Các Screen Options Phổ Biến**

```tsx
<Stack.Screen 
  options={{
    // Tiêu đề
    title: 'My Screen',
    headerTitle: 'Custom Title',
    
    // Style
    headerStyle: { backgroundColor: '#333' },
    headerTintColor: '#fff',
    headerTitleStyle: { fontSize: 20 },
    
    // Buttons
    headerLeft: () => <BackButton />,
    headerRight: () => <SettingsButton />,
    
    // Animation
    animation: 'slide_from_bottom', // iOS + Android
    animation: 'none', // Tắt animation
    
    // Ẩn header
    headerShown: false,
    
    // Presentation
    presentation: 'modal',
    presentation: 'transparentModal',
  }}
/>
```

**Khám phá tất cả options:**
```tsx
// Trong editor, Ctrl+Space để xem autocomplete
<Stack.Screen options={{ /* Ctrl+Space here */ }} />
```

### **E) Animation Styles**

**iOS mặc định:** Slide từ phải sang trái
**Android mặc định:** Slide + fade

**Custom animations:**
```tsx
// Cả iOS và Android slide từ dưới lên
<Stack.Screen 
  options={{
    animation: 'slide_from_bottom'
  }}
/>

// Tắt animation hoàn toàn
<Stack.Screen 
  options={{
    animation: 'none'
  }}
/>

// Animation khác
options={{
  animation: 'fade',
  animation: 'fade_from_bottom',
  animation: 'flip',
  animation: 'simple_push',
  animation: 'ios', // iOS style trên Android
  animation: 'default' // Platform default
}}
```

---

## **6. TÓM TẮT CÁC KHÁI NIỆM QUAN TRỌNG**

### **Navigation Methods**
- **push**: Luôn thêm màn hình mới
- **replace**: Thay thế màn hình hiện tại
- **dismissTo**: Dismiss về màn hình cụ thể (instance gần nhất)
- **navigate**: Giống push (từ RN 7), trước đó có logic khác

### **Passing Data**
- Dùng `params` trong Link hoặc router methods
- Params phải serializable (không truyền functions)
- Nhận data bằng `useLocalSearchParams()`
- Params luôn là string

### **Dynamic Routes**
- Dùng `[tên]` cho dynamic segments
- Có thể có nhiều dynamic segments
- Dynamic segment có thể ở giữa route
- Truy cập bằng `useLocalSearchParams()`

### **Screen Configuration**
- Cấu hình trong layout: `<Stack.Screen name="..." options={{...}} />`
- Cấu hình trong screen: `<Stack.Screen options={{...}} />`
- Screen options override layout options
- Có thể dùng function để dynamic options
- Ctrl+Space để xem tất cả options có sẵn

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