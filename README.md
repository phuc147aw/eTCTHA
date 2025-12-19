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

### Phương án Google Sheets (Apps Script)
Bạn có thể dùng Google Sheets làm backend chia sẻ, không cần server riêng.

1) Tạo Google Sheet và thêm sheet `links` với hàng tiêu đề ở dòng 1:
	 - A1 `id`, B1 `name`, C1 `url`, D1 `created_at`

2) Từ Google Sheet, vào Extensions → Apps Script, dán mã sau và Deploy → New deployment → Web app, chọn Access: Anyone:

```js
// Apps Script (container-bound to the Sheet)
const SHEET_NAME = 'links';

function _sheet() {
	return SpreadsheetApp.getActive().getSheetByName(SHEET_NAME);
}
function _out(obj) {
	return ContentService.createTextOutput(JSON.stringify(obj))
		.setMimeType(ContentService.MimeType.JSON)
		.setHeader('Access-Control-Allow-Origin', '*');
}

function doGet(e) {
	const sh = _sheet();
	const values = sh.getDataRange().getValues();
	const [header, ...rows] = values;
	const items = rows.filter(r => r[0]).map(r => ({
		id: r[0], name: r[1], url: r[2], created_at: r[3]
	}));
	return _out({ items });
}

function doPost(e) {
	const body = JSON.parse(e.postData && e.postData.contents || '{}');
	const sh = _sheet();
	if (body.action === 'upsert') {
		let id = body.id || Utilities.getUuid();
		const ids = sh.getRange(2, 1, Math.max(0, sh.getLastRow() - 1), 1).getValues().map(r => r[0]);
		const pos = ids.indexOf(id);
		const row = pos >= 0 ? pos + 2 : sh.getLastRow() + 1;
		sh.getRange(row, 1, 1, 4).setValues([[id, body.name || '', body.url || '', new Date()]]);
		return _out({ ok: true, id });
	}
	if (body.action === 'delete') {
		const ids = sh.getRange(2, 1, Math.max(0, sh.getLastRow() - 1), 1).getValues().map(r => r[0]);
		const pos = ids.indexOf(body.id);
		if (pos >= 0) sh.deleteRow(pos + 2);
		return _out({ ok: true });
	}
	return _out({ ok: false, error: 'unknown_action' });
}
```

3) Lấy URL Web App vừa deploy (dạng `https://script.google.com/macros/s/DEPLOY_ID/exec`).

4) Mở `index.html` và thêm cấu hình (bỏ comment):

```html
<script>
	window.ETCTHA_GS_URL = 'https://script.google.com/macros/s/DEPLOY_ID/exec';
</script>
```

- Khi cấu hình xong, trang sẽ dùng Google Sheets làm nơi lưu chia sẻ. Nếu không cấu hình, trang vẫn dùng Supabase (nếu có) hoặc localStorage.
