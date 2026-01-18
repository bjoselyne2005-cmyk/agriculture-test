import streamlit as st
import pandas as pd
import google.generativeai as genai
import os

# -------------------------------
# PAGE CONFIG
# -------------------------------
st.set_page_config(
    page_title="Agricultural Advisory Agent",
    layout="wide"
)

st.title("🌾 Agricultural Advisory Agent")
st.caption("Decision-support tool for farmers and agricultural experts")

# -------------------------------
# API KEY
# -------------------------------
GEMINI_API_KEY = st.secrets.get("GEMINI_API_KEY") or os.getenv("GEMINI_API_KEY")

if not GEMINI_API_KEY:
    st.error("Gemini API key not found. Add it to Streamlit secrets.")
    st.stop()

genai.configure(api_key=GEMINI_API_KEY)
model = genai.GenerativeModel("gemini-1.5-flash")

# -------------------------------
# FILE UPLOAD
# -------------------------------
uploaded_file = st.file_uploader(
    "Upload agricultural data (CSV)",
    type=["csv"]
)

if uploaded_file:
    df = pd.read_csv(uploaded_file)

    st.subheader("📊 Uploaded Data")
    st.dataframe(df)

    data_text = df.to_string(index=False)

    if st.button("🌱 Analyze Farm Data"):
        with st.spinner("Analyzing agricultural data..."):
            prompt = f"""
You are an Agricultural Advisory AI Agent.

TASKS:
1. Interpret the farm data.
2. Identify possible crop health issues (do NOT give final diagnosis).
3. Suggest suitable fertilizers or soil improvements.
4. Suggest pest or disease management practices.
5. Recommend good farming practices.
6. Suggest further tests if needed.
7. Summarize clearly for an agricultural officer.

RULES:
- Use cautious language (possible, may indicate, could suggest).
- Do NOT give final decisions.
- Keep advice practical and sustainable.
- Format output with clear headings.

AGRICULTURAL DATA:
{data_text}
"""

            response = model.generate_content(prompt)

        st.subheader("📝 Advisory Report")
        st.write(response.text)

# -------------------------------
# DISCLAIMER
# -------------------------------
st.divider()
st.warning(
    "This tool provides advisory support only. "
    "Final farming decisions should be made by farmers or agricultural experts."
)


