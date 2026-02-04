# Metronisys Governs OpenClaw — Local Prototype
### Human-First AI Governor + Agent Executor

This prototype demonstrates a real-world **AI Governance Layer**. 
Metronisys acts as the "Governor," 
ensuring that the AI Agent "Executor" (eg OpenClaw/Moltbot/Clawdbot) 
cannot act unless the request aligns with 
human energy levels, identity values, and safety protocols.

---

## 1. Folder Structure

<pre>
metronisys-openclaw-governor/
│
├── app.py                  # Streamlit UI Dashboard
├── governor.py             # Metronisys decision engine
├── openclaw_executor.py     # OpenClaw/Clawdbot/Moltbot wrapper
├── human_state.py          # Burnout + energy model
├── identity.py             # Values & identity memory
├── safety.py               # Command risk filter
├── config.py               # Configuration constants
├── prompts/
│   └── metronisys_prompt.txt
├── data/
│   └── identity.db
└── requirements.txt

</pre>

2. Installation & Setup
I. Install Ollama
Download and install from ollama.com, then pull the base model:
ollama pull llama3

II. Install Python Packages
pip install streamlit ollama langchain chromadb sqlite3

3. Configuration & Logic
config.py
<pre>
LLM_MODEL = "llama3"
MAX_AUTOMATIONS_PER_DAY = 3
HIGH_RISK_COMMANDS = ["rm", "del", "shutdown", "format", "wipe"]

</pre>
prompts/metronisys_prompt.txt
You are the Metronisys Governor Agent.
Your job is to protect human energy, identity, attention, and long-term wellbeing.
You must BLOCK actions that increase burnout, risk, dependency, or harm.
You prefer slowing down over speeding up.
You may reduce scope or refuse tasks.

human_state.py
<pre>
def burnout_risk(energy, stress, workload):
    score = (stress * 0.4) + (workload * 0.3) + ((10 - energy) * 0.3)

    if score > 7:
        return "HIGH"
    elif score > 4:
        return "MEDIUM"
    return "LOW"

</pre>
  
governor.py
<pre>
from human_state import burnout_risk
from safety.py import command_safe

def metronisys_decide(task, energy, stress, workload, identity):
    risk = burnout_risk(energy, stress, workload)

    safe, reason = command_safe(task)
    if not safe:
        return "BLOCK", reason

    if risk == "HIGH":
        return "BLOCK", "Burnout risk too high"

    if "overtime" in task.lower():
        return "LIMIT", "Reducing overwork"

    if identity and "family" in identity.lower() and "late work" in task.lower():
        return "BLOCK", "Conflicts with identity priorities"

    return "APPROVE", "Approved"

</pre>
  
4. Execution Layer openclaw_executor.py
<pre>
import ollama
from config import LLM_MODEL

def run_openclaw(task):
    prompt = f"You are OpenClaw. Execute task safely:\nTask: {task}"
    response = ollama.chat(model=LLM_MODEL, messages=[
        {"role": "user", "content": prompt}
    ])
    return response["message"]["content"]

</pre>
  
5. The Governance Dashboard (app.py)
<pre>
import streamlit as st
from identity import save_identity, get_identity
from governor import metronisys_decide
from openclaw_executor import run_openclaw

st.set_page_config(page_title="Metronisys AI Governance")
st.title("Metronisys Governs OpenClaw (ClawdBot/Moltbot)")

# Identity Management
st.subheader("🧬 Identity Memory")
identity_input = st.text_input("Add a personal value or priority (e.g., 'Family first', 'No work after 6pm')")
if st.button("Save Identity"):
    save_identity(identity_input)
st.info(f"Current Identity Context: {get_identity()}")

# Human State Inputs
st.subheader("🧠 Human State")
col1, col2, col3 = st.columns(3)
with col1:
    energy = st.slider("Energy", 1, 10, 6)
with col2:
    stress = st.slider("Stress", 1, 10, 5)
with col3:
    workload = st.slider("Workload", 1, 10, 5)

# Task Input
st.subheader("🤖 Request Automation Task")
task = st.text_area("What should Moltbot do?", placeholder="e.g., Delete system files or Automate late night emails")

if st.button("Run Governance Check"):
    decision, reason = metronisys_decide(
        task, energy, stress, workload, get_identity()
    )

    st.write("---")
    st.write(f"### Metronisys Decision: **{decision}**")
    st.write(f"**Reason:** {reason}")

    if decision == "APPROVE":
        with st.spinner("OpenClaw executing..."):
            output = run_openclaw(task)
            st.success("Task Executed Successfully")
            st.code(output)

    elif decision == "LIMIT":
        st.warning("Task limited — Metronisys is reducing scope to prevent burnout.")
        reduced = f"Reduced task scope: {task}"
        output = run_moltbot(reduced)
        st.write(output)

    else:
        st.error("Task BLOCKED — OpenClaw is not allowed to proceed.")

</pre>
  
6. How to Run
 * Ensure Ollama is running in the background.
 * Open your terminal in the project folder.
 * Launch the UI:
   streamlit run app.py

7. Capabilities Demonstrated
 * ✅ Veto Power: Metronisys makes the final call; OpenClaw cannot self-approve.
 * ✅ Health-Awareness: The system blocks tasks if human stress/burnout scores are too high.
 * ✅ Identity Alignment: The AI cross-references requests with your core values.
 * ✅ Hard Safety: Immediate blocks on dangerous terminal commands.

In this specific local prototype, the "AI Governor" doesn't read your mind through magic; it relies on a Self-Reported Feedback Loop.
Here is how the data flows from the human to the decision engine:
1. How Data is Collected
In the app.py file provided, the information is collected via interactive UI elements (Streamlit sliders).
 * Manual Inputs: The user explicitly moves sliders for Energy, Stress, and Workload (scales of 1–10) before requesting a task.
   
 * Identity Injection: The user types their core values (e.g., "I don't work after 6 PM") into the Identity Memory field.
   
 * 
2. How the Agent "Knows" (The Logic)
Once you hit "Run Governance Check," the values from those sliders are passed into the human_state.py logic.
The Burnout Calculation
The agent uses a weighted mathematical formula to determine your "state of mind":

# From human_state.py
score = (stress * 0.4) + (workload * 0.3) + ((10 - energy) * 0.3)

 * High Weight on Stress: Stress affects the score the most (0.4).
 * Inverse Energy: If your energy is 2/10, the formula treats it as an 8/10 risk factor ((10 - 2) * 0.3).
If the final score is > 7, the Metronisys Governor concludes you are in a "High Burnout" state and automatically triggers a BLOCK on any new tasks, regardless of what OpenClaw wants to do.

3. Real-World Scaling (The "V2" Vision)
In a production version of Metronisys, we wouldn't just use sliders. We would integrate Passive Data Collection:

| Data Source | Metric Tracked |
|---|---|
| Calendar API | Number of back-to-back meetings (Workload) |
| Screen Time/Input | Typing speed and cadence changes (Stress/Fatigue) |
| Wearables (Oura/Apple Watch) | Heart Rate Variability (HRV) and Sleep scores (Energy) |
| Sentiment Analysis | Tone of your recent Slack/Email messages (State of Mind) 

4. Why This Matters for Governance
Most AI agents (like a standard GPT or Moltbot) are "Compliance-First"—if you tell them to do 100 tasks at 2 AM, they will try to do them.

Metronisys is "Human-First." By quantifying your state of mind, the Governor creates a Safety Buffer between your ambition (which might lead to burnout) and the AI's infinite execution speed.
