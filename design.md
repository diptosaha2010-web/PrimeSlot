# PrimeSlot SMS Lab - Mobile App Design

## Overview
A mobile app for monitoring and forwarding SMS messages containing specific keywords to a remote server. The app features a dark theme UI with SIM configuration and real-time message logging.

## Screen List
1. **Home Screen** - Main dashboard with all controls and information

## Primary Content and Functionality

### Home Screen
- **Header Section**: App title "PrimeSlot" with logo icon
- **Status Bar**: Connection status indicator (green dot + "Connected · Mobile Data")
- **Background Alerts Section**: 
  - Title: "Background alerts"
  - Status badge: "Network alert ON" (green text)
  - Toggle switch for background alerts
  - Four status icons: SMS, Phone, Net, Battery (all showing green checkmarks)
- **SIM Configuration Section**:
  - Two input fields for SIM-1 and SIM-2 phone numbers
  - Each field has a label above it (SIM-1, SIM-2)
  - Each field has an inline "Save" button to the right
  - Input validation for Bangladeshi phone numbers (01XXXXXXXXX format, 11 digits)
  - Toast notifications for validation errors
- **Latest Message Section**:
  - Display area for forwarded SMS messages
  - "Clear Log" button in top-right corner
  - Shows "Waiting for messages..." when empty
- **Footer**: Removed (no banking/exclude list text)

## Key User Flows

1. **Configure SIM Numbers**:
   - User taps SIM-1 input field
   - Enters Bangladeshi phone number (e.g., 01712345678)
   - Taps "Save" button
   - Toast shows success/error message
   - Number is saved to local storage

2. **Monitor Messages**:
   - App runs in background with foreground service
   - SMS received with "(IVACBD)" keyword
   - Message is forwarded to server
   - Latest message is displayed in the app
   - User can clear log with "Clear Log" button

3. **Toggle Background Alerts**:
   - User taps toggle switch
   - Background alert monitoring starts/stops
   - Status updates in UI

## Color Choices
- **Background**: #0a0e17 (very dark blue/navy)
- **Card/Surface**: #1e293b (dark slate)
- **Primary Text**: #ffffff (white)
- **Secondary Text**: #9ca3af (light gray)
- **Accent/Success**: #10b981 (green)
- **Primary Button**: #6366f1 (indigo/blue gradient)
- **Border**: #334155 (dark gray)

## Design Notes
- Portrait orientation only (9:16 aspect ratio)
- One-handed usage priority
- Dark theme throughout
- Rounded corners on cards and buttons
- Consistent padding and spacing
- Clear visual hierarchy with typography
