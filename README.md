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
