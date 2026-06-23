# SKY HUNTER AI — Tài liệu dự án game

Game bắn máy bay (shmup) **điều khiển bằng tay qua camera**, co-op 2 người chơi cùng phe.
Toàn bộ game nằm trong **1 file duy nhất**: `index.html` (HTML + CSS + JS thuần, không build, không dependency cài đặt).

---

## 1. Cách chạy

Camera chỉ hoạt động qua `localhost` hoặc `https` (Chrome chặn `file://`). Mở trực tiếp file sẽ KHÔNG bật được camera.

```bash
cd sky-hunter
python3 -m http.server 5500
# rồi mở http://localhost:5500/index.html
```

Hoặc dùng **Live Server** của VS Code. Trình duyệt khuyến nghị: **Chrome**.

Có 2 nút ở màn hình menu:
- **▶ BẮT ĐẦU (CAMERA)** — bật webcam + AI nhận diện tay (cần cho phép quyền camera).
- **🖱 TEST BẰNG CHUỘT** — chơi 1 người bằng chuột, không cần camera (để test nhanh).

---

## 2. Công nghệ

- **MediaPipe Hands** (Google) — nhận diện bàn tay qua webcam, nạp từ CDN jsDelivr:
  - `@mediapipe/hands/hands.js`
  - `@mediapipe/camera_utils/camera_utils.js`
  - Cấu hình: `maxNumHands: 2`, `modelComplexity: 1`.
- **HTML5 Canvas 2D** — vẽ toàn bộ game, chạy theo `requestAnimationFrame` (loop tách rời với MediaPipe).
- **WebAudio API** — toàn bộ âm thanh là **tổng hợp bằng code** (oscillator + noise), KHÔNG có file nhạc nào. Xem object `Sound`.
- Không có backend, không lưu dữ liệu — chơi xong là hết, chưa có high score.

---

## 3. Điều khiển & cách chơi

- Vung **bàn tay trước camera** → lái phi thuyền. Lấy **landmark số 9** (gốc ngón giữa ≈ tâm lòng bàn tay) làm vị trí, **mirror trục X** cho khớp gương.
- Máy bay **tự bắn liên tục** (không cần bấm gì).
- **✊ NẮM TAY rồi XÒE RA = bung REFLECT LASER** (skill kiểu Giga Wing, để dành tối đa 2 lượt) — xem mục riêng bên dưới.
- **Hitbox tí hon**: vùng chết thật của máy bay chỉ ~`PLAYER_HITBOX` (9px) ở **chấm sáng giữa thân** — đạn lướt qua cánh KHÔNG chết. Đây là lý do màn đạn dày vẫn né được (cơ chế danmaku).
- **2 bàn tay = 2 người chơi cùng phe**: P1 màu xanh `#38e8ff`, P2 màu hồng `#ff3d7f`. Chung điểm.
- Tay nào không xuất hiện trước cam thì máy bay đó biến mất (slot `active = false`); đưa tay vào lại là sống lại.
- **Chế độ chuột (test)**: di chuột để lái, **giữ Space (hoặc giữ chuột) rồi THẢ = bắn skill** (khi còn lượt).

### ✊ Reflect Laser — skill đặc biệt 5 giây (linh hồn Giga Wing)
- **2 lượt mỗi mạng** (`REFLECT.maxStock = 2`). **KHÔNG tự hồi theo thời gian** — chỉ **reset lại 2 lượt khi "chết" (dính đạn)** (`loseLevel()`).
- **KHÔNG tự bung.** Khi muốn dùng: **NẮM TAY rồi XÒE RA** (nhả nắm = bắn). Chế độ chuột: **giữ Space rồi thả**. Mỗi lần bắn tốn 1 lượt.
- Thẻ người chơi hiển thị **số lượt còn lại** (`✊×N`) và **số lần chết** (`💀 N`).
- Khi bật, trong **5 giây** (`REFLECT.duration`):
  - **BẤT TỬ** với mọi đạn + thân địch.
  - Tự **phun cơn mưa đạn vàng khổng lồ** lên trên (`REFLECT.volley` viên mỗi `fireInterval`, dmg cao) — đúng kiểu *Reflect Laser*, quét sạch màn hình & nướng boss.
  - **Hút đạn địch** trong bán kính `REFLECT.radius` → mỗi viên +1 combo; hết 5s cộng điểm `absorbed × REFLECT.comboPer`.
