import os
import requests
from flask import Flask, request, jsonify
from dotenv import load_dotenv

# تحميل المتغيرات البيئية من ملف .env
load_dotenv()

app = Flask(__name__)

# التوكن الخاص ببوت تلجرام ومعرف الشات من المتغيرات البيئية
TOKEN = os.getenv('TELEGRAM_TOKEN')
CHAT_ID = os.getenv('TELEGRAM_CHAT_ID')

# دالة لإرسال الصورة أو الفيديو إلى بوت تلجرام
def send_media_to_telegram(media_file, media_type):
    if media_type == 'photo':
        url = f"https://api.telegram.org/bot{TOKEN}/sendPhoto"
        files = {'photo': media_file}
    elif media_type == 'video':
        url = f"https://api.telegram.org/bot{TOKEN}/sendVideo"
        files = {'video': media_file}
    else:
        return "نوع الملف غير مدعوم"

    data = {'chat_id': CHAT_ID}
    response = requests.post(url, files=files, data=data)
    return response.status_code == 200

@app.route('/upload', methods=['POST'])
def upload():
    if 'file' not in request.files:
        return jsonify({"error": "لم يتم العثور على ملف"}), 400
    media_file = request.files['file']
    
    # تحديد نوع الوسائط (صورة أو فيديو) بناءً على امتداد الملف
    if media_file.filename.lower().endswith(('.png', '.jpg', '.jpeg')):
        media_type = 'photo'
    elif media_file.filename.lower().endswith(('.mp4', '.mov', '.avi')):
        media_type = 'video'
    else:
        return jsonify({"error": "نوع الملف غير مدعوم"}), 400

    # إرسال الوسائط إلى تلجرام
    if send_media_to_telegram(media_file, media_type):
        return jsonify({"message": "تم إرسال الوسائط بنجاح"}), 200
    else:
        return jsonify({"error": "فشل إرسال الوسائط"}), 500

@app.route('/')
def index():
    return "<h1>مرحبًا، يمكنك رفع الوسائط عبر هذا الرابط /upload</h1>"

if __name__ == '__main__':
    app.run(debug=False, host='0.0.0.0', port=5000)
