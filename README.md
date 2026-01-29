# FORTEX36 PROJECT REPORT: PHOENIX

**Project Title:** HealthVault: Decentralized Emergency Health Record & Blood Response System  
**Hackathon Name:** FORTEX36  
**Date of Submission:** 29-01-2026  


---

## 2. Abstract

In emergency medical situations, immediate access to a patient's medical history and rapid blood donor coordination can be the difference between life and death. **HealthVault** is a comprehensive healthcare platform designed to solve these critical delays. It provides a localized, secure system where patients can store emergency medical data (allergies, blood group, conditions) accessible via a physical **QR Code**. When scanned by emergency responders or hospitals, it provides instant, read-only access to vital profiles even if the patient is unconscious. Furthermore, the system integrates a novel **Emergency Priority Engine** built with Python, which algorithmically scores blood supply requests (0-100) based on medical urgency (e.g., Accident vs. Elective Surgery) and broadcasts SMS alerts to compatible donors using Twilio. This project bridges the gap between patient data accessibility and real-time emergency resource allocation.


---

## 3. Problem Statement

Current healthcare infrastructure often faces two fatal bottlenecks:
1.  **Identity & History Gap:** When a trauma patient arrives at a hospital unconscious, doctors lack immediate access to critical information (allergies, pre-existing conditions, blood type), leading to delayed or risky treatments.
2.  **Inefficient Blood Mobilization:** Blood donation requests are often broadcast chaotically via social media without verification or prioritization. There is no automated way to distinguish a critical trauma case requiring immediate transfusion from a routine elective requirement, leading to donor fatigue and resource mismatch.

This project addresses these gaps by creating a unified, technology-driven ecosystem for emergency identity management and intelligent blood supply chain coordination.


---

## 4. Proposed Solution

**HealthVault** proposes a dual-module solution:

1.  **QR-Based Emergency Profile:**
    *   Patients carry a generated QR code (digital or physical card).
    *   In an emergency, any responder can scan the code to view a "Read-Only" emergency dashboard containing Blood Group, Emergency Contacts, and Medical History.
    *   This works across devices (mobile/desktop) using a robust local network backend.

2.  **Intelligent Blood Response System:**
    *   Hospitals can broadcast blood requests via their dashboard.
    *   A **Python Priority Engine** evaluates the request based on:
        *   **Context:** (Accident=90, Childbirth=80, Surgery=75, Thalassemia=60).
        *   **Time Decay:** Urgency increases as time passes (+1 score/10 mins).
        *   **Demand Pressure:** Score boosts for high-volume needs (+10 for >5 units).
    *   If the calculated score exceeds 85, the request is flagged as **CRITICAL**.
    *   The system automatically filters eligible, available donors and pushes **Real SMS Alerts** (via Twilio) directly to their phones.


---

## 5. Technology Stack

*   **Frontend:**
    *   **React.js (Vite):** Selected for its component-based architecture, enabling a responsive and dynamic UI for both Dashboard and Mobile views.
    *   **CSS3 (Glassmorphism):** Used to create a modern, premium, and trustworthy medical interface.

*   **Backend:**
    *   **Node.js & Express:** Chosen for its non-blocking I/O, ideal for handling handling multiple concurrent API requests for cross-device access.
    *   **File-Based JSON Storage:** Used as a lightweight, portable database solution for the hackathon prototype (easily upgradeable to MongoDB/SQL).

*   **Logic & Integration:**
    *   **Python:** Selected for the "Priority Engine" to handle deterministic scoring logic, ensuring medical accuracy.
    *   **Child Processes:** Node.js communicates with Python scripts to utilize the scoring algorithms.
    *   **Twilio API:** Integrated for reliable, real-time SMS delivery to specialized medical staff/donors.


---

## 6. System Architecture and Design

The system follows a modular 3-tier architecture:

1.  **Presentation Layer (Frontend):**
    *   **Patient Portal:** Profile management, Medical Record uploads, QR Code generation.
    *   **Hospital Portal:** Patient Search (ID-based), Record Viewing, Blood Broadcast Dashboard.
    *   **Public Access:** Emergency Mobile view triggered by QR Scan.

