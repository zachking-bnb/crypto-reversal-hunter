# NotebookLM + Claude Code setup

Tích hợp Google NotebookLM vào Claude Code cho repo này dùng package
[`notebooklm-py`](https://github.com/teng-lin/notebooklm-py) (bản 0.7.3).

> **Lưu ý:** các lệnh `pip install "notebooklm-py[mcp]"` và
> `notebooklm mcp install claude-code` **không tồn tại** trong package này —
> không có extra `[mcp]` và không có lệnh `mcp`. Cách tích hợp chính thức là
> **agent skill** (`notebooklm skill install`), đã được cài sẵn vào repo tại
> `.claude/skills/notebooklm/SKILL.md` (Claude Code) và
> `.agents/skills/notebooklm/SKILL.md` (universal agent skills).

## 1. Cài đặt (trên máy của bạn)

```bash
pip install "notebooklm-py[browser]"
```

Extra `[browser]` kéo theo Playwright để chạy `notebooklm login`.
Nếu muốn đọc cookie từ trình duyệt đang đăng nhập sẵn (không cần mở
cửa sổ login), cài thêm `notebooklm-py[cookies]` (chỉ hỗ trợ Python < 3.13).

## 2. Đăng nhập Google (1 lần, cần máy có trình duyệt)

```bash
notebooklm login
# hoặc lấy cookie từ Chrome đang đăng nhập sẵn:
notebooklm login --browser-cookies chrome
```

Phiên đăng nhập được lưu tại `~/.notebooklm/profiles/default/storage_state.json`.
Kiểm tra bằng:

```bash
notebooklm auth check --test
```

## 3. Skill cho Claude Code

Skill đã được commit vào repo nên Claude Code tự nhận khi mở project.
Để cập nhật lên bản mới sau khi nâng cấp package:

```bash
notebooklm skill install --scope project --force
```

Sau đó trong Claude Code chỉ cần gõ `/notebooklm` hoặc yêu cầu tự nhiên
("tạo podcast từ tài liệu X") — skill hướng dẫn Claude gọi CLI `notebooklm`.

## 4. Dùng trong môi trường headless / Claude Code trên web

Container remote không mở được cửa sổ login Google. Có 2 cách:

1. **Copy file auth**: đăng nhập trên máy local rồi copy
   `storage_state.json` lên đường dẫn
   `~/.notebooklm/profiles/default/storage_state.json` của môi trường remote.
2. **Biến môi trường** (fast path): đặt `NOTEBOOKLM_AUTH_JSON` chứa nguyên
   nội dung JSON của `storage_state.json` (ví dụ qua environment settings
   của Claude Code trên web). Khi biến này tồn tại, CLI bỏ qua file và
   dùng luôn cookie trong biến.

Cookie Google hết hạn theo thời gian — nếu `notebooklm auth check --test`
báo lỗi, đăng nhập lại trên máy local và cập nhật lại auth.
