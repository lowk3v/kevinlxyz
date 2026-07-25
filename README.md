# Trang cá nhân — Hugo + Netlify

Site tĩnh, **không JavaScript**, không theme bên thứ ba, không `node_modules`.
Toàn bộ layout là 6 file bạn sở hữu và sửa được.

```
hugo.toml                 ← thông tin cá nhân, menu, cấu hình
netlify.toml              ← lệnh build + security headers
data/achievements.yaml    ← danh sách thành tựu
content/
  _index.md               ← trang chủ (nội dung nằm ở hugo.toml)
  cv.md                   ← trang CV
  achievements.md         ← vỏ trang thành tựu
  blog/*.md               ← bài viết
assets/css/skins/         ← ba giao diện: celadon, brass, signal
assets/css/severity.css   ← nhãn mức độ nghiêm trọng, dùng chung mọi skin
layouts/_shortcodes/      ← shortcode sev và finding
layouts/                  ← template
static/                   ← file tĩnh: favicon.svg, cv.pdf, ảnh
```

## 1. Cài đặt

Chỉ cần một file nhị phân, không cần Node:

```bash
# macOS
brew install hugo

# Linux — tải bản extended khớp phiên bản trong netlify.toml
curl -sLO https://github.com/gohugoio/hugo/releases/download/v0.156.0/hugo_extended_0.156.0_linux-amd64.tar.gz
tar xzf hugo_extended_0.156.0_linux-amd64.tar.gz hugo && sudo mv hugo /usr/local/bin/
```

Chạy thử:

```bash
hugo server -D          # http://localhost:1313, tự reload khi lưu file
```

## 2. Viết nội dung

**Bài blog mới**

```bash
hugo new content blog/ten-bai-viet.md
```

Mở file, điền `description`, xoá `draft: true` khi muốn xuất bản.

```yaml
---
title: "Tiêu đề bài viết"
date: 2026-07-25
description: "Một câu tóm tắt, hiện ở trang danh sách và thẻ chia sẻ."
---
```

**Thành tựu mới** — thêm một khối vào đầu `data/achievements.yaml`:

```yaml
- year: "2026"
  title: "Tên thành tựu"
  org: "Đơn vị · thông tin phụ"      # tuỳ chọn
  note: "Một dòng mô tả."            # tuỳ chọn
  url: "https://..."                 # tuỳ chọn, biến tiêu đề thành link
```

**CV** — sửa `content/cv.md` bằng Markdown thường. Đặt file PDF vào
`static/cv.pdf` để link "Tải bản PDF" hoạt động.

**Thông tin cá nhân** — tên, chức danh, tagline, email, mạng xã hội đều nằm
trong khối `[params]` của `hugo.toml`. Không cần đụng vào layout.

## 3. Đổi giao diện

Có sẵn hai skin. Đổi bằng một dòng trong `[params]` của `hugo.toml`:

```toml
skin = "signal"   # "celadon" | "brass" | "signal"
```

| | celadon | brass | signal |
|---|---|---|---|
| Tiêu đề | serif | sans đậm | serif |
| Thân bài | sans | serif | serif |
| Cấu trúc | một gáy kẻ dọc liền | khối nổi có viền | mỗi mục một vạch riêng |
| Nền | xanh đá lạnh | tím than ấm | xám than lạnh |
| Điểm nhấn | celadon nhạt | đồng thau | thép xanh |
| Dành cho | blog cá nhân chung | blog cá nhân chung | offensive security |

`signal` giữ toàn bộ khung trang ở tông lạnh và **để dành dải đỏ–cam–vàng
cho nhãn severity**. Nếu bạn đổi `--accent` của nó sang màu ấm, nhãn
Critical sẽ không còn nổi bật — đó là điểm chính của skin này.

Cả hai dùng chung class HTML nên không cần đụng layout. Viết skin thứ ba:
copy một file trong `assets/css/skins/`, đổi tên, trỏ `skin` tới nó.
Nếu gõ sai tên, Hugo báo lỗi lúc build chứ không im lặng cho ra trang trắng.

Bảy biến màu ở đầu mỗi file skin:

```css
--bg:        #121417;   /* nền */
--surface:   #181b1f;   /* khối code, ô nổi */
--line:      #262a2f;   /* đường kẻ */
--text:      #e4e6e3;   /* chữ chính */
--muted:     #8b9298;   /* chữ phụ */
--accent:    #9dbfae;   /* link, điểm nhấn */
--accent-dim:#5d7a6c;   /* viền nút, nhãn nhỏ */
```