2.  **Application Layer (Backend):**
    *   **REST API:** Endpoints for Users, Patients, Hospitals, and Records (`/api/users`, `/api/records`).
    *   **Priority Controller:** Intercepts blood requests, spawns a Python subprocess to execute the `priority_engine` algorithm, and returns the calculated Urgency Score.
    *   **Notification Service:** Triggers Twilio API calls based on the engine's output and donor database queries.

3.  **Data Layer:**
    *   Structured JSON files (`data.json`) storing relational data between Users, Patients, and Logged Events.

**Flow:**
`User Request` -> `React UI` -> `Node Express API` -> `Python Engine (Calculation)` -> `Twilio (SMS)` -> `User Notified`


---

## 7. Implementation Details

### Core Modules:

1.  **Cross-Device Access:**
    *   Implemented a dynamic backend host configuration. The React app detects the local network IP to generate QR codes that work on mobile devices connected to the same WiFi, overcoming standard `localhost` restrictions.

2.  **The Priority Engine (`server/priority_engine/`):**
    *   Written in **Python** for clean logical separation.
    *   Class-based implementation (`engine.py`) with strict strict unit tests (`test_engine.py`).
    *   **Algorithm:** `Score = Base(Context) + (TimeDelta / 10) + DemandBonus`. Caps at 100.
    *   **Auto-Escalation:** System automatically marks requests >85 as "CRITICAL", bypassing standard filters.

3.  **Blood Broadcast & SMS:**
    *   **Hospital Dashboard:** Added a specific UI to select "Context" (e.g., Trauma) which maps to backend enums.
    *   **Backend logic:**
        ```javascript
        exec(`python cli.py "${input}"`, (err, stdout) => {
            const score = JSON.parse(stdout).score;
            if (score > THRESHOLD) sendSMS(donors);
        });
        ```


---

## 8. Results and Demonstration

**Key Achievements:**
1.  **Functional QR System:** Successfully demonstrated scanning a laptop screen with a mobile phone to load the patient's Emergency Profile instantly without login.
2.  **Algorithm Accuracy:** The Priority Engine correctly identified "Mass Casualty" scenarios (Score 100) vs. "Routine Care" (Score 60) in validation scripts.
3.  **Real-Time Alerts:** Integrated Twilio to send actual SMS messages to registered donor numbers when urgency thresholds were met.
4.  **Secure Access:** Implemented ID-based search restrictions so hospitals can only view records of consenting patients.


---

## 9. Challenges Faced

1.  **Cross-Device Connectivity:** Initially, the mobile phone could not connect to the laptop's `localhost`. We resolved this by configuring the Vite server to expose the Network IP (`0.0.0.0`) and dynamically generating QR codes with the machine's local IP address.
2.  **Backend Port Conflicts:** Running React and Express simultaneously caused port collisions. We implemented automation scripts (`kill-servers.bat`) to clean up zombie processes and assigned distinct ports (3000 & 3001).
3.  **Python-Node Integration:** Passing JSON data reliably between Node.js and Python command-line arguments required careful escaping and error handling to ensure the server didn't crash on invalid inputs.


---

## 10. Future Scope

1.  **Blockchain Integration:** Storing the immutable medical history logs on a private blockchain (Hyperledger) to ensure tamper-proof audit trails.
2.  **Geolocation Matching:** Upgrading the Blood Response system to filter donors not just by blood type, but by their real-time GPS proximity to the hospital using the Google Maps API.
3.  **Wearable Integration:** Automatically updating the "Patient Condition" (Heart Rate, Oxygen) via Apple Watch/Fitbit APIs when the QR code is scanned.


---

## 11. Conclusion

HealthVault successfully demonstrates how modern web technologies and algorithmic logic can modernize emergency healthcare. By combining the accessibility of QR codes with the intelligence of a Python-based priority engine, we created a system that not only stores data but actively helps manage critical resources. The project is a working prototype of a smarter, safer, and more connected medical ecosystem.


---

## 12. References

1.  **React Documentation:** https://react.dev/
2.  **Node.js API Reference:** https://nodejs.org/docs/
3.  **Twilio Programmable SMS:** https://www.twilio.com/docs/sms
4.  **Iconography:** Lucide React Icons
5.  **Medical Triage Standards:** Referenced broadly for Priority Engine logic (START Triage protocols).

**GitHub Repository:** [Insert Repository Link Here]
