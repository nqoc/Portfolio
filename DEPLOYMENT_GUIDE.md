# HƯỚNG DẪN TRIỂN KHAI PORTFOLIO LÊN INTERNET VỚI TÊN MIỀN TÙY CHỈNH & CLOUDFLARE DNS
**Học phần:** Mạng máy tính (Computer Networking) — Tuần 4  
**Sinh viên:** Nguyễn Như Ngọc — MSSV: 102250296  
**Mục tiêu:** Đưa trang Portfolio chạy 24/7 trên môi trường Internet bằng tên miền riêng, quản trị DNS tập trung qua Cloudflare DNS và kích hoạt chứng chỉ bảo mật HTTPS (SSL/TLS).

---

## TỔNG QUAN LUỒNG KIẾN TRÚC MẠNG (ARCHITECTURE WORKFLOW)

```
[ Người dùng truy cập: https://portfolio.nguyennhungoc.me ]
                         │
                         ▼
             [ 1. Nhà đăng ký tên miền (Registrar) ]
     (Namecheap / Name.com qua GitHub Pack HOẶC DigitalPlat FreeDomain)
     => Cấu hình trỏ Custom Nameservers về Cloudflare
                         │
                         ▼
             [ 2. Cloudflare DNS (Anycast CDN & Proxy) ]
     - Nameserver 1: ns1.cloudflare.com
     - Nameserver 2: ns2.cloudflare.com
     - Quản lý bản ghi: CNAME (@ / portfolio) trỏ về Hosting
     - Bật SSL/TLS Edge Certificate (Mã hóa HTTPS 24/7)
                         │
                         ▼
             [ 3. Nền tảng Hosting Static Web (24/7) ]
     (Cloudflare Pages HOẶC GitHub Pages)
     - Lưu trữ mã nguồn HTML5/CSS3/JS
     - Tự động Deploy khi đẩy code lên GitHub Repo
```

---

## BƯỚC 1: ĐĂNG KÝ TÊN MIỀN MIỄN PHÍ (CUSTOM DOMAIN)

Bạn có thể lựa chọn 1 trong 2 nguồn cấp tên miền miễn phí được giảng viên gợi ý:

