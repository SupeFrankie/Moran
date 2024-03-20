import cv2 #For the cameras
import requests
import base64 #System type
import time
import sqlite3 #For the databse to log alerts

# Initialize SQLite database for logging alerts
conn = sqlite3.connect('alerts.db')
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS alerts
             (id INTEGER PRIMARY KEY AUTOINCREMENT,
              timestamp TEXT,
              type TEXT,
              message TEXT)''')
conn.commit()

# Function to detect emergency conditions using computer vision
def detect_emergency(frame):
    # Placeholder detection logic
    # Replace this with your actual computer vision model and detection logic
    # For example, you could use object detection models to detect specific objects like weapons, etc.
    # This function should return True if an emergency condition is detected, False otherwise
    # Example:
    # emergency_detected = my_model.detect(frame)
    # return emergency_detected
    return False  # Placeholder return value

# Function to send alert message via SMS to residents
def send_resident_alert():
    try:
        # Replace these placeholders with your Safaricom API credentials
        resident_consumer_key = "your_consumer_key"
        resident_consumer_secret = "your_consumer_secret"
        resident_shortcode = "your_shortcode"
        resident_phone_number = "resident_phone_number"
        resident_message = "Emergency alert: Possible threat detected in your area. Please stay indoors and remain vigilant."

        # Get access token
        auth_url = "https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials"
        credentials = f"{resident_consumer_key}:{resident_consumer_secret}"
        encoded_credentials = base64.b64encode(credentials.encode()).decode()
        headers = {"Authorization": f"Basic {encoded_credentials}"}
        response = requests.get(auth_url, headers=headers)

        if response.status_code == 200:
            access_token = response.json().get("access_token")

            # Send SMS to residents
            sms_url = "https://sandbox.safaricom.co.ke/sms/v1/send"
            headers = {
                "Authorization": f"Bearer {access_token}",
                "Content-Type": "application/json"
            }
            payload = {
                "message": resident_message,
                "to": resident_phone_number,
                "from": resident_shortcode
            }
            response = requests.post(sms_url, json=payload, headers=headers)

            if response.status_code == 201:
                print("Resident alert message sent successfully.")
            else:
                print("Failed to send resident alert message. Error:", response.text)
        else:
            print("Failed to obtain access token for residents. Error:", response.text)
    except Exception as e:
        print("An error occurred while sending resident alert:", str(e))

# Function to send alert message via SMS to security personnel
def send_security_alert():
    try:
        # Replace these placeholders with your Safaricom API credentials for security personnel
        security_consumer_key = "your_security_consumer_key"
        security_consumer_secret = "your_security_consumer_secret"
        security_shortcode = "your_security_shortcode"
        security_phone_number = "security_phone_number"
        security_message = "Emergency alert: Please respond immediately to the situation in your area."

        # Get access token
        auth_url = "https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials"
        credentials = f"{security_consumer_key}:{security_consumer_secret}"
        encoded_credentials = base64.b64encode(credentials.encode()).decode()
        headers = {"Authorization": f"Basic {encoded_credentials}"}
        response = requests.get(auth_url, headers=headers)

        if response.status_code == 200:
            access_token = response.json().get("access_token")

            # Send SMS to security personnel
            sms_url = "https://sandbox.safaricom.co.ke/sms/v1/send"
            headers = {
                "Authorization": f"Bearer {access_token}",
                "Content-Type": "application/json"
            }
            payload = {
                "message": security_message,
                "to": security_phone_number,
                "from": security_shortcode
            }
            response = requests.post(sms_url, json=payload, headers=headers)

            if response.status_code == 201:
                print("Security alert message sent successfully.")
            else:
                print("Failed to send security alert message. Error:", response.text)
        else:
            print("Failed to obtain access token for security personnel. Error:", response.text)
    except Exception as e:
        print("An error occurred while sending security alert:", str(e))

# Function to log alerts in the database
def log_alert(timestamp, alert_type, message):
    c.execute("INSERT INTO alerts (timestamp, type, message) VALUES (?, ?, ?)", (timestamp, alert_type, message))
    conn.commit()

# Main function
def main():
    # Replace '0' with the appropriate camera index if using a USB camera
    # Replace 'your_stream_url' with the URL of the IP camera stream if using an IP camera
    cap = cv2.VideoCapture('your_stream_url')  

    while cap.isOpened():
        ret, frame = cap.read()

        # Perform object detection and tracking
        # Replace this with your actual computer vision object detection and tracking logic
        emergency_detected = detect_emergency(frame)

        if emergency_detected:
            timestamp = time.strftime('%Y-%m-%d %H:%M:%S')
            log_alert(timestamp, 'Emergency', 'Possible threat detected in the area')
            send_resident_alert()
            send_security_alert()

        # Display the video feed (optional)
        cv2.imshow('Video Feed
