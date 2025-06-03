# ai-therapy-assistant
import streamlit as st
from openai import OpenAI
import json
from datetime import datetime

from pydrive.auth import GoogleAuth
from pydrive.drive import GoogleDrive

# Cấu hình trang
st.set_page_config(page_title="AI Therapy Assistant", layout="centered")
st.title("AI.THERAPY.ASST")

client = OpenAI(api_key="YOUR_API_KEY")

CHAT_HISTORY_FILE = "chat_history.json"

# Google Drive Authentication
def authenticate_google_drive():
    gauth = GoogleAuth()
    gauth.LocalWebserverAuth()  # Mở trình duyệt để đăng nhập Google
    return GoogleDrive(gauth)

# Hàm lưu file JSON chat history
def save_chat_history(chat_history):
    history_with_time = []
    for chat in chat_history:
        history_with_time.append({
            "role": chat["role"],
            "content": chat["content"],
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        })
    with open(CHAT_HISTORY_FILE, "w", encoding="utf-8") as f:
        json.dump(history_with_time, f, ensure_ascii=False, indent=2)

# Hàm tải file lên Google Drive
def upload_to_google_drive(drive, filename):
    file_list = drive.ListFile({'q': f"title='{filename}' and trashed=false"}).GetList()
    if file_list:
        # Nếu file đã có, xóa để upload bản mới
        file_list[0].Delete()

    gfile = drive.CreateFile({'title': filename})
    gfile.SetContentFile(filename)
    gfile.Upload()
    return gfile['id']

# Khởi tạo session lưu hội thoại
if "chat_history" not in st.session_state:
    st.session_state.chat_history = []

# Hiển thị hội thoại cũ
for chat in st.session_state.chat_history:
    with st.chat_message(chat["role"]):
        st.markdown(chat["content"])

# Ô nhập liệu người dùng
user_input = st.chat_input("Bạn đang cảm thấy như thế nào hôm nay?")

if user_input:
    st.session_state.chat_history.append({"role": "user", "content": user_input})
    with st.chat_message("user"):
        st.markdown(user_input)

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=st.session_state.chat_history
    )

    ai_reply = response.choices[0].message.content

    st.session_state.chat_history.append({"role": "assistant", "content": ai_reply})
    with st.chat_message("assistant"):
        st.markdown(ai_reply)

    save_chat_history(st.session_state.chat_history)

    # Upload file chat_history.json lên Google Drive
    try:
        drive = authenticate_google_drive()
        file_id = upload_to_google_drive(drive, CHAT_HISTORY_FILE)
        st.success(f"📂 Đã lưu lịch sử chat lên Google Drive! File ID: {file_id}")
    except Exception as e:
        st.error(f"❌ Lỗi khi lưu lên Google Drive: {e}")
streamlit
openai
pydrive

