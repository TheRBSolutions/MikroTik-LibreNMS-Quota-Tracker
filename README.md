# MikroTik-LibreNMS-Quota-Tracker


This is a significant milestone for Pakistani Solutions. By publishing this on GitHub, you are demonstrating to your clients that your company doesn't just manage networks—you build custom, secure, and data-driven infrastructure.

Below is the complete, high-level documentation and the detailed, sanitized code for your repository: Pakistani-Solutions-NetSentinel.
1. Project Description (The GitHub README)
Executive Summary

Pakistani-Solutions-NetSentinel is a professional-grade network auditing and quota management system. It bridges the gap between real-time SNMP monitoring (LibreNMS) and granular per-user traffic accounting (MikroTik Queue Trees).
The Challenge

In high-traffic environments, standard monitoring often fails to provide a historical audit of total data consumption (GB) per individual user. Furthermore, open management APIs are frequent targets for brute-force attacks.
The Pakistani Solutions Approach

We developed a secure, automated sentinel that:

    Aggregates Data: Pulls per-user queue statistics from MikroTik via a restricted API.

    Analyzes & Stores: Records daily totals in a MySQL database within a Docker environment.

    Alerts & Reports: Sends instant SMTP notifications for quota violations and a comprehensive, ranked HTML report at 6:15 PM daily.

    Secures the Perimeter: Implements IP-level firewall white-listing and read-only API user roles to neutralize external threats.






2. The Detailed Code
File 1: config.py (The Sensitive Settings)

Users will edit this file locally. It is excluded from Git via .gitignore.
Python

# Configuration for Pakistani-Solutions-NetSentinel

# --- INFRASTRUCTURE ---
ROUTER_IP = '0.0.0.0'       # Placeholder for MikroTik IP
LIBRENMS_DB_HOST = 'db'      # Docker service name or IP
DB_NAME = 'librenms'
DB_USER = 'librenms'
DB_PASS = 'YOUR_DB_PASSWORD'

# --- API CREDENTIALS ---
# Use the limited 'api_readonly' user we created
API_USER = 'librenms_api'
API_PASS = 'YOUR_SECURE_API_PASSWORD'

# --- NOTIFICATIONS ---
SMTP_SERVER = 'smtp.gmail.com'
SMTP_PORT = 587
SENDER_EMAIL = 'your-email@gmail.com'
APP_PASSWORD = 'your-gmail-app-password'
RECIPIENT_EMAIL = 'admin@your-company.com'

# --- POLICIES ---
DAILY_LIMIT_GB = 10         # Default limit for employees
DATA_RETENTION_DAYS = 14    # Auto-cleanup period

File 2: quota_tracker.py (The Engine)

This script handles data collection and instant alerts.
Python

#!/usr/bin/env python3
import mysql.connector
import routeros_api
from datetime import datetime, timedelta
import smtplib
from email.mime.text import MIMEText
import config

def get_db_connection():
    return mysql.connector.connect(
        host=config.LIBRENMS_DB_HOST,
        user=config.DB_USER,
        password=config.DB_PASS,
        database=config.DB_NAME
    )

def notify_admin(user, used_gb):
    subject = f"🚨 Quota Alert: {user}"
    body = f"User {user} has exceeded the daily limit.\nUsed: {used_gb:.2f} GB\nLimit: {config.DAILY_LIMIT_GB} GB"
    msg = MIMEText(body)
    msg['Subject'] = subject
    msg['From'] = config.SENDER_EMAIL
    msg['To'] = config.RECIPIENT_EMAIL

    with smtplib.SMTP(config.SMTP_SERVER, config.SMTP_PORT) as server:
        server.starttls()
        server.login(config.SENDER_EMAIL, config.APP_PASSWORD)
        server.send_message(msg)

def sync_data():
    print(f"[{datetime.now()}] Initializing Pakistani Solutions Sentinel...")
    db = get_db_connection()
    cursor = db.cursor()

    # 1. Housekeeping: Delete old records
    retention_date = (datetime.now() - timedelta(days=config.DATA_RETENTION_DAYS)).date()
    cursor.execute("DELETE FROM custom_bandwidth_quotas WHERE check_date < %s", (retention_date,))
    
    # 2. MikroTik Connection
    try:
        connection = routeros_api.RouterOsApiPool(
            config.ROUTER_IP, 
            username=config.API_USER, 
            password=config.API_PASS, 
            plaintext_login=True
        )
        api = connection.get_api()
        queues = api.get_resource('/queue/simple').get()
        
        today = datetime.now().date()
        for q in queues:
            name = q['name']
            # Convert bytes to GB using: $GB = \frac{Bytes}{1024^3}$
            dl_gb = int(q.get('bytes-in', 0)) / (1024**3)
            ul_gb = int(q.get('bytes-out', 0)) / (1024**3)
            
            # Update Database
            cursor.execute("""
                INSERT INTO custom_bandwidth_quotas 
                (queue_name, bytes_download, bytes_upload, check_date)
                VALUES (%s, %s, %s, %s)
                ON DUPLICATE KEY UPDATE
                bytes_download = VALUES(bytes_download),
                bytes_upload = VALUES(bytes_upload)
            """, (name, dl_gb, ul_gb, today))

            # Quota Check
            if dl_gb > config.DAILY_LIMIT_GB:
                cursor.execute("SELECT alert_sent FROM custom_bandwidth_quotas WHERE queue_name=%s AND check_date=%s", (name, today))
                if not cursor.fetchone()[0]:
                    notify_admin(name, dl_gb)
                    cursor.execute("UPDATE custom_bandwidth_quotas SET alert_sent=1 WHERE queue_name=%s AND check_date=%s", (name, today))

        db.commit()
    finally:
        db.close()

if __name__ == "__main__":
    sync_data()

File 3: security_hardening.rsc (The Shield)

RouterOS configuration script for the technical case study.
Plaintext

# Pakistani Solutions Security Hardening Script
# Purpose: Block API Brute-Force and Restrict Access

# 1. Create a Restricted Group
/user group add name=api_readonly policy=read,api,!local,!telnet,!ssh,!ftp,!reboot,!write,!policy,!test,!winbox,!password,!web,!sniff,!sensitive,!romon

# 2. Create the API User
/user add name=librenms_api password=REPLACE_WITH_STRONG_PASSWORD group=api_readonly

# 3. Restrict API Service to the LibreNMS Server IP
/ip service set api address=YOUR_SERVER_IP/32 disabled=no

# 4. Firewall Filter (Top of Input Chain)
/ip firewall filter
add chain=input protocol=tcp dst-port=8728 src-address=YOUR_SERVER_IP action=accept comment="SENTINEL: Allow LibreNMS"
add chain=input protocol=tcp dst-port=8728 action=drop comment="SENTINEL: Block API Brute For
