---
title: pullydragged PRD - Live Chariot Tracking Platform
status: final
created: 2026-08-21
updated: 2026-08-21
---

# PRD: pullydragged (Live Chariot Tracking Web App)

## 0. Document Purpose
This Product Requirements Document (PRD) defines the functional, non-functional, and user experience specifications for **pullydragged**, a live chariot (Rath) tracking web application designed for the Rath Yatra festival. This document serves as the foundational contract for engineering, UX design, architecture, and epic creation. It establishes both the **MVP scope** (focusing on the **Itahari** and **Biratnagar** Rath Yatra processions) and the **long-term architectural vision** as a reusable festival tracking platform across Nepal and worldwide.

## 1. Vision
**pullydragged** transforms the traditional Rath Yatra experience by bringing real-time visibility to thousands of devotees and festival visitors. As a heavy wooden chariot (Rath) is pulled by crowds through city streets, devotees often struggle to know its exact location, remaining distance, or arrival times at key checkpoints. 

**pullydragged** provides an elegant, mobile-optimized live map that automatically tracks and centers on the chariot's GPS location in real time. Built with **Leaflet / OpenStreetMap** and powered by **Socket.io WebSockets**, it features an interactive **route-shaped progress bar** that dynamically fills as the chariot advances, along with rich **checkpoint cards** displaying historical info, photos, and live announcements. Built with modular multi-tenant event routing (e.g. `/itahari`, `/biratnagar`), the application serves as a scalable template for tracking chariot and parade festivals anywhere in the world.

## 2. Target User

### 2.1 Jobs To Be Done (JTBD)
- **Devotees & Public Attendees**: "When I am planning to join the Rath Yatra procession with my family, I want to see the exact real-time position of the chariot and its route progress on my phone, so that I can reach the nearest checkpoint on time without waiting for hours in the heat."
- **Festival Organizers & Operators**: "When I am managing the Rath Yatra event on the chariot, I want a reliable, single-tap GPS broadcaster on a dedicated smartphone that streams location updates seamlessly even in dense crowds, so that the public stays informed without manual overhead."
- **Global Festival Committees (Template Vision)**: "When organizing a parade or chariot procession in any city, I want to deploy a pre-configured tracking website by simply uploading route coordinates and checkpoint media."

### 2.2 Non-Users (v1)
- General vehicle fleet managers or commercial delivery drivers.
- Passive non-event web browsers requiring native app store downloads.

### 2.3 Key User Journeys

- **UJ-1. Ramesh (Devotee) tracks the chariot live in Biratnagar.**
  - **Persona + Context:** Ramesh, living in Biratnagar, wants to take his elderly parents and children to offer prayers at the chariot as it passes near Main Chowk.
  - **Entry State:** Unauthenticated on a mobile smartphone browser, navigating to `pullydragged.com/biratnagar`.
  - **Path:** Ramesh lands directly on the live map view powered by Leaflet. The map **auto-centers** on the glowing Chariot Icon moving along the highlighted route polyline. He looks at the top Progress Bar showing *65% Route Completed*. He taps on the upcoming *Main Chowk Checkpoint* pin on the map.
  - **Climax:** A bottom sheet/modal slides up displaying a crisp photo of the Main Chowk temple, historical significance, and a live announcement: *"Chariot resting here for 15 mins. Prasad distribution active."*
  - **Resolution:** Ramesh leaves home with his family at the exact right moment, joins the crowd seamlessly, and offers prayers without delay.
  - **Edge Case:** If mobile network connection flickers in the dense crowd, the map retains cached route data and displays a subtle indicator: *"Reconnecting... showing last known position (10s ago)"*.

- **UJ-2. Anil (Broadcaster) streams live GPS from the chariot.**
  - **Persona + Context:** Anil, a festival committee volunteer, is riding on the chariot with a dedicated smartphone.
  - **Entry State:** Authenticated via secret operator link at `pullydragged.com/biratnagar/broadcast`.
  - **Path:** Anil taps **"Start Live Broadcast"**. The app requests location permissions and activates a screen WakeLock to prevent device sleep. The interface shows a prominent green status: **"BROADCASTING LIVE - GPS Accuracy: High (3m)"**.
  - **Climax:** As the chariot moves through narrow streets, the app streams GPS coordinates every 3 seconds to the server via Socket.io WebSockets.
  - **Resolution:** Anil keeps the phone mounted or in his pocket; the broadcaster automatically retries and buffers points if network signals drop temporarily.