- Hết 5 giây: charge về 0, phải nạp lại mới bật được lần nữa.
- HUD: mỗi thẻ có **thanh laser** (đầy = vàng); trạng thái `đang nạp…` / `SẴN SÀNG ✊` / `⚡ LASER 5.0s` (đếm ngược).

### Vòng lặp gameplay
1. **Lính địch** rơi từ trên xuống theo đường sin.
2. Bắn hạ địch → **rớt viên năng lượng xanh**.
3. Đưa máy bay chạm viên năng lượng → **nạp năng lượng → lên cấp vũ khí + nạp khiên**.
4. **Cày lính liên tục ~1 phút rưỡi** (`STAGE_DURATION = 90s`, trần `ENEMY_CAP = 32`) → hết giờ + sạch lính thì **TRÙM khổng lồ** xuất hiện. HUD có đồng hồ **"BOSS SAU m:ss"** đếm ngược.
5. **Tổng 7 màn** (`FINAL_LEVEL`): màn 1–6 boss thường, **màn 7 = BOSS CUỐI**. Hạ boss cuối → màn hình **CHIẾN THẮNG** (`#victory`).

### Các loại địch
| Loại | Máu | Đặc điểm | Rớt năng lượng |
|------|-----|----------|----------------|
| `basic` | 1 | Máy bay chữ thập thường | ~18% rớt 1–2 viên |
| `shooter` | 2 | Bắn 1–3 tia nhanh (420), tần suất cao | ~18% rớt 1–2 viên |
| `tank` | 5 | To, chậm, trâu máu | chắc chắn 1 viên |
| `boss` | 5400 + level×2100 | Boss thường màn 1–6 (CỰC TRÂU ~x3), 4 pattern bullet hell dày | mưa 6 viên khi chết |
| `boss cuối` | core 27000 + 2 pháo ×2700 | Màn 7, **3 phase + phá bộ phận** | mưa orb khi chết |

> **Năng lượng RA HIẾM** (không phải lính nào cũng rớt) → người chơi không phải lúc nào cũng max cấp, nên boss & lính khó nhằn hơn nhiều. Thanh tuyệt chiêu vẫn tự hồi thụ động nên thỉnh thoảng vẫn có lượt.

Boss thường có **4 pattern** luân phiên (mỗi 6 giây): **xoắn ốc** → **hoa nở (vòng tròn)** → **tường đạn ngắm** → **mưa rèm dọc**. Số đạn địch trên màn bị chặn trần ở `EB_CAP` (≈620) để giữ FPS.

### ☠ Boss cuối (màn 7) — 3 phase, phá bộ phận
- **Phase 1**: 2 **pháo phụ** 2 bên cùng bắn; **core bất khả xâm phạm** — phải phá hết 2 pháo trước.
- **Phase 2** (hết pháo): core lộ diện, ăn damage được; thêm pattern **xoắn ốc kép**.
- **Phase 3** (core < 30% máu): **NỔI ĐIÊN** — xoắn 4 cánh nhanh + mưa rèm, đạn dày nhất game.
- Mỗi lần đổi phase: xoá bớt đạn trên màn (`clearSomeBullets`) + toast cảnh báo cho dễ thở.

### Cấp vũ khí (theo từng người chơi) — hàm `WEAPON.spec(level)`
| Lv | Đạn | Lính bay kèm (drone) |
|----|-----|----------------------|
| 1 | 1 tia thẳng | — |
| 2 | 2 tia | — |
| 3 | 3 tia | — |
| 4 | toả 5 tia | — |
| 5 | toả 5 tia | **+2 lính** tự bắn |
| 6 | toả 5 tia, bắn nhanh hơn | +2 lính |

"Lính" (drone) bay kèm 2 bên máy bay từ Lv.5, tự động bắn đạn vàng.

