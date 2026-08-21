---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-pullydragged-2026-08-21/prd.md
  - _bmad-output/planning-artifacts/architecture/architecture-pullydragged-2026-08-21/ARCHITECTURE-SPINE.md
---

# pullydragged - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for pullydragged, decomposing the requirements from the PRD and Architecture specifications into implementable stories.

## Requirements Inventory

### Functional Requirements

- **FR-1:** System MUST resolve incoming requests by Event Slug (e.g., `/itahari`, `/biratnagar`) to load city-specific route polylines, checkpoints, and branding.
- **FR-2:** System MUST provide a root directory/landing page allowing users to select active festivals if no slug is provided in the URL.
- **FR-3:** System MUST isolate Socket.io room channels per Event Slug (e.g. `room:itahari`, `room:biratnagar`) so real-time location streams do not cross-talk between cities.
- **FR-4:** Operator interface MUST capture High-Accuracy GPS coordinates (latitude, longitude, speed, timestamp) from the browser Geolocation API.
- **FR-5:** Operator interface MUST implement browser Screen WakeLock to maintain continuous background GPS transmission without sleeping.
- **FR-6:** Operator interface MUST queue failed location payloads locally during network dropouts and automatically flush buffered points via Socket.io when connectivity is restored.
- **FR-7:** Public map MUST auto-center on the chariot's current GPS position while allowing users to manually pan/zoom with a "Recenter on Chariot" floating action button.
- **FR-8:** System MUST calculate distance completed along the pre-defined Route Polyline and render a dynamic Progress Bar (0% to 100%).
- **FR-9:** Public Leaflet map MUST display custom map markers for all Checkpoints with visual indicators for *Upcoming*, *Current/Resting*, and *Passed* statuses.
- **FR-10:** Checkpoint view MUST display a high-quality photo, historical/cultural description, estimated arrival time, and current status.
- **FR-11:** Authorized administrators MUST be able to post real-time text announcements/updates (e.g., "Prasad distribution started", "15-minute rest") attached to specific checkpoints.
- **FR-12:** Public view MUST display live announcements as top notification banners or toast popups on the active map screen.
- **FR-13:** Admins MUST be able to create a new Event Instance with title, slug, city name, banner image, and festival date.
- **FR-14:** Admins MUST be able to upload or draw a Route Polyline (GeoJSON/GPX format) and assign ordered Checkpoints along the line.
- **FR-15:** System MUST store all event metadata in a structured schema to enable instant deployment of new festival tracking instances.

### NonFunctional Requirements

- **NFR-1 (Live Latency):** GPS position updates MUST be reflected on public devotee screens within < 3 seconds over Socket.io WebSockets.
- **NFR-2 (Page Performance):** Initial Leaflet map page load time on 3G/4G mobile devices MUST be under < 1.5 seconds.
- **NFR-3 (Broadcaster Reliability):** Broadcaster continuous GPS stream MUST maintain 99.5%+ uptime during 6+ hour procession.
- **NFR-4 (Battery Efficiency):** Battery drain on dedicated broadcaster smartphone MUST NOT exceed 15% per hour.
- **NFR-5 (Payload Efficiency):** Payload per Socket.io location frame MUST remain < 500 bytes to prevent server bandwidth exhaustion.

### Additional Requirements

- **Tech Stack & Starter Setup:** Next.js 14+ (App Router), React 18.3, Tailwind CSS v4, TypeScript, MongoDB Atlas + Mongoose ODM, Socket.io 4.7.x, Leaflet 1.9.4 / React-Leaflet 4.2.1, Turf.js (`@turf/turf`).
- **Dynamic Leaflet Client Import:** Client components using Leaflet (`react-leaflet`) MUST be dynamically loaded with `{ ssr: false }` to prevent SSR `window` reference crashes (AD-1).
- **Multi-Tenant Data & Room Scoping:** Mongo DB models and Socket.io room channels (`event:<eventSlug>`) MUST be strictly scoped by `eventSlug` (AD-2).
- **WakeLock & Offline Resilience Protocol:** Broadcaster interface MUST request Screen WakeLock API (`navigator.wakeLock.request('screen')`) and queue unsent location frames locally (`localStorage`/`IndexedDB`) during drops, flushing sequentially upon reconnect (AD-3).
- **Polyline Nearest-Point Progress Calculation:** Dynamic route completion percentage MUST be computed by projecting `(lat, lng)` points onto GeoJSON polylines using Turf.js (`@turf/nearest-point-on-line`) (AD-4).
- **MongoDB Data Persistence & Schema Design:** Mongoose schemas for Event, Route, Checkpoint, Announcement, and LocationLog models + REST endpoints `/api/v1/events`, `/api/v1/routes`, `/api/v1/checkpoints` (AD-5).

