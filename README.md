# Change Management – Change Lite (ServiceNow)

A scoped ServiceNow application built using Studio that demonstrates
enterprise-grade Change Management aligned with ITIL best practices.

This project showcases automated risk calculation, CAB approvals,
emergency change handling, blackout enforcement, and Incident-to-Change
automation with full GitHub source control.

---

## Project Objective
To design and implement a realistic Change Management solution that
balances governance and agility, using modern ServiceNow development
patterns such as Script Includes, Flow Designer, and scoped applications.

---

## Features Implemented
- Scripted risk scoring engine (Impact + Urgency + Category)
- Automatic risk level classification (Low / Moderate / High)
- CAB approval flow using Flow Designer
- Emergency change fast-track with mandatory justification
- Blackout window enforcement for non-emergency changes
- Incident → Change UI Action for traceability
- GitHub source control integration from Day 1

---

## Technical Components Used
- ServiceNow Studio (Scoped App)
- Script Includes
- Business Rules
- Flow Designer
- UI Policies
- UI Actions
- Schedules (Blackout)
- GitHub Source Control

---

## Installation
1. Clone this repository
2. Import the scoped application via Studio source control
3. Ensure Change Management and Flow Designer plugins are active
4. Create a blackout schedule and update the sys_id in the blackout rule
5. Test with Normal and Emergency changes

---

## Demo Scenarios
1. Create a Normal Change with high risk → CAB approval triggered
2. Approve or reject via Flow Designer approval
3. Create an Emergency Change → auto-approved with justification
4. Attempt Normal Change during blackout → blocked
5. Create Change directly from Incident using UI Action

---

## Notes
- Built entirely using Studio
- Designed for enterprise Change Management scenarios
- Focused on clean, reusable, and interview-ready logic
