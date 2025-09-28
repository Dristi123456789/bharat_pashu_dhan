import streamlit as st
import time
from streamlit_lottie import st_lottie
import json
from PIL import Image
import io
from gtts import gTTS
import os
from tempfile import NamedTemporaryFile

# load animation
def load_lottie(file_path: str):
    with open(file_path, "r") as f:
        return json.load(f)

cow_animation = load_lottie("/Users/dristi/Downloads/cow.json")

# page configuration
st.set_page_config(page_title="BHARAT PASUDHAN", layout="wide")

# session state
if "phase" not in st.session_state:
    st.session_state.phase = 1
if "language" not in st.session_state:
    st.session_state.language = "English"
if "chat_sessions" not in st.session_state:
    st.session_state.chat_sessions = []
if "current_session" not in st.session_state:
    st.session_state.current_session = None

# voice function
def speak_text(text, lang_code='en'):
    tts = gTTS(text=text, lang=lang_code)
    with NamedTemporaryFile(delete=True) as fp:
        temp_file = fp.name + ".mp3"
        tts.save(temp_file)
        os.system(f"afplay '{temp_file}'")  # MacOS playback

# show animation
if st.session_state.phase == 1:
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

    time.sleep(1)
    st.session_state.phase = 2
    st.rerun()

# language select
elif st.session_state.phase == 2:
    st.markdown("<h2 style='text-align:center;'>Choose Your Language</h2>", unsafe_allow_html=True)
    st.markdown("<h3 style='text-align:center;'>अपनी भाषा चुनें</h3>", unsafe_allow_html=True)

    col1, col2, col3 = st.columns([1, 2, 1])
    with col2:
        language = st.radio("", ["English", "हिन्दी"], index=0)

    if st.button("Continue ➡️"):
        st.session_state.language = language
        st.session_state.phase = 3
        st.rerun()

# main app
elif st.session_state.phase == 3:
    lang = st.session_state.language

    # Sidebar
    with st.sidebar:
        if lang == "English":
            st.header("Previous Chats")
            if st.button("➕ Start New Chat"):
                new_session = {"title": "New Chat", "messages": []}
                st.session_state.chat_sessions.append(new_session)
                st.session_state.current_session = len(st.session_state.chat_sessions) - 1
            for i, session in enumerate(st.session_state.chat_sessions):
                display_title = session["title"] if session["title"] != "New Chat" else f"Chat {i+1}"
                if st.button(display_title, key=f"chat_{i}"):
                    st.session_state.current_session = i
        else:
            st.header("पिछली चैट")
            if st.button("➕ नई चैट शुरू करें"):
                new_session = {"title": "नई चैट", "messages": []}
                st.session_state.chat_sessions.append(new_session)
                st.session_state.current_session = len(st.session_state.chat_sessions) - 1
            for i, session in enumerate(st.session_state.chat_sessions):
                display_title = session["title"] if session["title"] != "नई चैट" else f"चैट {i+1}"
                if st.button(display_title, key=f"chat_{i}"):
                    st.session_state.current_session = i

    # Main Interface
    if lang == "English":
        st.title("🐄 BHARAT PASUDHAN")
        st.subheader("Ab Bharat ke pasudhan ko dhundna hua aasaan!")
    else:
        st.title("🐄 भारत पशुधन")
        st.subheader("अब भारत के पशुधन को ढूँढना हुआ आसान!")

    current = st.session_state.current_session

    if current is not None:
        session = st.session_state.chat_sessions[current]

        # Display existing messages
        if session["messages"]:
            if lang == "English":
                st.markdown("### Chat History")
            else:
                st.markdown("### चैट इतिहास")

            for msg in session["messages"]:
                if msg["role"] == "user":
                    if "image" in msg:
                        image = Image.open(io.BytesIO(msg["image"]))
                        st.image(image, caption="Uploaded Image" if lang=="English" else "अपलोड की गई फोटो", use_column_width=True)
                    else:
                        st.markdown(
                            f"<div style='background:#f0f0f0; padding:10px; border-radius:10px;'>"
                            f"👤 {'You' if lang=='English' else 'आप'}: {msg['content']}</div>",
                            unsafe_allow_html=True
                        )
                else:
                    st.markdown(
                        f"<div style='background:#d1ffd1; padding:10px; border-radius:10px;'>"
                        f"{msg['content']}</div>",
                        unsafe_allow_html=True
                    )

        # Upload image
        upload_label = "Upload photo or capture from camera" if lang=="English" else "फोटो अपलोड करें या कैमरे से कैप्चर करें"
        uploaded_file = st.file_uploader(upload_label, type=["jpg", "png", "jpeg"], key=f"file_{current}")
        if uploaded_file:
            image_bytes = uploaded_file.read()
            image = Image.open(io.BytesIO(image_bytes))
            st.image(image, caption="Uploaded Image" if lang=="English" else "अपलोड की गई फोटो", use_column_width=True)

            breed_label = "Detected Breed (temporary)" if lang=="English" else "पशु की नस्ल (अस्थायी)"
            detected_breed = st.selectbox(
                breed_label,
                ["Sahiwal", "Red Sindhi"] if lang=="English" else ["साहीवाल", "रेड सिंधी"],
                key=f"breed_{current}"
            )

            result_text = f"🐄 BHARAT PASUDHAN: Detected Breed - {detected_breed}" if lang=="English" else f"🐄 भारत पशुधन: पहचानी गई नस्ल - {detected_breed}"

            session["messages"].append({"role": "user", "content": "Uploaded an image" if lang=="English" else "एक फोटो अपलोड की", "image": image_bytes})
            session["messages"].append({"role": "app", "content": result_text})
            session["title"] = f"Breed - {detected_breed}" if lang=="English" else f"नस्ल - {detected_breed}"

            st.success(result_text)

        # Ask a question
        q_label = "Ask anything about your breed" if lang=="English" else "अपनी नस्ल के बारे में कुछ भी पूछें"
        user_question = st.text_input(q_label, key=f"question_{current}")
        answer_text = ""
        if user_question:
            # Plain text for voice 
            answer_text = f"You asked about '{user_question}'." if lang=="English" else f"आपने पूछा '{user_question}'."
            # Add to chat history visually
            session["messages"].append({"role": "user", "content": user_question})
            session["messages"].append({"role": "app", "content": answer_text})
            st.info(answer_text)

        # Voice button below text input
        voice_label = "🔊 Listen" if lang=="English" else "🔊 सुनें"
        if st.button(voice_label):
            speak_text(answer_text, lang_code='en' if lang=="English" else 'hi')

    if current is None:
        st.write("Click ➕ 'Start New Chat' to begin a new session." if lang=="English" else "नई सत्र शुरू करने के लिए ➕ 'नई चैट शुरू करें' पर क्लिक करें।")