### UX Design Requirements

*None specified (No separate UX Design Contract present).*

### FR Coverage Map

- **FR-1:** Epic 1 - Multi-Event URL Slug Resolution (`/[eventSlug]`)
- **FR-2:** Epic 1 - Root Festival Selector Landing Page (`/`)
- **FR-3:** Epic 3 - Isolated Socket.io Event Rooms (`room:<eventSlug>`)
- **FR-4:** Epic 2 - Broadcaster High-Accuracy Geolocation API Ingestion
- **FR-5:** Epic 2 - Broadcaster Screen WakeLock Integration
- **FR-6:** Epic 2 - Broadcaster Local Queue & Auto-Flush on Reconnect
- **FR-7:** Epic 3 - Public Leaflet Live Map with Recenter FAB
- **FR-8:** Epic 3 - Dynamic Route Polyline Progress Bar (Turf.js)
- **FR-9:** Epic 4 - Interactive Checkpoint Map Pins (*Upcoming*, *Resting*, *Passed*)
- **FR-10:** Epic 4 - Rich Checkpoint Info Sheet (Photo, History, ETA)
- **FR-11:** Epic 4 - Admin Checkpoint Announcement Management
- **FR-12:** Epic 4 - Live Announcement Toast / Notification Banner
- **FR-13:** Epic 1 - Admin Event Instance Management
- **FR-14:** Epic 1 - Admin GeoJSON/GPX Polyline Upload & Checkpoints Setup
- **FR-15:** Epic 1 - Multi-Tenant Data Persistence Schema (Mongoose)

## Epic List

### Epic 1: Multi-Tenant Core & Festival Administrative Management
Organizers can manage festival instances, upload route polylines, define ordered checkpoints with rich media, and public visitors can select active festivals from the landing page or access custom event URLs.
**FRs covered:** FR-1, FR-2, FR-13, FR-14, FR-15

### Epic 2: Live GPS Broadcasting Subsystem
Chariot volunteers can open a dedicated broadcaster interface (`/[eventSlug]/broadcast`), acquire continuous high-accuracy GPS streaming, retain active screen WakeLock to prevent device sleep, and buffer/flush GPS points during network drops.
**FRs covered:** FR-4, FR-5, FR-6
**NFRs covered:** NFR-3, NFR-4

### Epic 3: Devotee Real-Time Interactive Tracking Map & Route Progress
Devotees can view a mobile-optimized live map that auto-centers on the chariot's position via isolated Socket.io WebSocket rooms, manually recenter via a floating action button, and view a dynamic progress bar tracking route completion.
**FRs covered:** FR-3, FR-7, FR-8
**NFRs covered:** NFR-1, NFR-2, NFR-5

### Epic 4: Checkpoints, Cultural Media & Real-Time Announcements
Devotees can tap interactive checkpoint pins (*Upcoming*, *Resting*, *Passed*) to open rich cultural cards with photos and arrival times, and receive real-time admin announcements as top notification banners.
**FRs covered:** FR-9, FR-10, FR-11, FR-12

---

## Epic 1: Multi-Tenant Core & Festival Administrative Management

Festival administrators can manage festival instances, upload route polylines, define ordered checkpoints with rich media, and public visitors can select active festivals from the landing page or access custom event URLs.

### Story 1.1: Project Scaffold & Database Foundation

As a System Architect,
I want a Next.js 14 App Router codebase initialized with MongoDB/Mongoose models for Events, Routes, and Checkpoints,
So that multi-tenant festival metadata can be persisted and indexed reliably.

**Acceptance Criteria:**

