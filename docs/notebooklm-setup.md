# NotebookLM MCP server + Claude Code

Repo này tích hợp Google NotebookLM vào Claude Code qua **MCP server thật**:
package [`notebooklm-mcp`](https://github.com/khengyun/notebooklm-mcp)
(FastMCP v2, bản 2.0.11). Server được đăng ký sẵn trong `.mcp.json` ở gốc
repo — Claude Code tự nhận khi mở project.

> **Cách hoạt động:** server điều khiển Chrome thật (Selenium +
> undetected-chromedriver) để thao tác NotebookLM web UI, và lưu phiên
> đăng nhập Google trong thư mục `chrome_profile_notebooklm/` (đã gitignore).
> Vì vậy máy chạy server **cần có Chrome** và cần đăng nhập Google 1 lần.

## 1. Cài đặt (trên máy của bạn)

Khuyến nghị dùng virtualenv vì `undetected-chromedriver` cần setuptools mới
(setuptools bản Debian hệ thống sẽ lỗi `AttributeError: install_layout`):

```bash
python3 -m venv ~/.venvs/notebooklm-mcp
~/.venvs/notebooklm-mcp/bin/pip install -U pip setuptools wheel
~/.venvs/notebooklm-mcp/bin/pip install notebooklm-mcp

# đưa lệnh vào PATH để .mcp.json gọi được
ln -sf ~/.venvs/notebooklm-mcp/bin/notebooklm-mcp ~/.local/bin/notebooklm-mcp
```

(Hoặc đơn giản `pip install notebooklm-mcp` nếu môi trường Python của bạn
cho phép.)

## 2. Khởi tạo + đăng nhập Google (1 lần)

Chạy trong thư mục gốc repo, với URL notebook bạn muốn dùng mặc định:

```bash
notebooklm-mcp init https://notebooklm.google.com/notebook/<NOTEBOOK_ID>
```

Lệnh này tạo `notebooklm-config.json` + thư mục `chrome_profile_notebooklm/`,
mở Chrome cho bạn đăng nhập Google, rồi lưu phiên để các lần sau chạy
headless. Kiểm tra:

```bash
notebooklm-mcp --config notebooklm-config.json test
```

Cả 2 file/thư mục trên đã nằm trong `.gitignore` — **không commit** vì chứa
cookie/phiên đăng nhập Google của bạn.

## 3. Dùng trong Claude Code

`.mcp.json` của repo đăng ký server:

```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "notebooklm-mcp",
      "args": ["--config", "notebooklm-config.json", "server", "--transport", "stdio"]
    }
  }
}
```

Mở Claude Code trong repo, chấp thuận MCP server `notebooklm` khi được hỏi,
sau đó các tool chat/điều hướng NotebookLM sẽ xuất hiện (gõ `/mcp` để xem).

## 4. Máy headless / Claude Code trên web

Server **phải launch Chrome**, nên trên container remote cần:

1. Có Chrome/Chromium cài sẵn và mạng cho phép tải chromedriver
   (proxy chặn sẽ lỗi `Tunnel connection failed: 403`).
2. Không đăng nhập tương tác được — hãy đăng nhập trên máy local rồi
   chuyển profile sang:

```bash
# máy local (đã đăng nhập):
notebooklm-mcp export-profile -t notebooklm_profile_export
# copy thư mục đó sang máy remote, rồi:
notebooklm-mcp import-profile -f notebooklm_profile_export -t chrome_profile_notebooklm
```

Thực tế: tích hợp này phù hợp nhất khi chạy **Claude Code local** trên máy
có Chrome. Phiên remote không có Chrome + profile sẽ không khởi động được
server (đây là hạn chế của cơ chế browser-automation, không phải lỗi config).

## Ghi chú

- Đây là project không chính thức (không phải của Google); cookie phiên
  Google nằm trong `chrome_profile_notebooklm/` — giữ cẩn thận như mật khẩu.
- Trước đây repo dùng agent skill của `notebooklm-py`; đã gỡ và thay bằng
  MCP server này.
