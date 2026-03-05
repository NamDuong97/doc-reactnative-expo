# useCallback
> **React Hook** — cho phép bạn **lưu trữ (cache) một hàm giữa các lần re-render** của component.

```js
const cachedFn = useCallback(fn, dependencies)
```

> 📌 **Lưu ý:** React Compiler có thể tự động memoize các giá trị và hàm, giảm bớt nhu cầu dùng `useCallback` thủ công.

---

## Tham chiếu API

### `useCallback(fn, dependencies)`

Gọi `useCallback` ở **top level** (cấp cao nhất) của component để cache một hàm giữa các lần re-render:

```js
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
}
```

### Tham số (Parameters)

- **`fn`** — Hàm bạn muốn cache. Có thể nhận bất kỳ tham số nào và trả về bất kỳ giá trị nào.
  React sẽ **trả lại hàm** (không gọi nó!) trong lần render đầu tiên. Ở các lần render tiếp theo, nếu dependencies không thay đổi thì React trả về đúng hàm cũ; ngược lại sẽ trả về hàm mới và lưu lại để dùng sau.

- **`dependencies`** — Danh sách tất cả các *reactive value* được dùng bên trong `fn`.
  Reactive value bao gồm props, state, và các biến/hàm khai báo trực tiếp trong component. React dùng thuật toán `Object.is` để so sánh từng dependency.

### Giá trị trả về (Returns)

- **Lần render đầu tiên:** trả về đúng hàm `fn` bạn truyền vào.
- **Các lần render tiếp theo:** trả về hàm `fn` đã cache (nếu dependencies không đổi), hoặc hàm `fn` mới trong lần render hiện tại.

### Lưu ý quan trọng (Caveats)

- `useCallback` là Hook — chỉ được gọi ở **top level** của component hoặc custom Hook. Không được gọi bên trong vòng lặp hay điều kiện.
- React sẽ không xóa cache trừ khi có lý do cụ thể (ví dụ: chỉnh sửa file trong development, hoặc component bị suspend trong lần mount đầu tiên).

---

## Hướng dẫn sử dụng

### 1. Bỏ qua việc re-render của component con

Khi tối ưu hiệu năng render, bạn cần cache các hàm truyền xuống component con.

Giả sử bạn truyền `handleSubmit` từ `ProductPage` xuống `ShippingForm`, và nhận thấy mỗi khi đổi `theme` thì app bị lag. Nguyên nhân: mặc định, khi component cha re-render, **toàn bộ component con cũng re-render theo**.

Bạn có thể dùng `memo` để bỏ qua re-render khi props không đổi:

```js
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

Nhưng nếu không dùng `useCallback`, mỗi lần `ProductPage` re-render sẽ tạo ra một `handleSubmit` **hoàn toàn mới** — dù nội dung y hệt:

```js
// ❌ Mỗi lần theme thay đổi, đây sẽ là một hàm KHÁC nhau
function handleSubmit(orderDetails) {
  post('/product/' + productId + '/buy', { referrer, orderDetails });
}
// => memo mất tác dụng, ShippingForm vẫn re-render mỗi lần
```

> 💡 **Tại sao hàm lại luôn "mới"?**
> Trong JavaScript, mỗi lần viết `function(){}` hay `() => {}` đều tạo ra một object hàm mới — giống như `{}` luôn tạo object mới. Vì vậy props luôn thay đổi, khiến `memo` mất tác dụng.

Giải pháp đúng với `useCallback`:

```js
// ✅ React cache hàm này giữa các lần re-render
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', {
    referrer,
    orderDetails,
  });
}, [productId, referrer]); // chỉ tạo lại nếu productId hoặc referrer thay đổi
```

> ⚠️ **Khi nào nên dùng?** Chỉ dùng `useCallback` khi có lý do cụ thể: truyền hàm cho component dùng `memo`, hoặc hàm là dependency của Hook khác. Nếu code chạy sai khi bỏ `useCallback` đi — đó là bug, hãy fix bug trước.

---

### useCallback và useMemo khác nhau thế nào?

Cả hai đều giúp tối ưu khi truyền dữ liệu xuống component con, nhưng cache những thứ khác nhau:

```js
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  // useMemo: cache KẾT QUẢ của hàm
  const requirements = useMemo(() => {
    return computeRequirements(product); // gọi hàm và lưu kết quả
  }, [product]);

  // useCallback: cache BẢN THÂN hàm (không gọi nó)
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', { referrer, orderDetails });
  }, [productId, referrer]);

  return <ShippingForm requirements={requirements} onSubmit={handleSubmit} />;
}
```

| Hook | Cache cái gì? | Dùng khi nào? |
|------|--------------|---------------|
| `useMemo` | Kết quả trả về của hàm | Tính toán tốn kém, cần tái sử dụng kết quả |
| `useCallback` | Bản thân hàm (tham chiếu) | Truyền callback xuống component `memo` |

> 💡 **Mẹo ghi nhớ:** `useCallback(fn, deps)` tương đương với `useMemo(() => fn, deps)` — chỉ khác là cache hàm thay vì cache kết quả.

---

### 2. Cập nhật state từ callback đã được memoize

Đôi khi bạn cần cập nhật state dựa trên giá trị state trước đó trong callback memoize:

```js
// ❌ Phải khai báo todos là dependency — không tốt
const handleAddTodo = useCallback((text) => {
  const newTodo = { id: nextId++, text };
  setTodos([...todos, newTodo]); // đọc todos trực tiếp
}, [todos]); // cache bị xóa mỗi khi todos thay đổi
```

Giải pháp: dùng **updater function** để loại bỏ dependency `todos`:

```js
// ✅ Không cần todos trong dependency array
const handleAddTodo = useCallback((text) => {
  const newTodo = { id: nextId++, text };
  setTodos(todos => [...todos, newTodo]); // nhận todos qua callback
}, []); // dependency array rỗng!
```

> 📌 **Updater function là gì?**
> Thay vì đọc state trực tiếp, bạn truyền một hàm vào setter: `setTodos(prev => [...prev, newItem])`.
> React sẽ gọi hàm này với giá trị state **hiện tại nhất**, giúp tránh *stale closure* (đọc phải giá trị cũ).

---

### 3. Tránh Effect chạy quá thường xuyên

Khi gọi một hàm bên trong `useEffect`, hãy cẩn thận:

```js
// ❌ Vấn đề: createOptions là dependency nhưng luôn mới mỗi render
function ChatRoom({ roomId }) {
  function createOptions() { // hàm mới mỗi lần render!
    return { serverUrl: 'https://localhost:1234', roomId };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // => re-connect liên tục!
}
```

**Giải pháp 1:** Dùng `useCallback` để ổn định tham chiếu hàm:

```js
// ✅ Giải pháp 1: useCallback
const createOptions = useCallback(() => {
  return { serverUrl: 'https://localhost:1234', roomId };
}, [roomId]); // chỉ thay đổi khi roomId thay đổi
```

**Giải pháp 2 (đơn giản hơn):** Chuyển hàm vào trong Effect:

```js
// ✅ Giải pháp 2: đặt hàm trong Effect — không cần useCallback!
useEffect(() => {
  function createOptions() {
    return { serverUrl: 'https://localhost:1234', roomId };
  }
  const options = createOptions();
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]); // chỉ phụ thuộc roomId
```

---

### 4. Tối ưu hóa Custom Hook

Khi viết custom Hook, hãy bọc mọi hàm trả về bằng `useCallback` để người dùng Hook có thể tối ưu phía họ khi cần:

```js
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return { navigate, goBack };
}
```

---

## Xử lý sự cố

### Vấn đề 1: `useCallback` luôn trả về hàm mới

Nguyên nhân phổ biến nhất: **quên truyền dependency array!**

```js
// ❌ Không có dependency array => luôn trả về hàm mới
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', { referrer, orderDetails });
}); // thiếu array!

