# 🤖 DEVIN BROWSER AGENT: Windows Autonomous Browser Engine
> **Purpose:** Drop this single file into any AI agent (Antigravity, Claude, ChatGPT, Cursor, etc.) to give it the ability to launch a visible Chrome window in front of the user and browse autonomously at Devin.ai speed with zero blindness and zero missed clicks.

---

## 🧠 The Windows Architecture Secret (Read First)
By default, AI agent subprocesses on Windows run inside an isolated virtual sandbox desktop (e.g. `exebox-...` on `WinSta0`). If an AI tries to launch Chrome or move the mouse normally, **the user sees nothing**.

To make automation visible on the user's physical screen, the AI **must always**:
1. Attach its thread to `WinSta0\Default` using `ctypes.windll.user32.SetThreadDesktop`.
2. Launch Google Chrome using `kernel32.CreateProcessW` with `STARTUPINFOW.lpDesktop = 'WinSta0\\Default'`.
3. Enable Chrome DevTools Protocol (`--remote-debugging-port=9222`) and a dedicated `--user-data-dir`.
4. Connect Playwright over CDP (`p.chromium.connect_over_cdp("http://localhost:9222")`).

---

## 📦 1-Step Environment Setup
Run this single command in terminal if dependencies are missing:
```bash
pip install playwright pygetwindow pyautogui
```

---

## 🚀 The Self-Contained Automation Engine (`agent_browser.py`)
Whenever the user asks you to automate the browser, generate and run a Python script following this exact template:

```python
import time
import ctypes
from ctypes import wintypes
from playwright.sync_api import sync_playwright

user32 = ctypes.windll.user32
kernel32 = ctypes.windll.kernel32

# ==============================================================================
# 1. ATTACH TO USER'S PHYSICAL MONITOR (WinSta0\Default)
# ==============================================================================
GENERIC_ALL = 0x10000000
h_default = user32.OpenDesktopW('Default', 0, False, GENERIC_ALL)
if h_default:
    user32.SetThreadDesktop(h_default)

# ==============================================================================
# 2. LAUNCH CHROME ON PHYSICAL SCREEN WITH CDP ENABLED
# ==============================================================================
class STARTUPINFOW(ctypes.Structure):
    _fields_ = [
        ('cb', wintypes.DWORD),
        ('lpReserved', wintypes.LPWSTR),
        ('lpDesktop', wintypes.LPWSTR),
        ('lpTitle', wintypes.LPWSTR),
        ('dwX', wintypes.DWORD),
        ('dwY', wintypes.DWORD),
        ('dwXSize', wintypes.DWORD),
        ('dwYSize', wintypes.DWORD),
        ('dwXCountChars', wintypes.DWORD),
        ('dwYCountChars', wintypes.DWORD),
        ('dwFillAttribute', wintypes.DWORD),
        ('dwFlags', wintypes.DWORD),
        ('wShowWindow', wintypes.WORD),
        ('cbReserved2', wintypes.WORD),
        ('lpReserved2', ctypes.c_void_p),
        ('hStdInput', wintypes.HANDLE),
        ('hStdOutput', wintypes.HANDLE),
        ('hStdError', wintypes.HANDLE),
    ]

class PROCESS_INFORMATION(ctypes.Structure):
    _fields_ = [
        ('hProcess', wintypes.HANDLE),
        ('hThread', wintypes.HANDLE),
        ('dwProcessId', wintypes.DWORD),
        ('dwThreadId', wintypes.DWORD),
    ]

si = STARTUPINFOW()
si.cb = ctypes.sizeof(STARTUPINFOW)
si.lpDesktop = 'WinSta0\\Default'   # Forces window onto visible monitor
si.dwFlags = 1                       # STARTF_USESHOWWINDOW
si.wShowWindow = 3                   # SW_SHOWMAXIMIZED
pi = PROCESS_INFORMATION()

user_data = r"C:\Users\espacegamers.ma\AppData\Local\Temp\ai_chrome_profile"
chrome_path = r"C:\Program Files\Google\Chrome\Application\chrome.exe"
target_url = "https://www.google.com"

chrome_cmd = f'"{chrome_path}" --remote-debugging-port=9222 --user-data-dir="{user_data}" --no-first-run --no-default-browser-check --start-maximized {target_url}'

kernel32.CreateProcessW(None, chrome_cmd, None, None, False, 0, None, None, ctypes.byref(si), ctypes.byref(pi))
user32.MessageBeep(0) # Audio chime to alert user
time.sleep(3.0)

# ==============================================================================
# 3. HUD BADGE & NEON ELEMENT HIGHLIGHTER
# ==============================================================================
def update_hud(page, status_text, highlight_selector=None):
    js = f"""
    (() => {{
        let hud = document.getElementById('ai-agent-hud');
        if (!hud) {{
            hud = document.createElement('div');
            hud.id = 'ai-agent-hud';
            hud.style.position = 'fixed';
            hud.style.top = '20px';
            hud.style.right = '24px';
            hud.style.zIndex = '2147483647';
            hud.style.background = 'rgba(15, 23, 42, 0.95)';
            hud.style.color = '#38bdf8';
            hud.style.padding = '12px 24px';
            hud.style.borderRadius = '16px';
            hud.style.boxShadow = '0 10px 40px rgba(0, 0, 0, 0.6), 0 0 20px rgba(56, 189, 248, 0.4)';
            hud.style.fontFamily = '-apple-system, sans-serif';
            hud.style.fontSize = '14px';
            hud.style.fontWeight = '600';
            hud.style.border = '2px solid #38bdf8';
            hud.style.display = 'flex';
            hud.style.alignItems = 'center';
            hud.style.gap = '10px';
            document.body.appendChild(hud);
        }}
        hud.innerHTML = `<span style="display:inline-block;width:10px;height:10px;border-radius:50%;background:#22c55e;box-shadow:0 0 10px #22c55e;"></span> 🤖 <strong>AI Agent</strong> | <span style="color:#ffffff;">{status_text}</span>`;

        if ('{highlight_selector or ""}') {{
            let el = document.querySelector('{highlight_selector or ""}');
            if (el) {{
                try {{ el.scrollIntoView({{ behavior: 'smooth', block: 'center' }}); }} catch(e) {{}}
                el.style.outline = '4px solid #f43f5e';
                el.style.outlineOffset = '4px';
                el.style.boxShadow = '0 0 25px #f43f5e';
                el.style.transition = 'all 0.3s ease';
            }}
        }}
    }})();
    """
    try: page.evaluate(js)
    except: pass

# ==============================================================================
# 4. CDP-CONNECTED AUTONOMOUS BROWSING
# ==============================================================================
with sync_playwright() as p:
    browser = p.chromium.connect_over_cdp("http://localhost:9222")
    context = browser.contexts[0]
    page = context.pages[0] if context.pages else context.new_page()

    # --- YOUR CUSTOM ACTIONS GO HERE ---
    update_hud(page, "Searching Google...", "textarea[name='q'], input[name='q']")
    time.sleep(1.0)

    # Locate and type with human cadence
    search_bar = page.locator("textarea[name='q'], input[name='q']").first
    search_bar.click()
    search_bar.type("Autonomous AI Agent", delay=40)
    search_bar.press("Enter")
    page.wait_for_load_state("domcontentloaded")
    time.sleep(1.5)

    update_hud(page, "Scanning search results...")
    page.evaluate("window.scrollBy({ top: 500, behavior: 'smooth' })")
    time.sleep(2.0)

    # Wrap up
    update_hud(page, "Mission Complete!")
    user32.MessageBeep(0)
    time.sleep(2.0)
```

