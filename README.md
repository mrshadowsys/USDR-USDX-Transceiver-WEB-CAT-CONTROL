USDX Remote HF Transceiver Control – ESP32-C3

Description:
This project provides a remote web interface for controlling a USDX HF transceiver via an ESP32-C3. It supports full CAT control, frequency tuning, mode selection, relay control, presets, and live logging of commands from connected devices. Designed for multi-user remote operation on a local network, it features a realistic control panel UI with neon-green displays and buttons.

Features:

Full frequency control: fine (0.1 kHz) to coarse (1 MHz) steps
Mode selection: LSB, USB, AM, FM, CW
HF Presets: LSB and USB bands (e.g., 6993, 7085, 7148 kHz, 27305, 27455 USB)
UHF and antenna relay control with toggle buttons
Live console log of last 20 CAT commands with device IPs
Real-time updates: all connected devices stay in sync
ESP32-C3 safe pin mapping for UART CAT and relays
Clean green-on-black UI for easy visibility

Hardware Requirements:

ESP32-C3 Supermini (HW-466AB)
USDX HF transceiver with CAT interface
Two relays for UHF link and antenna switching
Standard WiFi network

Pin Mapping (ESP32-C3 Supermini):

Function	Pin	Description
CAT RX	3	ESP32 UART RX to transceiver TX
CAT TX	4	ESP32 UART TX to transceiver RX
Status LED	8	Onboard LED for heartbeat
UHF Relay	5	Relay for UHF link control
Antenna Relay	6	Relay for Antenna 1 / Antenna 2 switch

Usage:

Flash the ESP32-C3 with this code
Connect relays and CAT interface to your transceiver
Connect ESP32 to your WiFi network
Access the web interface via http://<ESP32_IP>
Control frequencies, modes, relays, and view live command logs
