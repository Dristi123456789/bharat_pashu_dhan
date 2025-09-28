import streamlit as st
import time
from streamlit_lottie import st_lottie
import json
from PIL import Image
import io

# Import gif
def load_lottie(file_path: str):
    with open(file_path, "r") as f:
        return json.load(f)

cow_animation = load_lottie("/Users/dristi/Downloads/cow.json")  

# page configuration
st.set_page_config(page_title="BHARAT PASUDHAN", layout="wide")

# load configuration
col1, col2 = st.columns([1, 2])  

with col1:
    st_lottie(cow_animation, height=250, key="cow_splash")

with col2:
    placeholder = st.empty()
    text = "BHARAT PASUDHAN"
    display_text = ""
    for letter in text:
        display_text += letter
        placeholder.markdown(
            f"<h1 style='font-size:60px; color:black; font-family:Times New Roman;'>{display_text}</h1>",
            unsafe_allow_html=True
        )
        time.sleep(0.15)
    st.subheader("Ab Bharat ke pasudhan ko dhundna hua aasaan! 🐄")

st.markdown("---")  

# colours
st.markdown("""
<style>
.stApp { background-color: #F5F5F5; }
.css-1d391kg { background-color: #FFFFFF; }
.user-message { background-color: #f0f0f0; padding: 10px; border-radius: 10px; margin-bottom: 5px; }
.app-message { background-color: #d1ffd1; padding: 10px; border-radius: 10px; margin-bottom: 5px; }
h1, h2, h3 { color: black; }
</style>
""", unsafe_allow_html=True)

# session state
if "chat_sessions" not in st.session_state:
    st.session_state.chat_sessions = []
if "current_session" not in st.session_state:
    st.session_state.current_session = None

# slidebar
with st.sidebar:
    st.header("Previous Chats")
    
    # Start a new chat session
    if st.button("➕ Start New Chat"):
        new_session = {"title": "New Chat", "messages": []}
        st.session_state.chat_sessions.append(new_session)
        st.session_state.current_session = len(st.session_state.chat_sessions) - 1

    # List previous sessions
    for i, session in enumerate(st.session_state.chat_sessions):
        display_title = session["title"] if session["title"] != "New Chat" else f"Chat {i+1}"
        if st.button(display_title, key=f"chat_{i}"):
            st.session_state.current_session = i

# main interface
st.title("🐄 BHARAT PASUDHAN")
st.subheader("Ab Bharat ke pasudhan ko dhundna hua aasaan!")

current = st.session_state.current_session

if current is not None:
    session = st.session_state.chat_sessions[current]

    # Display existing messages
    if session["messages"]:
        st.markdown("### Chat History")
        for msg in session["messages"]:
            if msg["role"] == "user":
                if "image" in msg:
                    # Recreate Image 
                    image = Image.open(io.BytesIO(msg["image"]))
                    st.image(image, caption="Uploaded Image", use_column_width=True)
                else:
                    st.markdown(f"<div class='user-message'>👤 Aap: {msg['content']}</div>", unsafe_allow_html=True)
            else:
                st.markdown(f"<div class='app-message'>{msg['content']}</div>", unsafe_allow_html=True)

    # image upload
    uploaded_file = st.file_uploader("Upload photo or capture from camera", type=["jpg","png","jpeg"], key=f"file_{current}")
    if uploaded_file:
        image_bytes = uploaded_file.read()
        image = Image.open(io.BytesIO(image_bytes))
        st.image(image, caption="Uploaded Image", use_column_width=True)

        detected_breed = st.selectbox("Detected Breed (temporary)", ["Sahiwal", "Red Sindhi"], key=f"breed_{current}")
        result_text = f"🐄 BHARAT PASUDHAN: Detected Breed - {detected_breed}"

        # Add image and result to session
        session["messages"].append({"role": "user", "content": "Uploaded an image", "image": image_bytes})
        session["messages"].append({"role": "app", "content": result_text})
        session["title"] = f"Breed - {detected_breed}"

        st.markdown(f"<div class='app-message'>{result_text}</div>", unsafe_allow_html=True)

    # user question
    user_question = st.text_input("Ask anything about your breed", key=f"question_{current}")
    if user_question:
        answer_text = f"💬 BHARAT PASUDHAN: You asked about '{user_question}'. This Breed."
        session["messages"].append({"role": "user", "content": user_question})
        session["messages"].append({"role": "app", "content": answer_text})

        st.markdown(f"<div class='user-message'>👤 Aap: {user_question}</div>", unsafe_allow_html=True)
        st.markdown(f"<div class='app-message'>{answer_text}</div>", unsafe_allow_html=True)

# if no session
if current is None:
    st.write("Click ➕ 'Start New Chat' to begin a new session.")