**Given** a clean project workspace
**When** the server starts up
**Then** Next.js 14 App Router runs with Tailwind CSS v4 and MongoDB connects using a cached Mongoose singleton
**And** Mongoose schemas exist for `Event`, `Route`, `Checkpoint`, and `Announcement` indexed by `eventSlug` (AD-1, AD-2, AD-5, FR-15).

### Story 1.2: Root Festival Selector Landing Page & Multi-Tenant Slug Routing

As a Devotee,
I want to visit the root URL (`/`) to select an active festival or navigate directly to `/[eventSlug]`,
So that I am routed to my city's dedicated procession page (e.g. Itahari or Biratnagar).

**Acceptance Criteria:**

**Given** a user opening the web application at `/`
**When** active event instances exist in the database
**Then** event cards displaying city names, banner images, and dates are displayed (FR-2)
**And** clicking a festival card routes to `/[eventSlug]` (e.g. `/itahari`, `/biratnagar`), loading city-specific context and branding without cross-city data contamination (FR-1).

### Story 1.3: Admin Event Instance Management

As a Festival Administrator,
I want to create and manage Event Instances via `/admin` or REST endpoint `/api/v1/events`,
So that new Rath Yatra festival tracking pages can be deployed dynamically.

**Acceptance Criteria:**

**Given** an authorized festival administrator
**When** submitting a new event payload (title, slug, city, banner image URL, date)
**Then** a new Event document is stored in MongoDB (FR-13, FR-15)
**And** invalid payloads or duplicate slugs return structured HTTP validation errors.

### Story 1.4: Admin Route Polyline Upload & Checkpoint Placement

As a Festival Administrator,
I want to upload or input GeoJSON polylines and define ordered Checkpoints for an event instance,
So that the procession route and cultural stopping points are established.

**Acceptance Criteria:**

**Given** an existing Event Instance
**When** posting GeoJSON route coordinates to `/api/v1/routes` and checkpoint items (name, photo URL, description) to `/api/v1/checkpoints`
**Then** the route polyline and checkpoint sequence are saved and associated with `eventSlug` (FR-14, FR-15)
**And** public API GET endpoints `/api/v1/events/:slug/routes` and `/api/v1/events/:slug/checkpoints` return the stored GeoJSON and checkpoint array.

---

## Epic 2: Live GPS Broadcasting Subsystem

Chariot volunteers can open a dedicated broadcaster interface (`/[eventSlug]/broadcast`), acquire continuous high-accuracy GPS streaming, retain active screen WakeLock to prevent device sleep, and buffer/flush GPS points during network drops.

### Story 2.1: Broadcaster Geolocation Ingestion & Screen WakeLock

As a Chariot Broadcaster,
I want to open `/[eventSlug]/broadcast`, tap "Start Live Broadcast", enable screen WakeLock, and continuously acquire high-accuracy GPS coordinates,
So that the chariot's live location is continuously transmitted without the phone dimming or sleeping.

**Acceptance Criteria:**

**Given** an authorized volunteer on `/[eventSlug]/broadcast`
**When** tapping the "Start Live Broadcast" button
**Then** browser Geolocation API requests high-accuracy coordinates `(lat, lng, speed, timestamp)` every 3 seconds (FR-4)
**And** Screen WakeLock API (`navigator.wakeLock.request('screen')`) is engaged to prevent screen sleep (FR-5, AD-3)
**And** battery drain remains optimized under < 15% per hour (NFR-4).

### Story 2.2: Offline GPS Queueing & Socket.io Auto-Flush Protocol

As a Chariot Broadcaster,
I want location points to be queued in browser storage during cellular network dropouts and automatically flushed over Socket.io upon reconnect,
So that location updates are resilient to crowd network congestion.

**Acceptance Criteria:**

**Given** an active GPS broadcast session on `/[eventSlug]/broadcast`
**When** network connection drops during procession
**Then** failed location payloads are buffered in local storage (`localStorage`/`IndexedDB`) (FR-6)
**And** when WebSocket connectivity is re-established, buffered points are flushed sequentially over Socket.io to the server room `room:<eventSlug>` (FR-6, NFR-3).

---

## Epic 3: Devotee Real-Time Interactive Tracking Map & Route Progress

