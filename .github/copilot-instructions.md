# Copilot Instructions for Mergington High School Activities API

## Project Overview

This is a lightweight FastAPI web application for managing extracurricular activities at Mergington High School. The system has a **three-layer architecture**:

- **Backend**: FastAPI REST API (`src/app.py`) serving activity and signup endpoints
- **Frontend**: Plain JavaScript/HTML/CSS client (`src/static/`) consuming the API
- **Data**: In-memory dictionary database (non-persistent) centered on activities

## Architecture & Key Design Patterns

### Data Model
- **Activities**: Keyed by activity name (not ID) — see `activities` dict in `app.py` lines 22-35
  - Each activity stores: `description`, `schedule`, `max_participants`, `participants` (list of emails)
- **Students**: Implicit; represented only by email strings in participant lists
- **Persistence**: All data is in-memory; resets on server restart

### API Design
- **Root redirect**: `GET /` redirects to `/static/index.html` (line 39)
- **Activity listing**: `GET /activities` returns the entire activities dictionary
- **Signup endpoint**: `POST /activities/{activity_name}/signup?email=student@mergington.edu`
  - Uses query parameter for email (not POST body)
  - Activity name in URL path must match dictionary key exactly
  - Error handling: Returns 404 HTTPException if activity not found

### Frontend Architecture
- **Single-page app** loaded from `index.html`; no routing beyond static file mounting
- **API interaction**: `app.js` (lines 9-25) handles all fetch calls
  - Activities are fetched on DOMContentLoaded
  - Form submission posts signup request and displays success/error messages for 5 seconds
- **UI rendering**: Activities populated into DOM as cards (line 20) and dropdown options (line 29)

## Critical Workflows

### Running the Application
```bash
cd src
python app.py
# Server runs on http://localhost:8000
# API docs available at http://localhost:8000/docs
```

### Testing
- Project uses pytest; configuration in `pytest.ini` sets `pythonpath = .`
- Run tests from workspace root: `pytest`
- No test files currently present in repo

### Dependencies
- Minimal stack: `fastapi`, `uvicorn` only (see `requirements.txt`)
- Install with: `pip install -r requirements.txt`

## Project-Specific Conventions

### Naming & Identifiers
- Activities are identified by **exact name string** (e.g., "Chess Club"), not numeric IDs
- This means signup URLs must URL-encode activity names with special characters: `encodeURIComponent()` in JS (line 61)
- Participant emails use `.edu` domain per school context

### Error Handling
- Backend uses FastAPI's `HTTPException` for missing resources (line 48)
- Frontend catches and displays errors with CSS class toggling: `.error` / `.success` (lines 65-70)
- Messages auto-hide after 5 seconds with `.hidden` class

### File Organization
- API logic and in-memory DB: `src/app.py`
- Static assets mounted at `/static` route; server expects them in `src/static/`
- Styles use **flexbox layout** with mobile-responsive media queries (CSS lines ~33-35)

## Integration Points & Data Flows

1. **Page Load**: Browser requests `/` → FastAPI redirects to `/static/index.html` → `app.js` DOMContentLoaded fires
2. **Activity Display**: `app.js` fetches `/activities` → receives JSON dict → renders cards + populates dropdown
3. **Signup Flow**: Form submit → POST to `/activities/{name}/signup?email=X` → backend appends email to participants array → success message displayed
4. **State Management**: No state management library; DOM is single source of truth for form values; API data fetched fresh on page load

## Common Modifications

- **Adding an activity**: Add entry to `activities` dict in `app.py` (lines 22-35) with required fields
- **Validation logic**: Currently missing; add to signup handler (line 46) for email format, duplicate signup check, max capacity
- **Persistence**: Replace in-memory dict with database (e.g., SQLAlchemy) for production use
- **Authentication**: Currently no auth; add JWT/session support to signup endpoint for student identity verification
