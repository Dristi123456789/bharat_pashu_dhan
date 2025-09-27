# bharat_pashu_dhan
#AI powered application to distinguish between various types of cattle and buffaloes breeds found in India
import streamlit as st
import time
from streamlit_lottie import st_lottie
import json
from PIL import Image

# animation ko load 
def load_lottie(file_path: str):
    with open(file_path, "r") as f:
        return json.load(f)

cow_animation = load_lottie("/Users/dristi/Downloads/cow.json")  

# page ka configuration
st.set_page_config(page_title="BHARAT PASUDHAN", layout="wide")

# screen animation
col1, col2 = st.columns([1, 2])  # left = cow, right = text

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
        time.sleep(0.2)

    st.subheader("Ab Bharat ke pasudhan ko dhundna hua aasaan! 🐄")

st.markdown("---")  

# colour
st.markdown("""
<style>
.stApp { background-color: #F5F5F5; }
.css-1d391kg { background-color: #FFFFFF; }
.user-message {
    background-color: #f0f0f0;
    padding: 10px;
    border-radius: 10px;
    margin-bottom: 5px;
}
.app-message {
    background-color: #d1ffd1;
    padding: 10px;
    border-radius: 10px;
    margin-bottom: 5px;
}
h1, h2, h3 { color: black; }
</style>
""", unsafe_allow_html=True)

# chat log
if 'chat_log' not in st.session_state:
    st.session_state.chat_log = []

# slidebar
with st.sidebar:
    st.header("Previous Chats")
    if st.session_state.chat_log:
        for entry in st.session_state.chat_log:
            st.markdown(entry, unsafe_allow_html=True)
    else:
        st.write("Here you will see your previous conversation ")

# main chat interface
st.header("Chat Interface")

# Upload image
uploaded_file = st.file_uploader("Upload the photo or capture from camera", type=["jpg","png","jpeg"])
if uploaded_file:
    image = Image.open(uploaded_file)
    st.image(image, caption="Uploaded Image", use_column_width=True)

    # Temporary breed detection
    detected_breed = st.selectbox("Detected Breed (temporary)", ["Sahiwal", "Red Sindhi"])
    result_text = f"<div class='app-message'>🐄 BHARAT PASUDHAN: Detected Breed - {detected_breed}</div>"
    st.session_state.chat_log.append(result_text)
    st.markdown(result_text, unsafe_allow_html=True)

# User input
user_question = st.text_input("Ask anything about your breed")
if user_question and uploaded_file:
    answer_text = f"<div class='app-message'>💬 BHARAT PASUDHAN: You asked about '{user_question}'. This Breed</div>"

    st.session_state.chat_log.append(f"<div class='user-message'>👤 Aap: {user_question}</div>")
    st.session_state.chat_log.append(answer_text)

    st.markdown(f"<div class='user-message'>👤 Aap: {user_question}</div>", unsafe_allow_html=True)
    st.markdown(answer_text, unsafe_allow_html=True)