// ✅ Đúng cách
const handleSubmit = useCallback((orderDetails) => {
  post('/product/' + productId + '/buy', { referrer, orderDetails });
}, [productId, referrer]);
```

Nếu vẫn không được, debug bằng cách log dependencies:

```js
console.log([productId, referrer]);
// Lưu 2 log thành temp1 và temp2 trong DevTools Console, rồi kiểm tra:
Object.is(temp1[0], temp2[0]); // dependency đầu có giống không?
Object.is(temp1[1], temp2[1]); // dependency sau có giống không?
```

---

### Vấn đề 2: Không thể dùng `useCallback` trong vòng lặp

Hooks không được gọi bên trong vòng lặp — đây là quy tắc bất biến của React:

```js
// ❌ SAI: không gọi useCallback trong loop
function ReportList({ items }) {
  return items.map(item => {
    const handleClick = useCallback(() => sendReport(item), [item]); // LỖI!
    return <Chart onClick={handleClick} />;
  });
}
```

**Giải pháp 1:** Tách thành component riêng:

```js
// ✅ useCallback ở top level của component riêng
function ReportList({ items }) {
  return items.map(item => <Report key={item.id} item={item} />);
}

function Report({ item }) {
  const handleClick = useCallback(() => sendReport(item), [item]); // OK!
  return <Chart onClick={handleClick} />;
}
```

**Giải pháp 2:** Bọc component trong `memo` — không cần `useCallback`:

```js
// ✅ Dùng memo trên component
const Report = memo(function Report({ item }) {
  function handleClick() { sendReport(item); } // không cần useCallback
  return <Chart onClick={handleClick} />;
});
// => nếu item không đổi, Report sẽ không re-render
//    và Chart cũng sẽ không re-render theo
```

---

## Tóm tắt

| ✅ NÊN dùng | ❌ KHÔNG cần dùng |
|------------|-----------------|
| Truyền hàm vào component dùng `memo` | Hàm chỉ dùng nội bộ, không truyền xuống |
| Hàm là dependency của Hook khác | Component nhận không dùng `memo` |
| Hàm trả về từ custom Hook | Tối ưu sớm khi chưa có vấn đề rõ ràng |

> ✅ **Nguyên tắc vàng:**
> `useCallback` **không ngăn** việc tạo hàm mới — hàm vẫn được tạo ra mỗi lần render. Điểm khác biệt là React sẽ bỏ qua hàm mới đó và trả lại hàm đã cache nếu dependencies không thay đổi.
>
> Dùng **React DevTools Profiler** để xác định component thực sự cần tối ưu trước khi thêm `useCallback`.