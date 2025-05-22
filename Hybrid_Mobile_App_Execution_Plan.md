# Hybrid Mobile App Project Execution Plan 

## Team Composition
- **Vishal**: AI Integration
- **Tushar**: Calendar Integration
- **Kiran**: Notifications
- **Krishna**: DB Implementation
- **Rajkumar**: DB Implementation

## Technology Stack
- React (JavaScript)
- Vite (Build Tool)
- Capacitor (Wrapper)
- SQLite (Local Storage)
- SQLite VSS / Vec Plugin (Vector Search)
- OpenAI API (RAG Implementation)

---

## Day 1: Foundation Setup & Component Skeletons

### Vishal
- Research and document RAG architecture.
- Integrate OpenAI API (basic setup and test prompt).
- Draft placeholder UI for AI query box.

### Tushar
- Create basic calendar UI (month view with dummy entries).
- Evaluate Capacitor calendar-related plugins or native calendar options.
- Prepare modular calendar component.

### Kiran
- Design static notification bell UI component.
- Implement dummy notifications and popover logic.
- Ensure reusability with props for dynamic message injection.

### Krishna
- Setup SQLite database with Capacitor.
- Define initial schema.
- Insert mock data for test queries.

### Rajkumar
- Collaborate on schema finalization and seed data strategy.
- Begin abstraction of CRUD operations into reusable service.

---

## Day 2: Functional Integration & Local Persistence

### Vishal
- Build embedding generation pipeline using OpenAI.
- Store embeddings in SQLite with schema support.
- Prepare sample prompt context data for testing.

### Tushar
- Hook calendar to SQLite (read/write events).
- Enable add/update/delete functionality in calendar.
- Write helper functions for date formatting and DB sync.

### Kiran
- Link notifications to SQLite for persistent storage.
- Add logic for unread/read tracking.
- Configure event-based triggers (time or dummy actions).

### Krishna
- Develop generic database service (hook-based CRUD ops).
- Expose utilities for other components (notifications, calendar, AI).
- Validate performance with test queries.

### Rajkumar
- Integrate DB service into components.
- Write helper methods for complex filters and joins.
- Assist Krishna in debugging and optimization.

---

## Day 3: RAG AI, Final Integration, Testing

### Vishal
- Implement vector search (using SQLite VSS/Vec).
- Process user input, perform vector match, return result via OpenAI.
- Complete AI chat UI and context injection.

### Tushar
- Finalize calendar UI with styling.
- Test full calendar interaction (CRUD + data persistence).
- Add fallback handling (e.g., empty states).

### Kiran
- Polish notification component (styling, behavior).
- Implement configurable triggers and edge cases.
- Ensure smooth integration with main layout.

### Krishna
- Coordinate with Vishal in optimizing vector search queries.
- Create DB triggers for RAG events.
- Conduct full DB validation and test recovery from corrupt states.

### Rajkumar
- Perform end-to-end testing.
- Optimize slow queries.
- Create test dataset and unit test scripts.

---

## Outcome
- Working hybrid app with local-only persistence.
- Integrated calendar, notification, and AI smart assistant.
- Clean, modular, and extensible component architecture.