Font hiện dùng font hệ thống (serif cho tiêu đề, sans cho nội dung, mono cho
nhãn) — không gọi ra CDN nào, nên CSP giữ được mức nghiêm ngặt nhất. Nếu muốn
font riêng, đặt file `.woff2` vào `static/fonts/`, khai báo `@font-face` trỏ
tới `/fonts/...` và giữ nguyên `font-src 'self'`.

## 3b. Nhãn mức độ nghiêm trọng

Hai shortcode, dùng được ở mọi skin.

**Giữa dòng:**

```
Lỗi này được chấm {{</* sev high */>}} sau khi xem lại.
{{</* sev critical "CVSS 9.1" */>}}
{{</* sev medium "CVE-2026-1337" */>}}
```

**Khối finding:**

```
{{</* finding level="critical" title="SSRF tại /api/avatar" id="F-001" cvss="9.1" */>}}
Nội dung Markdown bình thường: đoạn văn, danh sách, khối code.

#### Khắc phục

Chuyển sang allowlist theo domain.
{{</* /finding */>}}
```

`level` nhận: `critical`, `high`, `medium`, `low`, `info`. Gõ sai thì Hugo
báo lỗi kèm số dòng và **dừng build** — Netlify sẽ báo deploy failed thay vì
xuất bản một nhãn không có màu.

Xem `content/blog/ssrf-toi-quyen-quan-tri.md` để có ví dụ đầy đủ.

## 4. Deploy lên Netlify

```bash
git init && git add -A && git commit -m "khởi tạo"
git remote add origin git@github.com:username/repo.git
git push -u origin main
```

Trên Netlify: **Add new site → Import an existing project → GitHub → chọn repo**.
Netlify đọc `netlify.toml` nên không cần điền gì thêm — bỏ trống cả build
command lẫn publish directory.

Sau khi có domain, sửa `baseURL` trong `hugo.toml` cho khớp rồi push lại.
Nếu để sai, RSS và thẻ canonical sẽ trỏ nhầm.

Từ đó về sau: `git push` là deploy.

## 5. Bảo mật — việc còn lại sau khi deploy

`netlify.toml` đã cấu hình sẵn CSP, HSTS, `X-Frame-Options`, `Permissions-Policy`.
Những việc phải làm thủ công:

- [ ] Bật **2FA** cho tài khoản Netlify và GitHub. Đây là bề mặt tấn công thật
      sự của một site tĩnh — không ai hack HTML, họ chiếm tài khoản deploy.
- [ ] Netlify → Site settings → **Build & deploy → Deploy contexts**: tắt
      deploy preview nếu repo public và bạn không cần nó.
- [ ] Kiểm tra điểm bảo mật tại `securityheaders.com` — cấu hình này đạt A+.
- [ ] Bật HTTPS + **HSTS preload** trong Netlify → Domain management.
- [ ] Không bao giờ đặt secret vào `hugo.toml` hay biến môi trường build.
      Mọi thứ trong site tĩnh đều công khai.

**Nếu sau này thêm JavaScript hoặc dịch vụ ngoài**, CSP hiện tại sẽ chặn nó.
Đó là ý đồ. Khi đó mở đúng phần cần thiết trong `netlify.toml`, ví dụ thêm
analytics tự host: `script-src 'self'; connect-src 'self' https://analytics.domain.cua-ban`.
Tránh `'unsafe-inline'` — nó vô hiệu hoá gần hết tác dụng của CSP.

## Ghi chú

- Site build trong ~80ms với 11 trang; mỗi trang HTML dưới 10KB.
- Không có `style=""` inline ở bất kỳ đâu — bắt buộc, vì `style-src 'self'`
  trong CSP sẽ chặn chúng. Nếu thêm layout mới, dùng class, đừng dùng inline.
- Cả ba skin đã rà toàn bộ tương phản chữ/nền trên 5 loại trang, đạt WCAG AA
  — kể cả nhãn severity trên nền tint của chính nó.
- CSS được nối, minify và gắn hash SHA-384 (SRI) tự động lúc build.
- `unsafe = false` trong cấu hình Goldmark chặn HTML thô nhúng trong Markdown.
