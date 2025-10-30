import tkinter as tk
import random
import time
import threading
from flask import Flask, render_template_string

# ---------------------- PHẦN GIAO DIỆN CẢNH BÁO ----------------------
MESSAGES = [
    "Your computer has been infected!",
    "Task Manager has been disabled by your administrator.",
    "Critical Error: System files corrupted.",
    "Virus detected: quarantine recommended.",
    "Unauthorized access detected.",
    "System stability at risk. Save your work now.",
    "Memory leak detected in process explorer.exe.",
    "Security breach: user privileges revoked.",
    "Fatal exception: reboot required.",
    "Warning: suspicious network activity."
]

TITLES = [
    "Virus Warning",
    "System Error",
    "Security Alert",
    "Warning",
    "Critical"
]

def create_alert(root, x, y, title, message, width=380, height=130):
    w = tk.Toplevel(root)
    w.overrideredirect(True)
    w.attributes("-topmost", True)
    outer = tk.Frame(w, bg="#333333", bd=2)
    outer.pack(fill="both", expand=True)

    titlebar = tk.Frame(outer, bg="#0b62a4", height=28)
    titlebar.pack(fill="x")
    title_label = tk.Label(titlebar, text=title, fg="white", bg="#0b62a4",
                           font=("Segoe UI", 10, "bold"), anchor="w", padx=8)
    title_label.pack(side="left", fill="y", expand=True)

    def close_click():
        fade_out_and_destroy(w)

    close_btn = tk.Label(titlebar, text="✕", fg="white", bg="#0b62a4",
                         font=("Segoe UI", 10, "bold"), width=3)
    close_btn.pack(side="right")
    close_btn.bind("<Button-1>", lambda e: close_click())

    body = tk.Frame(outer, bg="#f0f0f0")
    body.pack(fill="both", expand=True)

    left = tk.Frame(body, bg="#f0f0f0", width=80)
    left.pack(side="left", fill="y", padx=10, pady=12)
    icon_lbl = tk.Label(left, text="💀", font=("Segoe UI Emoji", 28), bg="#f0f0f0")
    icon_lbl.pack()

    right = tk.Frame(body, bg="#f0f0f0")
    right.pack(side="left", fill="both", expand=True, padx=(0,10), pady=12)
    msg_lbl = tk.Label(right, text=message, justify="left", anchor="nw",
                       bg="#f0f0f0", font=("Segoe UI", 10))
    msg_lbl.pack(fill="both", expand=True)

    btn_frame = tk.Frame(outer, bg="#d9d9d9")
    btn_frame.pack(fill="x")
    ok_btn = tk.Button(btn_frame, text="OK", width=10, command=close_click,
                       bg="#f3f3f3", relief="raised")
    ok_btn.pack(side="right", padx=12, pady=8)

    w.geometry(f"{width}x{height}+{x}+{y}")

    def start_move(evt):
        w._drag_start_x = evt.x
        w._drag_start_y = evt.y

    def do_move(evt):
        nx = w.winfo_x() + evt.x - w._drag_start_x
        ny = w.winfo_y() + evt.y - w._drag_start_y
        w.geometry(f"+{nx}+{ny}")

    titlebar.bind("<Button-1>", start_move)
    titlebar.bind("<B1-Motion>", do_move)
    title_label.bind("<Button-1>", start_move)
    title_label.bind("<B1-Motion>", do_move)

    try:
        w.attributes("-alpha", 0.0)
        fade_in(w)
    except Exception:
        pass

    return w

def fade_in(win, step=0.08, delay=15):
    try:
        a = win.attributes("-alpha")
    except Exception:
        return
    a = a + step
    if a >= 1.0:
        win.attributes("-alpha", 1.0)
        return
    win.attributes("-alpha", a)
    win.after(delay, lambda: fade_in(win, step, delay))

def fade_out_and_destroy(win, step=0.06, delay=15):
    try:
        a = win.attributes("-alpha")
    except Exception:
        win.destroy()
        return
    a = a - step
    if a <= 0.0:
        win.destroy()
        return
    win.attributes("-alpha", a)
    win.after(delay, lambda: fade_out_and_destroy(win, step, delay))

def spawn_alerts(root, count=6, area_padding=120):
    windows = []
    screen_w = root.winfo_screenwidth()
    screen_h = root.winfo_screenheight()
    for _ in range(count):
        w = random.randint(320, 460)
        h = random.randint(110, 160)
        x = random.randint(area_padding, max(area_padding, screen_w - w - area_padding))
        y = random.randint(area_padding, max(area_padding, screen_h - h - area_padding))
        title = random.choice(TITLES)
        msg = random.choice(MESSAGES)
        win = create_alert(root, x, y, title, msg, width=w, height=h)
        windows.append(win)
        root.update()
        time.sleep(0.06)
    return windows

def occasional_shake(windows, interval=(1200, 3000), magnitude=8):
    def loop():
        while True:
            time.sleep(random.uniform(interval[0]/1000.0, interval[1]/1000.0))
            if not windows:
                continue
            win = random.choice(list(windows))
            try:
                ox = win.winfo_x()
                oy = win.winfo_y()
            except Exception:
                continue
            for i in range(6):
                dx = random.randint(-magnitude, magnitude)
                dy = random.randint(-magnitude, magnitude)
                try:
                    win.geometry(f"+{ox+dx}+{oy+dy}")
                    time.sleep(0.02)
                except Exception:
                    break
            try:
                win.geometry(f"+{ox}+{oy}")
            except Exception:
                pass

    t = threading.Thread(target=loop, daemon=True)
    t.start()
    return t

def start_tkinter_alerts():
    root = tk.Tk()
    root.attributes("-topmost", True)
    root.configure(bg="black")
    root.geometry("1x1+0+0")
    root.bind("<Escape>", lambda e: root.destroy())

    windows = spawn_alerts(root, count=7)
    occasional_shake(windows)

    def periodic_spawn():
        while True:
            time.sleep(random.uniform(3.5, 7.5))
            try:
                new = spawn_alerts(root, count=random.randint(1, 3))
                windows.extend(new)
            except Exception:
                pass

    threading.Thread(target=periodic_spawn, daemon=True).start()
    root.mainloop()

# ---------------------- PHẦN WEB ----------------------
app = Flask(__name__)

HTML_PAGE = """
<!DOCTYPE html>
<html>
<head>
    <title>Trò chơi khắm</title>
    <style>
        body { font-family: Arial; text-align: center; background: #fafafa; padding-top: 100px; }
        button {
            background-color: red; color: white; border: none; padding: 15px 30px;
            font-size: 18px; border-radius: 10px; cursor: pointer; transition: 0.3s;
        }
        button:hover { background-color: darkred; }
    </style>
</head>
<body>
    <h2>Đây là trò chơi khắm 😈</h2>
    <p>Không ép bạn phải nhấn nút... Nhưng nếu bạn nhấn, sẽ có chuyện xảy ra 🤡</p>
    <form action="/start" method="post">
        <button type="submit">Nhấn nếu bạn dám</button>
    </form>
    <p3>Nếu lỡ ấn thì chỉ có cách reset lại máy tính mới gỡ được :)))</p>
</body>
</html>
"""

@app.route("/")
def index():
    return render_template_string(HTML_PAGE)

@app.route("/start", methods=["POST"])
def start():
    threading.Thread(target=start_tkinter_alerts, daemon=True).start()
    return "<h1>Haha 😈 trò chơi đã bắt đầu! Bạn có thể đóng trang web.</h1>"

if __name__ == "__main__":
    print("Mở trình duyệt tại: http://127.0.0.1:5000")
    app.run(port=5000, debug=False)
