import streamlit as st
from google import genai
from google.genai import types
from PIL import Image
from gtts import gTTS
from io import BytesIO
import os

# ---------------------------------
# PAGE CONFIGURATION
# ---------------------------------

st.set_page_config(
    page_title="AI Plant Doctor",
    page_icon="🌱",
    layout="centered"
)

st.title("🌱 AI Plant Doctor")
st.subheader("Plant Disease Detection and Treatment System")

st.write(
    "Scan a plant leaf or upload a photo "
    "to identify possible diseases and get treatment."
)

# ---------------------------------
# API CONFIGURATION
# ---------------------------------

API_KEY = st.secrets.get(
    "GEMINI_API_KEY",
    os.getenv("GEMINI_API_KEY")
)

if not API_KEY:
    st.error("Please configure your Gemini API key.")
    st.stop()

client = genai.Client(api_key=API_KEY)

# ---------------------------------
# INDIAN LANGUAGE SELECTION
# ---------------------------------

languages = {
    "English": ("English", "en"),
    "Hindi": ("Hindi", "hi"),
    "Kannada": ("Kannada", "kn"),
    "Telugu": ("Telugu", "te"),
    "Tamil": ("Tamil", "ta"),
    "Malayalam": ("Malayalam", "ml"),
    "Bengali": ("Bengali", "bn"),
    "Marathi": ("Marathi", "mr"),
    "Gujarati": ("Gujarati", "gu"),
    "Punjabi": ("Punjabi", "pa")
}

selected_language = st.selectbox(
    "🌐 Select your language",
    list(languages.keys())
)

language_name, language_code = languages[selected_language]

# ---------------------------------
# CAMERA AND IMAGE UPLOAD
# ---------------------------------

st.header("📸 Scan Your Plant")

camera_photo = st.camera_input(
    "Take a picture of the plant leaf"
)

uploaded_photo = st.file_uploader(
    "Or upload a leaf photo from your gallery",
    type=["jpg", "jpeg", "png"]
)

image_file = camera_photo if camera_photo else uploaded_photo

# ---------------------------------
# IMAGE PREVIEW
# ---------------------------------

if image_file is not None:

    image_bytes = image_file.getvalue()
    image = Image.open(BytesIO(image_bytes)).convert("RGB")

    st.image(
        image,
        caption="Your scanned plant leaf",
        use_container_width=True
    )

    # ---------------------------------
    # DISEASE DETECTION
    # ---------------------------------

    if st.button("🔍 Detect Plant Disease", type="primary"):

        with st.spinner("AI is examining your plant..."):

            try:

                prompt = f"""
                You are an agricultural plant disease assistant.

                Examine the uploaded leaf photograph carefully.

                Respond in {language_name}.

                Provide the following information:

                1. Plant name.
                2. Whether the leaf appears healthy or diseased.
                3. Most likely disease, if visible.
                4. Confidence level and uncertainty.
                5. Visible symptoms.
                6. Possible causes.
                7. Immediate treatment.
                8. Organic and natural remedies.
                9. Suitable chemical treatment, if appropriate.
                10. Prevention methods.
                11. Watering and plant care recommendations.

                Important rules:

                - If the image is not a plant leaf, say so.
                - If the image is unclear, request another photo.
                - Do not invent a disease.
                - Distinguish visual symptoms from confirmed diagnosis.
                - Do not recommend dangerous pesticide mixtures.
                - Mention that pesticide labels and local agricultural
                  regulations must be followed.
                - If uncertain, recommend contacting an agricultural
                  expert or local KVK.
                - Use simple language that farmers can understand.
                - Give practical and clear instructions.
                """

                response = client.models.generate_content(
                    model="gemini-3.8-flash",
                    contents=[
                        prompt,
                        types.Part.from_bytes(
                            data=image_bytes,
                            mime_type=image_file.type
                        )
                    ],
                    config=types.GenerateContentConfig(
                        tools=[{"google_search": {}}]
                    )
                )

                result = response.text

                if not result:
                    st.error("AI could not generate a diagnosis.")
                    st.stop()

                # Save result for voice playback
                st.session_state["plant_result"] = result
                st.session_state["plant_language"] = language_code

            except Exception as e:
                st.error(f"Detection error: {e}")

    # ---------------------------------
    # DISPLAY RESULT
    # ---------------------------------

    if "plant_result" in st.session_state:

        result = st.session_state["plant_result"]

        st.divider()

        st.header("🩺 Disease Detection Result")

        st.markdown(result)

        # ---------------------------------
        # VOICE ASSISTANT
        # ---------------------------------

        st.divider()

        st.header("🔊 Voice Assistant")

        if st.button("▶ Listen to the Result"):

            try:

                with st.spinner("Preparing voice output..."):

                    speech = gTTS(
                        text=result,
                        lang=st.session_state["plant_language"],
                        slow=False
                    )

                    audio_buffer = BytesIO()
                    speech.write_to_fp(audio_buffer)
                    audio_buffer.seek(0)

                    st.audio(
                        audio_buffer,
                        format="audio/mp3"
                    )

            except Exception as e:

                st.error(
                    "Voice generation failed. "
                    "Check internet connection and language support."
                )
                st.caption(str(e))

        st.success("Plant analysis completed!")

else:

    st.info(
        "Take a picture or upload a plant leaf photo "
        "to begin disease detection."
    )

st.divider()

st.caption(
    "AI Plant Doctor | Smart Agriculture Project"
)
