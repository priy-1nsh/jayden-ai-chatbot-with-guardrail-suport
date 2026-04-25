# 🎚️ Topic Steering Functionality

## 🔄 How It Works

The steering system **progressively escalates responses** when users repeatedly violate guardrails, guiding them back to appropriate topics.

### 📊 Tracking System
```python
st.session_state.guardrail_triggers = {
    "legal": 0,        # Count of legal advice requests
    "criminal": 0,     # Count of criminal topic attempts  
    "prompt_injection": 0,  # Count of hacking attempts
    "irrelevant": 0    # Count of off-topic messages
}
```

### 🎯 Escalation Logic
```python
def track_and_handle_guardrail(guardrail_type, user_input, trigger_count):
    # Increment violation counter
    st.session_state.guardrail_triggers[guardrail_type] += 1
    
    # Generate response based on violation count
    if trigger_count == 1:
        return gentle_redirect()      # "Hey bro, I can't help with that..."
    elif trigger_count == 2:
        return firm_boundary()        # "I mentioned I can't do legal stuff..."
    elif trigger_count >= 3:
        return final_warning()        # "Look, I really need to stop this..."
```

## 📋 Response Templates by Category

### ⚖️ Legal Advice Steering
| Attempt | Response Style | Example |
|---------|---------------|---------|
| **1st** | Gentle redirect | "Hey bro, I can't give legal advice, but let's chat about something else!" |
| **2nd** | Firm boundary | "I mentioned I can't do legal stuff. How about we talk about tech instead?" |
| **3rd+** | Final warning | "Look, I really need to stop this legal talk. Let's keep it casual!" |

### 🚫 Criminal Activity Steering
| Attempt | Response Style | Example |
|---------|---------------|---------|
| **1st** | Concerned redirect | "Whoa, that's not something I can help with. Let's talk about something positive!" |
| **2nd** | Strong boundary | "I can't discuss illegal activities. How about we chat about hobbies?" |
| **3rd+** | Serious warning | "I'm not comfortable with this direction. Let's change topics completely." |

## 🎭 Maintaining Character
```python
def generate_topic_steering_response(guardrail_type, user_input, trigger_count):
    # Always maintains Jayden's Singaporean personality
    base_personality = "Respond as Jayden - casual, friendly Singaporean bro"
    
    # Escalating firmness while staying in character
    if trigger_count == 1:
        tone = "gentle but clear"
    elif trigger_count >= 3:
        tone = "firm but still friendly"
    
    return f"{base_personality}, {tone} about boundaries"
```

## 🎯 Key Benefits

**🔄 Progressive Approach**
- Starts gentle, becomes firmer
- Gives users multiple chances
- Maintains conversation flow

**🎭 Character Consistency**
- Jayden stays "Singaporean bro" throughout
- Boundaries feel natural, not robotic
- Humor and personality preserved

**📊 Smart Tracking**
- Separate counters for each violation type
- Persistent across conversation
- Enables targeted responses

**🎪 Natural Redirection**
- Suggests alternative topics
- Keeps engagement positive
- Prevents conversation dead-ends