---

## 📋 Rules for Any AI Agent Running this Engine
1. **Never guess blind coordinates:** Always use Playwright locators (`page.locator(...)`, `text=...`, CSS selectors, or accessibility roles).
2. **Never hide the browser:** Always use `WinSta0\Default` and `SW_SHOWMAXIMIZED` so the user watches every action in real time.
3. **Use `force=True` on tricky clicks:** If a link or button is partially obstructed by a popup, use `locator.click(force=True, timeout=3000)`.
4. **Prevent Console Unicode Errors:** On Windows `cp1252`, avoid printing complex emojis directly to `print()`; use standard ASCII status tags like `[SUCCESS]`, `[INFO]`, or `[ERROR]`.
5. **Keep user control intact:** The browser runs visibly in front of the user without locking their physical mouse cursor, allowing the user to inspect and take over at any moment.

---

## 💻 Multi-App Desktop Automation (Beyond the Browser)
To control native Windows desktop applications (Notepad, Excel, File Explorer, etc.) alongside the browser:

### 1. Launch any Desktop App Visually on Screen:
```python
# Launch Notepad or any desktop software on WinSta0\Default:
launch_on_screen(r'notepad.exe "C:\path\to\file.txt"')

# Open Windows File Explorer and automatically highlight a specific file:
launch_on_screen(r'explorer.exe /select,"C:\path\to\file.txt"')
```

### 2. Download Files from the Web:
```python
# In Playwright, intercept downloads automatically:
with page.expect_download() as download_info:
    page.click("button:has-text('Download')")
download = download_info.value
download.save_as(r"C:\Users\espacegamers.ma\Downloads\report.csv")
```

### 3. Open Any File in Its Default Windows Program:
```python
import os
os.startfile(r"C:\Users\espacegamers.ma\Downloads\report.csv")
```

