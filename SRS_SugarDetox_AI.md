# Software Requirements Specification (SRS)

**Project Name:** SugarDetox AI (Internal Code Name)
**Document Version:** 1.0
**Target Platform:** iOS (Native)

## 1. Introduction

### 1.1 Purpose

The purpose of this document is to define the software requirements for an iOS application designed to help users track, reduce, and eliminate added sugar from their diets. It outlines the functional, non-functional, and AI-specific architectural requirements necessary for development, testing, and deployment.

### 1.2 Product Scope

The application is a B2C health utility featuring AI-assisted nutritional scanning, habit tracking, and structured detox challenges. It monetizes via a freemium model using Apple In-App Purchases (IAP) for auto-renewing subscriptions. The MVP is restricted to the iOS ecosystem, leveraging native Apple frameworks for computer vision and local data persistence.

### 1.3 References & Competitive Baseline

The following live applications serve as baseline references for user experience (UX), feature parity, and monetization flows. The development team must review these applications prior to UI/UX wireframing:

1. **No Sugar Challenge: Sugar Free** (Primary Reference for UI/UX and AI Scanner)
   [App Store Link](https://apps.apple.com/app/no-sugar-challenge-sugar-free/id6740812600)
2. **Quit Sugar** (Reference for tracking logic and localized implementations)
   [App Store Link](https://apps.apple.com/app/quit-sugar/id6760316103)
3. **Sober: Sobriety Tracker** (Reference for gamification and milestone badges)
   [App Store Link](https://apps.apple.com/us/app/sober-sobriety-tracker/id863872931)
4. **Formula - weight loss diet app** (Reference for the onboarding funnel)
   [App Store Link](https://apps.apple.com/us/app/formula-weight-loss-diet-app/id6511248415)

## 2. Overall Description

### 2.1 Product Perspective

The system operates as a standalone iOS application with a lightweight cloud backend solely for subscription verification, analytics, and proxying requests to third-party Large Language Model (LLM) APIs. Core user data (streaks, logged foods) must prioritize local, on-device storage.

### 2.2 User Characteristics

The target users are non-technical consumers seeking health improvements. The interface must require zero manual data entry for food logging where possible, relying instead on camera-based automation to reduce user friction.

### 2.3 Operating Environment

- **Operating System:** iOS 17.0 and later.
- **Hardware:** iPhone (optimized for iPhone 13 and newer to ensure fast neural engine processing for VisionKit tasks).

## 3. System Architecture & AI Implementation

To deliver high accuracy with low latency, the system utilizes a hybrid Edge-to-Cloud AI architecture.

| Component | Technology Stack | Purpose in System |
| --- | --- | --- |
| Frontend UI | SwiftUI | Renders fluid, responsive interfaces, animations, and the onboarding funnel. |
| Local Database | SwiftData / CoreData | Stores user progress, streak histories, and unlocked milestones locally to ensure offline capability. |
| Edge AI (OCR) | Apple VisionKit (Native) | Captures live camera feed, detects bounding boxes around text on ingredient labels, and extracts raw string data on-device. |
| Cloud AI (LLM) | Google Gemini API (via custom proxy) | Receives the raw OCR text, analyzes it against 60+ aliases for hidden sugar (e.g., maltodextrin, dextrose), and returns a structured JSON response. |
| Monetization | RevenueCat SDK | Manages App Store receipts, subscription states, and paywall logic. |

### 3.1 AI Scanner Data Flow

1. **Trigger:** User opens the "Scanner" view and points the camera at a physical product label.
2. **Extraction (Edge):** `VNRecognizeTextRequest` (Vision framework) processes the video frames and extracts the text locally.
3. **Transmission:** The app sends the extracted string to the backend proxy (to protect API keys).
4. **Processing (Cloud):** The backend queries the Gemini API with a strict system prompt:

   > Analyze this ingredient list. Return a JSON object with `contains_added_sugar` (boolean), `flagged_ingredients` (array of strings), and `risk_level` (Low/Medium/High).

5. **Rendering:** The SwiftUI view updates dynamically based on the JSON response, highlighting the dangerous ingredients in red.

## 4. Specific Requirements

### 4.1 Functional Requirements

#### REQ-01: Personalized Onboarding Funnel

- The system shall present a multi-step questionnaire upon first launch capturing current sugar consumption, primary goals, and largest craving triggers.
- The system shall use this data to recommend a specific challenge track (e.g., 7-Day Total Detox vs. 21-Day Gradual Reduction).

#### REQ-02: AI Ingredient Scanner

- The system shall allow users to scan food labels using the device's rear camera.
- The system shall identify and visually flag hidden sugars from the scanned text.
- The scanner functionality shall be locked behind the premium subscription tier after 3 free trial scans.

#### REQ-03: Habit & Streak Tracking

- The system shall maintain a daily calendar where users log their status ("Sugar-Free Day" or "Slipped Up").
- The system shall calculate and display the user's current consecutive streak.

#### REQ-04: Gamification & Milestones

- The system shall award digital badges at predefined intervals (Day 1, Day 3, Day 7, Day 30).
- The system shall display dynamic health insights tied to the streak (e.g., "Day 14: Your taste buds have recalibrated").

### 4.2 Non-Functional Requirements (NFR)

- **PERF-01 (Performance):** The AI scanner must return a Pass/Fail result to the user within 2.5 seconds of text extraction under standard 4G/5G network conditions.
- **SEC-01 (Security):** LLM API keys must never be bundled within the client iOS application. All AI requests must route through a secure, authenticated developer backend.
- **PRIV-01 (Privacy):** User health data and logged streaks must remain on the device. Analytics tracking (e.g., AppsFlyer) must be strictly anonymized and compliant with Apple's App Tracking Transparency (ATT) framework.
- **REL-01 (Reliability):** The core tracking calendar and milestone features must function 100% offline.

## 5. Delivery & Acceptance Criteria

The software development agency is expected to deliver:

1. Fully documented Swift source code via a Git repository.
2. A staging environment via TestFlight for QA testing.
3. Integration of RevenueCat for immediate App Store submission readiness.
4. Conformance to Apple's Human Interface Guidelines (HIG) to ensure App Store approval.