---

## 4. Trạng thái HIỆN TẠI (đã chỉnh theo yêu cầu người chơi)

> Đây là các tinh chỉnh độ khó đang bật. Nếu muốn khó lại như bản gốc thì sửa ngược lại.

- **♾️ BẤT TỬ nhưng TỤT 1 CẤP khi dính đạn**: trúng đạn/đâm địch **KHÔNG mất mạng, KHÔNG game over** — gọi `loseLevel(p)`: **vũ khí −1 cấp** (sàn Lv.1), reset năng lượng, **giữ charge laser**. KHÔNG về thẳng Lv.1 nữa.
  - Hệ quả: `gameOver()` và màn hình "HẾT MẠNG" hiện không bao giờ được gọi (code vẫn còn, để dành bật lại). HUD hiển thị `MẠNG: ∞`.
  - Lưu ý: lính địch **thoát xuống đáy** chỉ rung màn (`loseLife()`), KHÔNG tụt cấp.
- **Khi đang bật Reflect Laser thì bất tử hoàn toàn** với đạn + thân địch (không bị tụt cấp).
- **Năng lượng RA HIẾM**: chỉ ~18% lính rớt 1–2 viên (tank chắc chắn 1). Chỉnh tỉ lệ trong `killEnemy()`.
- **Lên cấp nhanh**: ngưỡng lên cấp khởi đầu `energyMax = 3` (bản gốc là 6), mỗi cấp +1. Lên cấp dùng vòng `while` nên ăn dồn có thể nhảy nhiều cấp một lúc.

---

## 5. Bản đồ code (trong `index.html`, phần `<script>`)

Tất cả ở 1 file, tìm theo tên hàm/biến:

- `Sound` — engine âm thanh WebAudio (shoot/boom/power/levelup/hurt/boss…).
- `game` — object chứa toàn bộ state runtime (score, level, mảng enemies/bullets/orbs/particles/drones, boss, `bossQueued`…).
- `players` — 2 slot người chơi cố định; kích hoạt khi có tay/chuột map vào. `makePlayer()` khởi tạo (gồm field reflect: `reflectCharge`, `reflecting`, `wantReflect`, `absorbed`).
- `handMap` — map nhãn tay MediaPipe ("Left"/"Right") → slot người chơi, giữ ổn định màu.
- **Skill Reflect Laser**: hằng `REFLECT` (radius/max/regen/orbGain/duration/fireInterval/volley/spread/laserSpeed/comboPer); `updateReflect()` (nạp charge → bật 5s → phun mưa đạn vàng + hút đạn + cộng combo). Hằng `PLAYER_HITBOX`.
- **Penalty**: `loseLevel()` (dính đạn → vũ khí −1 cấp, giữ charge) — thay cho `resetSkills` cũ.
- **Input**: `isFist()` (nhận diện nắm tay), `assignHand()` + `onHandResults()` (camera); `mouseCtl` + `mouseReflect` + listener `mousemove`/`mousedown`/`keydown` Space (chuột).
- **MediaPipe**: `initCamera()` — khởi tạo `Hands` + `Camera`.
- **Sprite**: `SPRITES` manifest, `loadSprite()`, `sprite(key)`, `drawSpriteImg()`. **Chỉ boss dùng ảnh** (`boss_01..08` trong `assets/`); player/drone/lính vẽ tay hoàn toàn. Xem `assets/README.md`.
- **Vũ khí**: `WEAPON.spec(level)`, `firePlayer()` (gồm cả logic drone/lính).
- **Địch**: `spawnEnemy()`, `startLevel()` (mỗi màn = wave lính rồi `bossQueued`), `spawnBoss()` (boss thường), `spawnFinalBoss()` (boss cuối + `parts`), `updateBoss()` + `updateFinalBoss()` (3 phase), `clearSomeBullets()`.
- **Vòng đời**: `beginGame()`, `gameOver()`, `winGame()` (thắng boss cuối → `#victory`), `loseLife()`, `killEnemy()`, `killBoss()` (boss cuối → `winGame`, còn lại → `startLevel(lv+1)`), `collectEnergy()`. Hằng `FINAL_LEVEL`.
- **Vòng lặp chính**: `update(dt)` (logic + va chạm) → `render()` (vẽ) → `loop()` qua `requestAnimationFrame`.
- **Va chạm**: dùng `dist2()` (bình phương khoảng cách) so với tổng bán kính — đạn-ta vs địch/boss, đạn-địch vs người chơi, thân địch vs người chơi, orb vs người chơi.
- **HUD/UI**: các overlay `#menu`, `#gameover`, `#hud`, `#players`, `#boss-wrap`, `#toast`; cập nhật qua `updateHUD()` và `updatePlayerCards()`.

