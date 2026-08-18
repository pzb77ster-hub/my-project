# remote_desktop.py 파일 내용
code_content = """import io
from flask import Flask, Response, render_template_string, request
import pyautogui

app = Flask(__name__)
pyautogui.FAILSAFE = False

HTML_TEMPLATE = \"\"\"
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Mobile Remote Desktop</title>
    <style>
        body { margin: 0; background: #000; text-align: center; overflow: hidden; }
        img { width: 100vw; height: auto; touch-action: none; cursor: crosshair; }
    </style>
</head>
<body>
    <img id="screen" src="/stream">
    <script>
        const img = document.getElementById('screen');
        img.addEventListener('click', (e) => {
            const rect = img.getBoundingClientRect();
            const x = (e.clientX - rect.left) / rect.width;
            const y = (e.clientY - rect.top) / rect.height;
            fetch(`/click?x=${x}&y=${y}`);
        });
    </script>
</body>
</html>
\"\"\"

def gen_frames():
    while True:
        screenshot = pyautogui.screenshot()
        screenshot = screenshot.resize((1280, 720))
        buffer = io.BytesIO()
        screenshot.save(buffer, format="JPEG", quality=40)
        frame = buffer.getvalue()
        yield (b'--frame\\r\\n'
               b'Content-Type: image/jpeg\\r\\n\\r\\n' + frame + b'\\r\\n')

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

@app.route('/stream')
def stream():
    return Response(gen_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

@app.route('/click')
def click():
    x_pct = float(request.args.get('x', 0))
    y_pct = float(request.args.get('y', 0))
    sw, sh = pyautogui.size()
    pyautogui.click(int(x_pct * sw), int(y_pct * sh))
    return '', 204

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, threaded=True)
"""

with open("remote_desktop.py", "w", encoding="utf-8") as f:
  f.write(code_content)
