# assets/ — sprite cho Sky Hunter AI

Game ưu tiên dùng ảnh PNG trong thư mục này; **thiếu file nào thì tự vẽ bằng code** (fallback),
nên game luôn chạy được kể cả khi chưa có ảnh.

## ĐANG DÙNG: ảnh boss `boss_01.png` … `boss_08.png`
- **Màn 1 → 6**: dùng `boss_01.png` → `boss_06.png` (mỗi màn 1 boss khác nhau).
- **Boss cuối (màn 7)**: `boss_07.png`, và **đổi sang `boss_08.png` khi boss NỔI ĐIÊN (phase 3)**.
- Ảnh ngang cũng OK — code tự giữ tỉ lệ và vẽ to theo bề rộng.
- (Máy bay người chơi & lính hiện vẫn vẽ bằng code vì chưa có ảnh — thêm `player.png`, `enemy-*.png` theo bảng dưới nếu muốn.)

## Danh sách file (đặt ĐÚNG tên này)

| File | Dùng cho | Hướng mũi | Kích thước gợi ý |
|------|----------|-----------|------------------|
| `player.png` | Máy bay người chơi | **hướng LÊN** ⬆ | ~128–256px |
| `drone.png` | Lính bay kèm (Lv.5+) | lên | ~64px |
| `enemy-basic.png` | Lính thường | **hướng XUỐNG** ⬇ | ~96–128px |
| `enemy-shooter.png` | Lính bắn đạn | xuống | ~96–128px |
| `enemy-tank.png` | Lính trâu máu | xuống | ~160px |
| `boss-normal.png` | Boss màn 1–6 | xuống | ~512px |
| `finalboss-core.png` | Thân/core boss cuối | xuống | ~640px |
| `finalboss-turret.png` | Pháo phụ boss cuối (2 cái) | — | ~160px |

## Chuẩn ảnh
- **PNG nền trong suốt** (alpha), nhìn từ trên xuống.
- Tông **neon sci-fi** (xanh `#38e8ff` / hồng `#ff3d7f` / vàng `#ffd23f`) cho khớp giao diện.
- Vuông là đẹp nhất (code tự giữ tỉ lệ, vẽ theo cạnh dài nhất).

## Nguồn ảnh miễn phí (CC0 — không lo bản quyền)
- **Kenney – Space Shooter Redux / Space Shooter Extension**: https://kenney.nl/assets/space-shooter-redux
  (tải zip, lấy các file `playerShip*.png`, `enemy*.png`, `ufo*.png`… rồi đổi tên theo bảng trên).
- Hoặc tự tạo bằng AI (DALL·E/Bing) với mô tả "top-down neon spaceship, transparent background".

> Thay ảnh: chỉ cần bỏ file đúng tên vào đây rồi tải lại trang — không cần sửa code.
