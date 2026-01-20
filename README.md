# MikroTik-LibreNMS-Quota-Tracker


1. The Configuration Template (config.py.example)

Instead of putting IPs in the main script, we create a template. You will tell your users to rename this to config.py and fill in their own details.
Python

# Pakistani-Solutions-NetSentinel Configuration Template

# --- NETWORK SETTINGS ---
MIKROTIK_IP = '192.168.88.1'      # Replace with your Router IP
LIBRENMS_IP = '192.168.88.10'     # Replace with your LibreNMS Server IP

# --- CREDENTIALS ---
MIKROTIK_USER = 'api_user'
MIKROTIK_PW = 'YOUR_STRONG_PASSWORD'

DB_CONFIG = {
    'host': 'db',                 # Usually 'db' or 'localhost'
    'user': 'librenms',
    'password': 'DB_PASSWORD',
    'database': 'librenms'
}

# --- EMAIL SETTINGS (SMTP) ---
SMTP_USER = 'your-email@gmail.com'
SMTP_PASS = 'your-app-password'
REPORT_RECEIVER = 'admin@example.com'

# --- QUOTA SETTINGS ---
DEFAULT_DAILY_LIMIT_GB = 10

2. Sanitized quota_tracker.py

This script now pulls data from the config file, keeping the main code clean and "IP-blind."
Python

#!/usr/bin/env python3
# Developed by Pakistani Solutions
import mysql.connector
import routeros_api
from datetime import datetime, timedelta
import smtplib
from email.mime.text import MIMEText
import config  # This imports your hidden settings

def run_tracker():
    try:
        # Connect to Database using config
        db = mysql.connector.connect(**config.DB_CONFIG)
        cursor = db.cursor()

        # Connect to MikroTik using config
        connection = routeros_api.RouterOsApiPool(
            config.MIKROTIK_IP, 
            username=config.MIKROTIK_USER, 
            password=config.MIKROTIK_PW, 
            plaintext_login=True
        )
        api = connection.get_api()
        queues = api.get_resource('/queue/simple').get()
        
        # ... logic to process queues and send alerts using config.SMTP_USER ...
        
    except Exception as e:
        print(f"Error: {e}")

