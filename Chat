import streamlit as st
import pandas as pd
from transformers import pipeline

# Load Hugging Face model (can swap with IBM watsonx Granite if available)
chatbot = pipeline("conversational", model="microsoft/DialoGPT-medium")

# Initialize session state
if "history" not in st.session_state:
    st.session_state.history = []
if "df" not in st.session_state:
    st.session_state.df = None

st.title("💬 Personal Finance Chatbot")
st.write("Powered by Hugging Face + IBM Watsonx")

# User profile inputs
persona = st.selectbox("I am:", ["Student", "Professional"])
income = st.number_input("Monthly Income", value=30000)
currency = st.selectbox("Currency", ["INR", "USD", "EUR", "GBP"])
goal = st.text_input("Your Goal", "Build a 3-month emergency fund")

# CSV upload
uploaded = st.file_uploader("Upload your expenses CSV", type=["csv"])
if uploaded:
    df = pd.read_csv(uploaded)
    df["Amount"] = pd.to_numeric(df["Amount"], errors="coerce").fillna(0.0)
    st.session_state.df = df
    st.success("CSV loaded successfully ✅")

# Chat input
user_input = st.text_input("Ask me anything about your finances:")

def generate_context():
    if st.session_state.df is None:
        return "No expense data uploaded yet."
    df = st.session_state.df
    total_spend = df["Amount"].sum()
    top_cat = df.groupby("Category")["Amount"].sum().sort_values(ascending=False).head(1)
    if not top_cat.empty:
        top_name, top_value = top_cat.index[0], top_cat.iloc[0]
    else:
        top_name, top_value = "Misc", 0
    savings = max(income - total_spend, 0)
    return (
        f"You spent {currency} {top_value:,.0f} on {top_name}. "
        f"Your total expenses = {currency} {total_spend:,.0f}. "
        f"Estimated savings = {currency} {savings:,.0f}."
    )

if user_input:
    context = generate_context()
    prompt = (
        f"Persona: {persona}\n"
        f"Monthly income: {income} {currency}\n"
        f"Goal: {goal}\n"
        f"Data summary: {context}\n"
        f"User asked: {user_input}\n"
        f"Answer in a helpful, clear way tailored for {persona.lower()}."
    )
    response = chatbot(prompt)[0]["generated_text"]
    st.session_state.history.append((user_input, response))

# Display chat
for q, a in st.session_state.history:
    st.markdown(f"**You:** {q}")
    st.markdown(f"**Bot:** {a}")
