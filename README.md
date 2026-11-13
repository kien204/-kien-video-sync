# yt2tiktok — Tự động tải YouTube Shorts (1p–3p, 720p) & đăng TikTok siêu nhanh

Phát triển từ `yt2tiktok_push_v4_1.py`, tối ưu cho mục tiêu **tải + xử lý + đẩy lên TikTok trong ~3s–6s** đối với video dọc **1–3 phút ở độ phân giải 720p**.

## Các tối ưu tốc độ so với v4.1
1. **0ms FFmpeg (Fast-path)**: `FMT_720` ưu tiên tải sẵn luồng `avc1` (H.264) + `mp4a` (AAC) từ YouTube để bỏ qua hoàn toàn bước re-encode `libx264` (vốn tốn 20–60s).
2. **Stream-copy video (`-c:v copy`)**: Nếu video đã là 720p H.264 nhưng khác container/audio, chỉ remux/chuyển audio (~0.3s).
3. **Hardware Encoding (`VIDEO_CODEC=auto`)**: Tự phát hiện `h264_nvenc` / `h264_qsv` / `h264_amf` hoặc dùng `libx264 -preset ultrafast` nếu bắt buộc phải scale/pad từ video ngang 16:9.
4. **In-process `yt-dlp`**: Gọi `yt_dlp.YoutubeDL` trực tiếp trong Python, bỏ chi phí khởi tạo subprocess (~1–1.5s) và bỏ subprocess `--dump-single-json` thứ hai.
5. **HTTP Keep-Alive & Cache `creator_info`**: Dùng `requests.Session` giữ kết nối TLS, cache `creator_info` (`CREATOR_INFO_TTL=600`), và bỏ qua 1 lượt `tt_status` dư thừa ngay sau `tt_init`.
6. **Hỗ trợ 1p–3p & Chạy local không cần domain**: `MAX_DURATION=185`, có thêm `ENABLE_RSS_POLL=1` (chạy không cần HTTPS domain) song song với WebSub (`/yt`), cùng ô nhập link thủ công trên Dashboard (`/`).

> [!NOTE]
> **Giới hạn thực tế**: Tốc độ **3s–6s** đạt được cho phần **Download + Chuẩn hóa + Upload byte lên TikTok** khi:
> - Video YouTube đã là Short dọc 9:16, 720p (`avc1` + `mp4a`), kích thước ~15–45MB,
> - Mạng upload đủ nhanh (100+ Mbps -> 30MB mất ~2.5s),
> - Sau khi upload xong, bước TikTok server xử lý (`PUBLISH_COMPLETE`) phụ thuộc hoàn toàn vào máy chủ TikTok (thường thêm 3–15s).
> - Khi vừa đăng lên YouTube, YouTube mất vài chục giây để render bản 720p (`HD_WAIT=240`, `HD_RETRY=4`).

## Cài đặt & Chạy trên Windows
1. Yêu cầu: Python 3.10+ và `ffmpeg` + `ffprobe` trong `PATH`.
2. Copy `.env.example` thành `.env` và điền:
   - `YT_CHANNEL_ID`: ID kênh YouTube (`UC...`)
   - `TIKTOK_CLIENT_KEY`, `TIKTOK_CLIENT_SECRET`, `TIKTOK_REFRESH_TOKEN`
   - Nếu chạy local chưa có domain HTTPS: đặt `WEBSUB_ENABLED=0` và `ENABLE_RSS_POLL=1`.
   - Nếu có domain / Cloudflare Tunnel (`https://...`): đặt `WEBSUB_ENABLED=1`, `PUBLIC_URL=https://...`, `WEBSUB_SECRET=...`.
3. Chạy `start.bat` (hoặc `python app.py`), mở trình duyệt tại `http://localhost:8080`.
"# -kien-video-sync" 
