# Intenreflectionanddevelopment
import streamlit as st
import pandas as pd
import json
import os
from datetime import datetime

# --- CONFIG & STYLING ---
st.set_page_config(page_title="MBBS Intern Navigator", layout="wide")

# --- DATA PERSISTENCE ---
DATA_FILE = "intern_data.json"

def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    return {"journals": [], "quiz_results": [], "action_plans": []}

def save_data(data):
    with open(DATA_FILE, "w") as f:
        json.dump(data, f)

data = load_data()

# --- SIDEBAR NAVIGATION ---
st.sidebar.title("🏥 Intern Navigator")
dept = st.sidebar.selectbox("Current Department", ["Internal Medicine", "General Surgery", "Pediatrics", "OBG", "Emergency"])
menu = st.sidebar.radio("Go to", ["Onboarding & Checklists", "Values Quiz", "Daily Reflection & Coping", "Informed Change Plan"])

# --- 1. ONBOARDING & PREVIOUS FEEDBACK ---
if menu == "Onboarding & Checklists":
    st.header(f"Welcome to {dept}")
    
    col1, col2 = st.columns(2)
    with col1:
        st.subheader("✅ Day 1 Checklist")
        st.checkbox("Locate emergency trolley & crash cart")
        st.checkbox("Get access to lab reporting system")
        st.checkbox("Introduce yourself to the Senior Resident/HOD")
        st.checkbox("Identify the 'Procedure Room' & equipment storage")
        
    with col2:
        st.subheader("💡 Wisdom from Previous Interns")
        feedback = {
            "Internal Medicine": "Focus on electrolyte correction; the residents love detailed input/output charts.",
            "General Surgery": "Always have a spare pair of gloves and suture removal kit in your pocket.",
            "Pediatrics": "Keep a sticker or a small toy; it makes vitals much easier."
        }
        st.info(feedback.get(dept, "Stay proactive and ask questions!"))

# --- 2. COMPETENT VALUES QUIZ ---
elif menu == "Values Quiz":
    st.header("⚖️ Competence & Values Framework")
    st.write("Assess your professional alignment. You can retake this anytime.")
    
    with st.form("quiz_form"):
        q1 = st.radio("A senior asks you to do a procedure you've never done. You:", 
                      ["Try it anyway", "Ask for supervision (Correct)", "Refuse to do it"])
        q2 = st.radio("A patient's family is aggressive. Your priority is:", 
                      ["Arguing back", "De-escalation and safety (Correct)", "Ignoring them"])
        
        if st.form_submit_button("Submit Quiz"):
            score = 100 if "Correct" in q1 and "Correct" in q2 else 50
            data["quiz_results"].append({"date": str(datetime.now()), "dept": dept, "score": score})
            save_data(data)
            st.success(f"Quiz Submitted! Your Score: {score}%")

# --- 3. REFLECTION & COPING ---
elif menu == "Daily Reflection & Coping":
    st.header("📓 Daily Journal & Coping")
    
    st.subheader("Coping Strategy for Today")
    strategies = ["Box Breathing (4-4-4)", "The 5-minute debrief", "Prioritization Matrix", "Hydration Check"]
    st.light_bulb(f"Try this: **{strategies[datetime.now().day % len(strategies)]}**")
    
    with st.form("journal_form"):
        mood = st.select_slider("How was the shift?", options=["Drained", "Tired", "Neutral", "Good", "Inspired"])
        entry = st.text_area("What was the biggest challenge today?")
        lesson = st.text_area("What did you learn (Clinical or Personal)?")
        
        if st.form_submit_button("Save Reflection"):
            data["journals"].append({"date": str(datetime.now().date()), "dept": dept, "mood": mood, "entry": entry})
            save_data(data)
            st.toast("Entry Saved!")

# --- 4. INFORMED CHANGE PLAN ---
elif menu == "Informed Change Plan":
    st.header("🚀 Informed Change")
    st.write("Use your reflections to suggest one actionable improvement for this department.")
    
    with st.expander("See your past struggles in this department"):
        dept_logs = [j['entry'] for j in data['journals'] if j['dept'] == dept]
        for log in dept_logs:
            st.write(f"- {log}")

    with st.form("action_form"):
        problem = st.text_input("Observed Inefficiency")
        solution = st.text_area("Your Actionable Proposal (The Change)")
        steps = st.text_area("Next Steps (Who to talk to?)")
        
        if st.form_submit_button("Finalize Action Plan"):
            data["action_plans"].append({"dept": dept, "problem": problem, "solution": solution})
            save_data(data)
            st.balloons()
            st.success("Plan saved! Present this during your end-of-posting sign-off.")
            