### Lựa chọn 1A: Sử dụng GitHub Student Developer Pack (Khuyên dùng - Nhận Domain thật .me / .live)
1. Truy cập [https://education.github.com/pack](https://education.github.com/pack) và đăng nhập bằng tài khoản GitHub (đã verify email sinh viên `@sv1.dut.udn.vn` hoặc thẻ sinh viên DUT).
2. Tìm ưu đãi tên miền:
   - **Namecheap:** Miễn phí 1 năm tên miền `.me` kèm SSL.
   - **Name.com:** Miễn phí 1 năm tên miền `.live`, `.software`, `.studio`.
   - **.TECH Domains:** Miễn phí 1 năm tên miền `.tech`.
3. Đăng ký tên miền mong muốn (ví dụ: `nguyennhungoc.me` hoặc `nhungoc.live`).
4. Truy cập trang quản trị tài khoản tại nhà cung cấp (Namecheap/Name.com Dashboard).

### Lựa chọn 1B: Sử dụng DigitalPlat FreeDomain (Nhanh chóng, không cần thẻ)
1. Truy cập [https://dash.domain.digitalplat.org](https://dash.domain.digitalplat.org).
2. Tạo tài khoản miễn phí bằng Email.
3. Chọn đăng ký tên miền miễn phí (đuôi `.us.kg`, `.dpdns.org`, `.qzz.io`...).
4. Sau khi đăng ký thành công, giữ mở trang quản trị để chuẩn bị đổi Nameservers.

---

## BƯỚC 2: CẤU HÌNH CLOUDFLARE DNS (FREE TIER)

1. Truy cập [https://dash.cloudflare.com](https://dash.cloudflare.com) và đăng ký/đăng nhập tài khoản Cloudflare.
2. Tại màn hình chính, nhấn **Add a site** (hoặc **Add domain**).
3. Nhập tên miền bạn vừa đăng ký ở Bước 1 (ví dụ: `nguyennhungoc.me`).
4. Chọn gói dịch vụ: Kéo xuống dưới cùng và chọn **Free ($0)** $\to$ nhấn **Continue**.
5. Cloudflare sẽ tự động quét các bản ghi DNS hiện hữu (nếu có). Nhấn **Continue**.
6. Cloudflare sẽ hiển thị **2 địa chỉ Nameserver độc quyền** dành riêng cho tài khoản của bạn. Ví dụ:
   - `adrian.ns.cloudflare.com`
   - `dina.ns.cloudflare.com`
   *(Hãy ghi nhớ 2 địa chỉ này).*

---

## BƯỚC 3: TRỎ NAMESERVERS TỪ NHÀ ĐĂNG KÝ VỀ CLOUDFLARE

1. Quay trở lại trang quản trị tên miền ở Bước 1 (Namecheap hoặc DigitalPlat):
   - **Nếu dùng Namecheap:** Chọn **Domain List** $\to$ nhấn **Manage** cạnh tên miền $\to$ Tìm mục **Nameservers** $\to$ Đổi từ *Namecheap BasicDNS* sang **Custom DNS** $\to$ Điền 2 Nameservers của Cloudflare vào 2 ô $\to$ Nhấn dấu tích xanh $(\checkmark)$ để lưu.
   - **Nếu dùng DigitalPlat:** Vào mục quản trị tên miền $\to$ Chọn **Manage Nameservers** $\to$ Điền 2 Nameservers của Cloudflare $\to$ Nhấn **Save**.
2. Quay lại Cloudflare, nhấn nút **Check nameservers now**.
3. *Thời gian lan truyền DNS (DNS Propagation):* Thường mất từ 2 đến 15 phút. Khi thành công, Cloudflare sẽ báo trạng thái **Active** (dấu tích xanh: *"Great news! Cloudflare is now protecting your site"*).

---

## BƯỚC 4: TRIỂN KHAI SOURCE CODE PORTFOLIO (HOSTING 24/7)

Bạn có thể chọn triển khai qua **Cloudflare Pages** (cực kỳ tối ưu vì cùng hệ sinh thái Cloudflare) hoặc **GitHub Pages**.

### Cách 1: Triển khai qua Cloudflare Pages (Tối ưu & Khuyên dùng nhất)
1. Đẩy toàn bộ source code của project Portfolio (chứa file `index.html`, thư mục `assets/`) lên một GitHub Repository của bạn (ví dụ: `https://github.com/nqoc/portfolio`).
2. Trên Cloudflare Dashboard, ở thanh menu bên trái chọn **Compute (Workers & Pages)** $\to$ **Pages** $\to$ **Create application** $\to$ **Connect to Git**.
3. Chọn liên kết với tài khoản GitHub và cấp quyền chọn repository `portfolio`.
4. Cấu hình triển khai (Build settings):
   - **Framework preset:** `None`
   - **Build command:** Để trống
   - **Build output directory:** `/` (hoặc để trống nếu `index.html` ở thư mục gốc)
5. Nhấn **Save and Deploy**.
6. Chỉ sau 30 giây, website sẽ được triển khai thành công với đường dẫn mặc định: `https://<ten-project>.pages.dev`.

### Cách 2: Triển khai qua GitHub Pages
1. Tạo Repo trên GitHub `portfolio`, đẩy code lên nhánh `main`.
2. Vào **Settings** của Repo $\to$ chọn **Pages** ở thanh bên trái.
3. Tại mục **Build and deployment**:
   - **Source:** Deploy from a branch.
   - **Branch:** `main` / `root (/)`.
4. Nhấn **Save**. Sau 1–2 phút, website sẽ chạy tại `https://nqoc.github.io/portfolio/`.

---

## BƯỚC 5: GẮN TÊN MIỀN TÙY CHỈNH (CUSTOM DOMAIN) & BẬT HTTPS (SSL)

### Nếu dùng Cloudflare Pages:
1. Tại dự án Pages trên Cloudflare, chuyển qua tab **Custom domains**.
2. Nhấn **Set up a custom domain**.
3. Điền tên miền của bạn (ví dụ: `portfolio.nguyennhungoc.me` hoặc `nguyennhungoc.me`).
4. Vì tên miền của bạn đã được quản trị bởi Cloudflare DNS ở Bước 2–3, Cloudflare sẽ **tự động tạo bản ghi CNAME** trỏ về máy chủ Pages mà bạn không cần phải cấu hình thủ công!
5. Nhấn **Activate domain**.

### Nếu dùng GitHub Pages:
1. Trong GitHub Repo $\to$ **Settings** $\to$ **Pages** $\to$ Điền Custom domain của bạn vào ô **Custom domain** (ví dụ: `portfolio.nguyennhungoc.me`) $\to$ Nhấn **Save**.
2. Vào Cloudflare Dashboard $\to$ Chọn tên miền $\to$ Mục **DNS** $\to$ **Records** $\to$ Nhấn **Add record**:
   - **Type:** `CNAME`
   - **Name:** `portfolio` (hoặc `@` nếu dùng domain gốc)
   - **Target:** `nqoc.github.io`
   - **Proxy status:** Bật **Proxied** (Đám mây màu cam) để hưởng lợi từ CDN và DDoS Protection.
   - **TTL:** Auto $\to$ Nhấn **Save**.

### Kích hoạt SSL/TLS Full & Always Use HTTPS trên Cloudflare:
1. Tại Cloudflare Dashboard $\to$ Chọn tên miền $\to$ Chọn tab **SSL/TLS**:
   - Chọn chế độ mã hóa: **Full** (hoặc **Full (strict)**).
2. Chuyển sang tab con **Edge Certificates**:
   - Bật **Always Use HTTPS** $\to$ `On` (Tự động chuyển hướng mọi truy cập `http://` sang `https://`).
   - Bật **Automatic HTTPS Rewrites** $\to$ `On`.
   - Bật **Minimum TLS Version** $\to$ `TLS 1.2` hoặc `TLS 1.3`.

---

## BƯỚC 6: KIỂM TRA NGHIỆM THU (VERIFICATION CHECKLIST)

Trước khi nộp bài, hãy kiểm tra danh sách tiêu chí nghiệm thu:

- [x] **Truy cập 24/7 bằng HTTPS:** Mở trình duyệt (thử cả chế độ ẩn danh), nhập tên miền riêng (ví dụ `https://nhungoc.live`). Trình duyệt hiển thị biểu tượng ổ khóa bảo mật an toàn (SSL Certificate do Google Trust Services hoặc Let's Encrypt cấp qua Cloudflare).
- [x] **Giao diện hiển thị đầy đủ thông tin:**
  - Giới thiệu bản thân (Nguyễn Như Ngọc, SV Khoa CNTT - DUT, Chương trình tiếng Nhật).
  - 5 dự án tiêu biểu (SmartMatch AI, FairInsight, EduApp, AI Rabies Dog Detection, Sleep Quality Research).
  - Giải thưởng & Thành tích (KHKT thành phố, Tin học trẻ, BK Smart Campus, Web Design).
  - Kỹ năng & Công nghệ (Python, SQL, R, Flutter, HTML/CSS, TensorFlow, NLP...).
  - Swarm Cursors & HUD điều khiển mượt mà.
  - Form liên hệ & các liên kết mạng xã hội (GitHub, Email).
- [x] **File Báo cáo nghiên cứu WebSocket:** File `RESEARCH_WEBSOCKET.md` đã được biên soạn chi tiết, đầy đủ cơ chế RFC 6455, so sánh, bảo mật, và code demo.

---

## NỘI DUNG NỘP BÀI (SUBMISSION TEMPLATE)

Khi nộp vào bảng theo dõi của giảng viên:
1. **Họ và tên:** Nguyễn Như Ngọc
2. **MSSV:** 102250296
3. **Lớp:** CNTT – Chương trình tiếng Nhật (DUT)
4. **Link GitHub Repository:** `https://github.com/nqoc/portfolio` *(hoặc link repo của bạn)*
5. **Link Live Demo (Custom Domain HTTPS):** `https://<ten-mien-cua-ban>`
6. **Link Doc Research WebSocket:** Đường dẫn file `RESEARCH_WEBSOCKET.md` trên GitHub hoặc Google Docs export.