## 3. Glossary

- **Chariot (Rath)** — The sacred ceremonial wooden cart pulled by crowds during the festival.
- **Rath Yatra** — The annual chariot festival celebrated in Itahari, Biratnagar, and other regional hubs.
- **Devotee** — Public user accessing the web application to view live tracking and checkpoint information.
- **Broadcaster (Operator)** — Authorized user carrying the dedicated GPS-enabled mobile device on the chariot.
- **Route Polyline** — The predefined geographic coordinate path representing the complete procession route.
- **Progress Bar** — Visual UI indicator styled along or matching the route shape that fills dynamically as the chariot completes distance milestones.
- **Checkpoint** — Pre-designated cultural or resting location along the route featuring photos, descriptions, and status updates.
- **Event Slug** — Unique URL identifier (e.g. `/itahari`, `/biratnagar`) routing users to a specific festival instance.
- **WakeLock** — Browser API preventing the broadcaster device screen from dimming or sleeping during location streaming.
- **Leaflet.js** — Open-source JavaScript mapping library used for rendering map tiles and route overlays.
- **Socket.io** — WebSocket-based real-time communication framework used for broadcasting GPS updates.

---

## 4. Features

### 4.1 Multi-Event & Festival Routing Engine
**Description:** Core architecture supporting unique URL slugs per city/event (`/itahari`, `/biratnagar`). Realizes long-term vision as a reusable template for festivals worldwide.

#### Functional Requirements:
- **FR-1:** System MUST resolve incoming requests by Event Slug (e.g., `/itahari`, `/biratnagar`) to load city-specific route polylines, checkpoints, and branding. Realizes UJ-1.
- **FR-2:** System MUST provide a root directory/landing page allowing users to select active festivals if no slug is provided in the URL.
- **FR-3:** System MUST isolate Socket.io room channels per Event Slug (e.g. `room:itahari`, `room:biratnagar`) so real-time location streams do not cross-talk between cities.

### 4.2 Live GPS Broadcasting & Location Ingestion Subsystem
**Description:** Operator interface running on a dedicated mobile phone on the chariot, continuously broadcasting GPS location points to the server via Socket.io. Realizes UJ-2.

#### Functional Requirements:
- **FR-4:** Operator interface MUST capture High-Accuracy GPS coordinates (latitude, longitude, speed, timestamp) from the browser Geolocation API. Realizes UJ-2.
- **FR-5:** Operator interface MUST implement browser Screen WakeLock to maintain continuous background GPS transmission without sleeping. Realizes UJ-2.
- **FR-6:** Operator interface MUST queue failed location payloads locally during network dropouts and automatically flush buffered points via Socket.io when connectivity is restored.

### 4.3 Real-Time Interactive Map & Progress Engine
**Description:** Public-facing mobile-first interactive map built with Leaflet.js. Displays the chariot icon, auto-centers on location, and animates route completion progress. Realizes UJ-1.

#### Functional Requirements:
- **FR-7:** Public map MUST auto-center on the chariot's current GPS position while allowing users to manually pan/zoom with a "Recenter on Chariot" floating action button. Realizes UJ-1.
- **FR-8:** System MUST calculate distance completed along the pre-defined Route Polyline and render a dynamic Progress Bar (0% to 100%). Realizes UJ-1.
- **FR-9:** Public Leaflet map MUST display custom map markers for all Checkpoints with visual indicators for *Upcoming*, *Current/Resting*, and *Passed* statuses.

### 4.4 Checkpoint & Announcement Cards
**Description:** Rich interactive content cards displayed when clicking a checkpoint pin or when the chariot arrives at a resting point. Realizes UJ-1.

