# BOTTOM TABS NAVIGATOR - TÓM TẮT QUAN TRỌNG

## **I. TỔNG QUAN BOTTOM TABS**

- Bottom Tabs là kiểu navigation cực kỳ phổ biến trên mobile
- Hiếm gặp trên web, nhưng là chuẩn UX trên mobile apps
- Mỗi tab đại diện cho:
  - Một screen đơn
  - Hoặc một navigator (thường là Stack)
- Navigation trên mobile có cấu trúc dạng **tree**, không phải chỉ là stack

---

## **II. KÍCH HOẠT BOTTOM TABS**

- Bottom Tabs được tạo bằng cách: Layout file return `<Tabs />` navigator
- Layout file quyết định loại navigation
- **Không cần khai báo screen thủ công**
- Tất cả screen dưới layout tự động trở thành tab

---

## **III. TABS.SCREEN**

- **Không bắt buộc** phải khai báo `Tabs.Screen`
- Nên khai báo để:
  - Sắp xếp thứ tự tab
  - Cấu hình icon
  - Cấu hình label, title
  - Cấu hình behavior
- `name` của `Tabs.Screen` là path **relative** tới layout

---

## **IV. TAB ICONS**

- Tab icon là React component
- `tabBarIcon` là function
- Function nhận props:
  - `focused` - tab có đang active không
  - `color` - màu tự động (active/inactive)
  - `size` - kích thước icon
- Icon **nên dùng** `color` & `size` do Tabs truyền vào
- **Tránh hardcode** màu icon

---

## **V. CẤU HÌNH MÀU & LABEL**

- Tabs hỗ trợ cấu hình toàn cục qua `screenOptions`
- Có thể cấu hình:
  - `tabBarActiveTintColor`
  - `tabBarInactiveTintColor`
  - `title`
  - `tabBarLabel`
- `title` mặc định được dùng làm tab label
- Có thể:
  - Đặt label riêng
  - Hoặc ẩn label hoàn toàn

---

## **VI. ẨN TAB**

- Screen có thể tồn tại nhưng **không hiển thị** trên tab bar
- Dùng `href: null`
- Screen vẫn có thể được navigate tới
- Thường dùng cho:
  - Screen phụ
  - Screen chi tiết
  - Flow đặc biệt

---

## **VII. TAB BADGE**

- Tabs hỗ trợ badge sẵn
- Badge hiển thị số hoặc ký hiệu
- Có thể tùy chỉnh style badge
- Thường dùng cho:
  - Thông báo
  - Tin nhắn
  - Giỏ hàng (Cart)

---

## **VIII. NEST STACK TRONG TABS** ⭐ **(CỰC KỲ QUAN TRỌNG)**

- Một tab có thể là:
  - Một screen đơn
  - Hoặc một Stack navigator
- **Rất phổ biến** trong app thực tế
- **Folder chỉ để tổ chức code**
- **Chỉ layout file mới quyết định navigator**
- Không có layout → screen dùng layout cha

---

## **IX. VÌ SAO SCREEN TRONG FOLDER LẠI HIỆN THÀNH TAB**

- Router tìm layout **gần nhất**
- Nếu folder không có layout:
  - Dùng layout cha
- Nếu layout cha là Tabs:
  - Screen sẽ thành tab
- **Layout quyết định số phận screen**

---

## **X. DOUBLE HEADER KHI NEST STACK TRONG TABS**

- Xảy ra khi:
  - Tabs có header
  - Stack bên trong cũng có header
- **Giải pháp:**
  - Ẩn header của navigator ngoài
- Thông thường:
  - Ẩn header của Tabs (`headerShown: false`)
  - Giữ header của Stack

---

## **XI. INDEX ROUTE VÀ TABS**

- `index.tsx` là route **bắt buộc**
- Là screen mở đầu app
- Router luôn tìm index route khi app launch
- **Không thể** xóa hoặc thay thế index route
- Nếu index không tồn tại → **Not Found**

---

## **XII. GROUPING FOLDER VÀ HOME TAB**

- Grouping folder `( )` không ảnh hưởng URL
- Dùng để:
  - Nest index screen vào stack
  - Giữ index route hợp lệ
- **Bắt buộc** dùng grouping folder nếu:
  - Muốn index có nested stack

---

## **XIII. TAB NAVIGATION STATE**

- Mỗi tab có **state riêng**
- Khi chuyển tab:
  - Stack state được **giữ nguyên**
- Khi tap lại tab đang active:
  - Stack **reset** về screen đầu
- Đây là behavior **mặc định** của Tabs

---

## **XIV. POPTOTOPONBLUR**

- `popToTopOnBlur`:
  - Reset stack khi rời tab
- Khi bật:
  - Rời tab → stack reset
- **Không phải** behavior mặc định
- Cần **cân nhắc UX** trước khi dùng

---

## **XV. ANIMATION KHI RESET TAB**

- Reset stack có animation mặc định
- Có thể gây cảm giác không mượt
- Có thể:
  - Điều kiện hóa animation
  - Dựa trên pathname hiện tại
- Cho phép:
  - Animation khi navigate nội bộ
  - Không animation khi reset tab

---

## **XVI. BACK BEHAVIOR CỦA BOTTOM TABS**

- `router.back()` trong Tabs:
  - **Không hoạt động như Stack**
- Mặc định:
  - Quay về **tab đầu tiên**
- Đây là behavior **chuẩn** của Bottom Tabs
- **Không phải lỗi**

---

## **XVII. BACKBEHAVIOR**

- Có thể cấu hình `backBehavior`
- `order`:
  - Back theo thứ tự tab
- Khi bật `order`:
  - Tabs behave giống stack hơn
- `initialRouteName` thường **không đáng tin**
- **Nên tránh** dùng `initialRouteName` với Tabs

---

## **XVIII. NGUYÊN TẮC VÀNG CẦN NHỚ** 🌟

1. **Layout quyết định navigator**
2. **Folder không có layout → không có quyền**
3. **Tabs ≠ Stack**
4. **Mobile navigation = tree structure**
5. **Grouping folder cực kỳ quan trọng với index**
6. **Back behavior của Tabs cần cấu hình rõ**
7. **Mỗi tab là một nhánh navigation độc lập**

---

## **CHECKLIST KHI LÀM VIỆC VỚI TABS**

- [ ] Layout file return `<Tabs />`?
- [ ] Khai báo `Tabs.Screen` để control thứ tự?
- [ ] Icon có dùng props `color` và `size`?
- [ ] Nested stack có ẩn header của Tabs?
- [ ] Index route có dùng grouping folder `(home)`?
- [ ] Đã test behavior khi tap tab đang active?
- [ ] Đã cân nhắc `popToTopOnBlur`?
- [ ] Back behavior có phù hợp với UX?
