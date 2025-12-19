# eTCTHA — Giao diện thiên nhiên

Giao diện đơn giản, đẹp mắt theo chủ đề thiên nhiên để lưu và mở nhanh các liên kết.

## Cách dùng
- Mở file `index.html` trực tiếp (double-click) hoặc dùng PowerShell:

```powershell
Start-Process "c:\Users\dangt\Desktop\eTHADS\index.html"
```

- Nhấn "Thêm ô" để tạo ô mới (vào chế độ nhập).
- Nhập `Tên` và `Đường link` rồi bấm "Lưu".
- Ở chế độ xem: bấm vào phần `Đường link` để mở trực tiếp (không cần nút Mở).
- Dùng nút "Sửa" để quay lại chế độ nhập; "Xóa" để xóa ô.

## Ghi chú
- Giao diện responsive: hiển thị đẹp trên cả máy tính và điện thoại.
- Chủ đạo màu xanh lá, hiệu ứng thủy tinh và hoa văn lá nhẹ nhàng.

## Lưu chung cho mọi người (tùy chọn)
GitHub Pages là trang tĩnh, dữ liệu chỉ lưu cục bộ (localStorage). Để mọi người cùng thấy dữ liệu, hãy dùng Supabase (miễn phí):

1. Tạo dự án tại https://supabase.com, vào Database tạo bảng `links`:
	- `id`: `uuid` (Primary Key, default `gen_random_uuid()`)
	- `name`: `text`
	- `url`: `text`
	- `created_at`: `timestamp` (default `now()`)
2. Bật Row Level Security (RLS) và tạo policies đơn giản:
	- Select: `true` (cho phép đọc công khai)
	- Insert/Update/Delete: `true` (hoặc giới hạn theo nhu cầu; cân nhắc chống spam)
3. Lấy `Project URL` và `anon public key` trong phần Project Settings → API.
4. Mở `index.html`, bỏ comment và điền cấu hình:

```html
<!-- Trong index.html -->
<script>
  window.ETCTHA_CONFIG = { url: 'https://YOUR-project.supabase.co', key: 'YOUR_PUBLIC_ANON_KEY' };
</script>
```

- Khi cấu hình xong, trang sẽ tải/lưu/xoá vào bảng `links` (mọi người đều thấy). Nếu không cấu hình, trang sẽ dùng localStorage cục bộ.

Lưu ý bảo mật: anon key là public; hãy dùng RLS để bảo vệ dữ liệu. Nếu cần kiểm soát chặt (chỉ người đăng nhập mới sửa/xoá), bật Auth và cập nhật policies theo `auth.uid()`.