Nền game: nếu bật camera thì vẽ **hình webcam mirror** làm background (phủ lớp tối cho dễ nhìn); chế độ chuột thì vẽ gradient + sao bay.

---

## 6. Các "núm" tinh chỉnh thường dùng

- **Độ khó né đạn (quan trọng)**: hằng `PLAYER_HITBOX` (9 → 7 khó hơn / 12 dễ hơn).
- **Cân bằng skill laser**: object `REFLECT` — `maxStock` (số lượt/mạng = 2), `duration` (5s), `volley`/`fireInterval` (độ dày mưa đạn), `comboPer` (điểm combo). Reset lượt ở `loseLevel()`.
- **Độ dài mỗi màn (cày bao lâu mới tới boss)**: hằng `STAGE_DURATION` (90s). Giảm cho nhanh, tăng cho lâu.
- **Mật độ lính**: `batch`/`spawnTimer` trong khối spawn + trần `ENEMY_CAP` (55). Tăng cap nếu muốn đông hơn.
- **Tự giảm lag**: cờ `spritesOn` — nếu `fpsAvg < 45` quá 2.5s thì tự tắt sprite máy bay/lính (vẽ tay) cho mượt (boss vẫn dùng ảnh). Logic ở hàm `loop()`.
- **Độ trâu của boss**: máu trong `spawnBoss()` (`5400 + lv*2100`) và `spawnFinalBoss()` (core 27000, pháo 2700). Giảm nếu thấy đánh quá lâu.
- **Độ dày/nhanh đạn boss**: `fireCd` + số đạn từng pattern trong `updateBoss()`/`updateFinalBoss()`; tốc đạn lính shooter trong vòng lặp enemies.
- **Số màn / boss cuối**: hằng `FINAL_LEVEL` (7); ngưỡng phase 3 (30%) trong `updateFinalBoss()`.
- **Bật lại chế độ có mạng**: trong `loseLife()` thêm lại `game.lives--; ... if (game.lives<=0) gameOver();`. Thay `loseLevel(p)` ở 2 chỗ va chạm nếu muốn cơ chế phạt khác.
- **Đổi tốc độ lên cấp**: sửa `energyMax` trong `makePlayer()` và bước `+2` trong `collectEnergy()`.
- **Số lượng/độ khó địch mỗi màn**: hàm `startLevel()` (`const count = 4 + lv`) và xác suất loại địch.
- **Bắt đầu sẵn súng max**: đặt `weapon: 6` trong `makePlayer()`.
- **Độ mạnh / kích thước boss thường**: máu `70 + lv*45` và `r` trong `spawnBoss()`; mật độ đạn trong `updateBoss()`.
- **Mật độ bullet hell / FPS**: hằng `EB_CAP` (950 — trần số đạn địch).
- **Tốc độ bắn / số tia**: bảng trong `WEAPON.spec()`.
- **Thay ảnh sprite**: bỏ PNG đúng tên vào `assets/` (xem `assets/README.md`) — không cần sửa code.

---

## 7. Ý tưởng có thể làm tiếp (chưa làm)

- Chế độ **đối kháng PvP** (hiện chỉ co-op).
- Cử chỉ **nắm tay = bắn dồn / xoè tay = bắn toả**.
- **Lưu điểm cao** (localStorage).
- Thêm nhiều loại boss / item buff tạm thời (khiên, bom màn hình).