#### Functional Requirements:
- **FR-10:** Checkpoint view MUST display a high-quality photo, historical/cultural description, estimated arrival time, and current status. Realizes UJ-1.
- **FR-11:** Authorized administrators MUST be able to post real-time text announcements/updates (e.g., "Prasad distribution started", "15-minute rest") attached to specific checkpoints.
- **FR-12:** Public view MUST display live announcements as top notification banners or toast popups on the active map screen.

### 4.5 Festival Template & Admin Route Configuration
**Description:** Administrative setup system for defining new festival instances, uploading route GPX/GeoJSON coordinates, and managing checkpoints.

#### Functional Requirements:
- **FR-13:** Admins MUST be able to create a new Event Instance with title, slug, city name, banner image, and festival date.
- **FR-14:** Admins MUST be able to upload or draw a Route Polyline (GeoJSON/GPX format) and assign ordered Checkpoints along the line.
- **FR-15:** System MUST store all event metadata in a structured schema to enable instant deployment of new festival tracking instances.

---

## 5. Non-Goals (Explicit for v1 / MVP)
- Native iOS / Android app store builds (v1 is 100% Web/PWA).
- User registration / account creation for public devotees.
- Monetization, ticketing, or paid ad integrations in v1.
- Complex turn-by-turn navigation routing for vehicles.

---

## 6. MVP Scope

### 6.1 In Scope for MVP
- Multi-event URL routing for **Itahari** (`/itahari`) and **Biratnagar** (`/biratnagar`).
- Mobile-first Web App with Leaflet.js / OpenStreetMap.
- Real-time WebSocket broadcasting using **Socket.io**.
- Auto-centering chariot tracking with custom Rath icon.
- Route-shaped visual Progress Bar.
- Dedicated Broadcaster mobile web page with WakeLock & offline buffer.
- Interactive Checkpoint cards with photos, descriptions, and live announcements.
- Admin route & checkpoint configuration.

### 6.2 Deferred to Future Releases (v2 / Global Template Expansion)
- Automated GPX route generator tool.
- Multi-language support (Nepali, English, Maithili).
- Historical event archive and playback animation of past processions.

---

## 7. Success Metrics & Counter-Metrics

### Primary Metrics
- **SM-1 (Live Latency):** GPS position updates reflected on public devotee screens within **< 3 seconds** over Socket.io WebSockets. (Validates FR-4, FR-7)
- **SM-2 (Page Performance):** Initial Leaflet map page load time on 3G/4G mobile devices under **< 1.5 seconds**. (Validates FR-1, FR-7)
- **SM-3 (Broadcaster Reliability):** 99.5%+ uptime of continuous GPS stream during 6+ hour procession. (Validates FR-4, FR-5, FR-6)

### Counter-Metrics (Do Not Optimize At Expense Of Integrity)
- **SM-C1 (Broadcaster Battery Consumption):** Battery drain on the dedicated broadcaster smartphone MUST NOT exceed **15% per hour**. (Counterbalances aggressive high-frequency GPS polling).
- **SM-C2 (Server Bandwidth Overhead):** Payload per Socket.io location frame MUST remain **< 500 bytes** to prevent server bandwidth exhaustion during high user concurrency.

---

## 8. Resolved Technology Choices

1. **Map Engine**: **Leaflet.js + OpenStreetMap** (Selected for zero licensing costs, lightweight footprint, and high reliability under heavy traffic).
2. **Real-Time Communication Protocol**: **Socket.io (WebSockets with HTTP Long-Polling fallback)** (Selected for sub-second bidirectional streaming and room-based multi-tenant channel isolation).

---

## 9. Assumptions Index

- `[ASSUMPTION: GPS Hardware]` A modern Android/iOS smartphone with active mobile data and GPS will be mounted on the chariot during the procession.
- `[ASSUMPTION: Connectivity]` Cellular network (4G/5G) will be operational in Itahari and Biratnagar, with temporary local congestion handled by offline message queueing.
- `[ASSUMPTION: Infrastructure]` Node.js server with Socket.io will be deployed to support concurrent WebSocket connections from devotees.
