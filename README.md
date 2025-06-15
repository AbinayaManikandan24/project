# Install required packages
!pip install ipywidgets --quiet
!pip install scikit-learn --quiet
!pip install pandas --quiet
!pip install matplotlib --quiet
!pip install twilio --quiet

# Imports
from IPython.display import display
import ipywidgets as widgets
import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt
from twilio.rest import Client

# 📊 Simulated Azure Cloud Cost Data
data = {
    'Day': np.arange(1, 31),
    'Azure_VM': np.random.randint(100, 600, size=30),
    'Azure_Storage': np.random.randint(50, 300, size=30),
    'Azure_SQL': np.random.randint(80, 400, size=30),
    'Azure_AppService': np.random.randint(70, 350, size=30)
}

df = pd.DataFrame(data)

# ⚙️ Train Cost Prediction Models
models = {}
services = ['Azure_VM', 'Azure_Storage', 'Azure_SQL', 'Azure_AppService']

for service in services:
    X = df[['Day']]
    y = df[service]
    model = LinearRegression()
    model.fit(X, y)
    models[service] = model

# ✅ Twilio SMS Sending Function
def send_sms_alert(to_number, message_body):
    account_sid = 'ACb4079714aa09c0ef8606e8gvuvyy63'  # Replace with your Twilio SID
    auth_token = '06a997a0bed3186bhuhuiub32a8ddbf'    # Replace with your Twilio Auth Token
    from_number = '+1 845 737 4448' # Replace with your Twilio phone number, e.g. +1XXXXXXXXXX

    client = Client(account_sid, auth_token)
    message = client.messages.create(
        body=message_body,
        from_=from_number,
        to=to_number
    )
    print(f"📲 SMS sent to {to_number}: SID {message.sid}")

# ✅ UI Elements
service_dropdown = widgets.Dropdown(
    options=services,
    description='Service:',
    style={'description_width': 'initial'},
    layout=widgets.Layout(width='50%')
)

usage_input = widgets.IntText(
    value=0,
    description='Current Usage (units):',
    style={'description_width': 'initial'},
    layout=widgets.Layout(width='50%')
)

threshold_slider = widgets.IntSlider(
    value=400,
    min=100,
    max=1000,
    step=50,
    description='Cost Threshold ($):',
    style={'description_width': 'initial'},
    layout=widgets.Layout(width='60%')
)

phone_number_input = widgets.Text(
    value='+91XXXXXXXXXX',  # Example: your own number with country code
    description='Phone Number:',
    style={'description_width': 'initial'},
    layout=widgets.Layout(width='50%')
)

run_button = widgets.Button(
    description='Run Cost Optimization',
    button_style='success',
    layout=widgets.Layout(width='50%')
)

output_area = widgets.Output()

# ✅ Main Logic
def run_optimization(b):
    with output_area:
        output_area.clear_output()

        selected_service = service_dropdown.value
        current_usage = usage_input.value
        threshold = threshold_slider.value
        phone_number = phone_number_input.value

        day_today = 30
        model = models[selected_service]
        predicted_cost = model.predict([[day_today]])[0]

        print(f"\nService Selected: {selected_service}")
        print(f"Current Usage Entered: {current_usage} units")
        print(f"Predicted Cost for Today: ${predicted_cost:.2f}")
        print(f"Threshold Set: ${threshold}")

        # 📢 Prepare alert message
        if predicted_cost > threshold:
            alert_message = f"""\u26A0 HIGH CLOUD COST ALERT!
Service: {selected_service}
Predicted Cost: ${predicted_cost:.2f}
Threshold: ${threshold}

Suggested Actions:
- Scale down unused resources.
- Switch to reserved instances.
- Review autoscaling settings.
- Delete unnecessary storage.
"""
            print("\n\u26A0 High Cost Alert! Sending SMS...")
        else:
            alert_message = f"""\u2705 CLOUD COST UNDER CONTROL
Service: {selected_service}
Predicted Cost: ${predicted_cost:.2f}
Threshold: ${threshold}

No immediate action needed. Keep monitoring!
"""
            print("\n\u2705 Cost is under control. Sending SMS...")

        # ✅ Send SMS via Twilio
        try:
            send_sms_alert(phone_number, alert_message)
        except Exception as e:
            print(f"\u274C SMS sending failed: {e}")

        # ✅ Plot cost trend
        plt.figure(figsize=(8, 4))
        plt.plot(df['Day'], df[selected_service], marker='o')
        plt.axhline(y=threshold, color='r', linestyle='--', label='Threshold')
        plt.title(f"{selected_service} - Cost Trend")
        plt.xlabel('Day')
        plt.ylabel('Cost ($)')
        plt.legend()
        plt.grid(True)
        plt.show()

# 👉 Hook up button to main logic
run_button.on_click(run_optimization)

# 👉 Display UI
display(service_dropdown, usage_input, threshold_slider, phone_number_input, run_button, output_area)