Devotees can view a mobile-optimized live map that auto-centers on the chariot's position via isolated Socket.io WebSocket rooms, manually recenter via a floating action button, and view a dynamic progress bar tracking route completion.

### Story 3.1: Public Leaflet Interactive Map & Socket.io Room Isolation

As a Devotee,
I want to open `/[eventSlug]` and view a real-time interactive Leaflet map that streams the chariot's position in room `room:<eventSlug>`,
So that I can see the chariot's current location with sub-3 second latency without data crossing over from other cities.

**Acceptance Criteria:**

**Given** a devotee on `/[eventSlug]` (e.g. `/itahari`)
**When** Leaflet map component initializes dynamically on client (`{ ssr: false }`) (AD-1)
**Then** Socket.io client joins room `room:<eventSlug>` (AD-2, FR-3)
**And** location updates sent by the broadcaster update the custom Rath icon marker on the map within < 3 seconds (NFR-1, NFR-2, NFR-5, FR-7).

### Story 3.2: Auto-Centering Map & Recenter Floating Action Button (FAB)

As a Devotee,
I want the map to automatically follow the chariot marker while allowing manual panning and offering a "Recenter on Chariot" button,
So that I can freely explore the map and quickly snap back to the active chariot position.

**Acceptance Criteria:**

**Given** an active tracking map on `/[eventSlug]`
**When** new GPS updates arrive and user has not interacted with map
**Then** map view auto-centers smoothy on the chariot marker (FR-7)
**When** user manually pans or zooms the map
**Then** auto-centering pauses and a "Recenter on Chariot" FAB appears; tapping the FAB re-centers map on the chariot and resumes auto-follow (FR-7).

### Story 3.3: Dynamic Route Polyline Progress Bar Calculation

As a Devotee,
I want to see a top progress bar showing the percentage (0% to 100%) of the route completed by the chariot,
So that I can instantly gauge how far along the procession is.

**Acceptance Criteria:**

**Given** incoming chariot GPS coordinates and an active route polyline
**When** position is received on client or server
**Then** distance along line is calculated using Turf.js `@turf/nearest-point-on-line` (AD-4)
**And** the UI renders a top progress bar smoothly updating from 0.0% to 100.0% (FR-8).

---

## Epic 4: Checkpoints, Cultural Media & Real-Time Announcements

Devotees can tap interactive checkpoint pins (*Upcoming*, *Resting*, *Passed*) to open rich cultural cards with photos and arrival times, and receive real-time admin announcements as top notification banners.

### Story 4.1: Interactive Checkpoint Map Pins & Status Indicators

As a Devotee,
I want checkpoint pins along the route map with distinct visual indicators for *Upcoming*, *Current/Resting*, and *Passed* statuses,
So that I can easily see which checkpoints the chariot has passed and where it is heading next.

**Acceptance Criteria:**

**Given** a route polyline with checkpoints
**When** the chariot progresses along the line
**Then** checkpoint map markers update visual state automatically (Upcoming = blue/default, Resting/Current = glowing gold, Passed = muted checkmark) (FR-9).

### Story 4.2: Rich Cultural Checkpoint Info Sheet

As a Devotee,
I want to tap any checkpoint pin on the map to open a slide-up sheet with photos, historical description, ETA, and status,
So that I can learn about the cultural significance and plan my visit.

**Acceptance Criteria:**

**Given** a devotee viewing the live map
**When** tapping a checkpoint pin
**Then** a responsive bottom sheet/modal slides up with photo, cultural title, history description, estimated arrival time, and current resting status (FR-10).

### Story 4.3: Admin Live Announcements & Notification Toast Banner

As a Festival Administrator,
I want to post real-time updates attached to checkpoints, and as a Devotee, receive these updates as top notification banners,
So that critical crowd announcements (e.g. "Prasad distribution started") reach all devotees instantly.

**Acceptance Criteria:**

**Given** an admin on `/[eventSlug]/admin`
**When** submitting a text announcement for a checkpoint
**Then** Socket.io broadcasts `server:announcement-posted` to room `room:<eventSlug>` (FR-11)
**And** all active devotee map screens display an animated notification toast banner with the announcement text (FR-12).


