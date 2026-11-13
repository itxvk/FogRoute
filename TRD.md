# FogRoute – Technical Requirements Document (TRD)

**Team:** Vynora (Vishesh, Vaishnavi, Tulsi)

## 1. Purpose

This document explains how FogRoute is built, which services it depends on, how data flows, and what must be true for it to run safely in production. The product behaviour is described in the PRD.

## 2. Architecture Summary

FogRoute is a single-page app written in plain HTML, CSS and JavaScript. There is no custom server. The browser talks directly to a few hosted services.

| Layer | Technology | Role |
| --- | --- | --- |
| Front-end | One HTML file with inline CSS and JS | All pages, logic and styling |
| Routing | Hash based (`#home`, `#reports`, `#verify`, `#map`, `#sos`, `#profile`) | Switch pages without reload |
| Auth and database | Supabase (Auth + PostgREST + Postgres) | Accounts, reports, votes, profiles, route inquiries |
| Weather | Open-Meteo Forecast API | Visibility, temperature, humidity, weather code |
| Air quality | Open-Meteo Air Quality API | US AQI by hour |
| Maps | Leaflet 1.9.4 + Esri tile servers | Live map with Street, Dark and Satellite layers |
| 3D visuals | three.js r128 | Background scene, risk orb and SOS beacon |
| Fonts | Google Fonts (Bricolage Grotesque) | Headings |
| Hosting | Any static host (Render, Vercel, GitHub Pages, Netlify) | Serves the file |

**Request flow for a route check:**

1. User taps *Check route*.
2. App refreshes reports from Supabase.
3. App calls Open-Meteo Forecast and Air Quality for the midpoint of the two localities.
4. App computes the score, hourly strip, tip and alternate corridor in the browser.
5. App renders the result and saves a row in `route_inquiries` (fire and forget).

## 3. Front-end Design

### 3.1 Pages

One `<section class="page">` per page; the `go(page)` function toggles the `on` class, updates `aria-current` on navigation buttons, updates the URL hash and runs page-specific setup (map load, SOS beacon, profile refresh).

### 3.2 State

| Variable | Meaning |
| --- | --- |
| `PL` | Dictionary of 113 localities to `[lat, lon]` |
| `reps` | Fog reports loaded from the server |
| `votes` | Report ids this user already voted on (cached in localStorage) |
| `SESS` | Auth session: access token, refresh token, expiry, user |
| `VISM` | Live visibility, temperature and humidity per locality for the map |
| `SKY` | Current visibility, rain and storm flags for the 3D background |

### 3.3 Local storage keys

| Key | Content |
| --- | --- |
| `fr_sess` | Supabase session JSON |
| `fr_theme` | `light` or `dark` |
| `fr_vid` | Random visitor id |
| `fr_votes_<userId>` | Votes cast by that user |
| `fr_fam_<userId>`, `fr_fam_global` | Family emergency contact |

### 3.4 Timers

- Reports refresh every 20 seconds and when the tab becomes visible again.
- Home sky and live weather label refresh every 10 minutes.
- Report cards redraw every minute so ages stay current.

## 4. Core Logic

### 4.1 Weather request

- Point used: the average latitude and longitude of the start and destination.
- Forecast call asks for hourly `visibility`, `temperature_2m`, `relative_humidity_2m`, timezone `Asia/Kolkata`, from the day before to the day after the chosen date.
- Air-quality call asks for hourly `us_aqi`. If it fails, AQI defaults to 120.
- Values between two hours are linearly interpolated using the chosen minutes.

### 4.2 Risk score

```
base  = 5 (vis ≥ 2000 m) | 20 (≥ 1000) | 50 (≥ 500) | 75 (≥ 200) | 92 (< 200)
+5 if humidity ≥ 90
+5 if AQI > 300, else +3 if AQI > 200
+ min(16, 8 × count of active Dense reports at start or destination)
score = min(100, total)
level = Low (< 35) | Moderate (< 65) | High
```

If a fresh, uncleared Dense report is at the destination or within 0.045° (about 5 km), the score is raised to at least 78 and a hazard banner is shown.

### 4.3 Better time suggestion

When the level is not Low, the app checks each of the next 12 hours and suggests the first hour whose score is below 35. Otherwise it advises changing the time.

### 4.4 Alternate corridor

Four fixed corridors are defined in code (NH-19 southern bypass, VIP Road riverfront, Metro viaduct arterial, South Central ring), each with a list of hub localities and a fixed `safetyBonus`. For each hub the app:

1. Measures start→hub→destination distance against the direct distance.
2. Skips hubs where the detour ratio is above 2.2.
3. Adds a penalty of 5.0 if the hub has an active Dense report.
4. Picks the lowest penalty and shows `score − safetyBonus` (minimum 12) as the "alternate risk".

This is a rule-based suggestion. It is not computed from real road data or live weather along the alternate path.

### 4.5 Report freshness and status

- Loaded: reports from the last 24 hours, newest first, limit 200.
- Fresh: created within the last 3 hours.
- Cleared: `cleared_count > still_count`.
- Active report count on Home: fresh and not cleared.

### 4.6 Map colours

- Red `#B5483A`: any fresh active Dense report at the locality, or visibility below 500 m.
- Amber `#C98A1B`: any other fresh active report, or visibility below 2000 m.
- Green `#2F7D5E`: otherwise.

Visibility for all 113 localities is fetched in a single Open-Meteo call with comma-separated coordinates.

## 5. Backend: Supabase (inferred)

### 5.1 Tables

**`fog_reports`**

| Column | Type | Notes |
| --- | --- | --- |
| id | uuid or bigint, PK | Used in vote buttons |
| locality | text | One of the 113 names |
| landmark | text, nullable | Max 50 characters from the UI |
| intensity | text | `Low`, `Medium` or `Dense` |
| reporter\_name | text | Name shown on the card |
| user\_id | uuid, nullable | Should reference `auth.users` |
| created\_at | timestamptz | Default `now()` |
| still\_count | int, default 0 | Votes for *Fog still exists* |
| cleared\_count | int, default 0 | Votes for *Fog cleared* |

**`profiles`**

| Column | Type | Notes |
| --- | --- | --- |
| id | uuid, PK | Same as `auth.users.id` |
| name | text | Display name; source of truth for report names |
| emergency\_contact | text, nullable | Phone number |
| emergency\_contact\_name | text, nullable | Relation or name |

**`route_inquiries`**

| Column | Type |
| --- | --- |
| from\_locality, to\_locality, origin, destination | text |
| travel\_date | date |
| travel\_time | time or text |
| risk\_score | int |
| risk\_level | text |
| visibility\_m, humidity\_pct, temperature\_c, aqi | int |
| alternate\_route\_suggested | text, nullable |
| user\_id | uuid, nullable |

### 5.2 RPC

`cast_vote(rid, v)` is called with the report id and `'y'` or `'n'`. It should increment `still_count` or `cleared_count` and return `true` if the vote was recorded and `false` if this user had already voted. Duplicate-vote protection must live in the database (for example a `report_votes` table with a unique `(report_id, user_id)`), because the browser check can be bypassed.

### 5.3 Auth

- Email and password through Supabase Auth (`/auth/v1/signup`, `/token?grant_type=password`, `/token?grant_type=refresh_token`, `/logout`).
- Name stored in user metadata and in `profiles`. Emergency contact is also copied to user metadata as a backup.
- A new `profiles` row should be created automatically when a user signs up (database trigger).
- Email confirmation is currently turned off in Supabase, according to your project notes. The app still handles the case where confirmation is on.

### 5.4 External calls

| Service | Endpoint |
| --- | --- |
| Supabase REST | `/rest/v1/fog_reports`, `/rest/v1/profiles`, `/rest/v1/route_inquiries`, `/rest/v1/rpc/cast_vote` |
| Supabase Auth | `/auth/v1/...` |
| Open-Meteo | `api.open-meteo.com/v1/forecast`, `air-quality-api.open-meteo.com/v1/air-quality` |
| Tiles | `server.arcgisonline.com` (World Street Map, World Imagery, Dark Gray Canvas) |
| CDN | `cdnjs.cloudflare.com` (Leaflet, three.js), `fonts.googleapis.com` |
| Google Maps | Opens `google.com/maps/dir/` in a new tab for alternate routes |

## 6. Security

| Topic | Current state | Required action |
| --- | --- | --- |
| Supabase anon key in the HTML | Present. This is normal for Supabase, since the anon key is meant to be public. | Make sure **Row Level Security is ON for every table**. Without it, anyone with the key can read or change data. Never put the `service_role` key in front-end code. |
| Insert ownership | Client sends `user_id` | Add RLS policy so `user_id = auth.uid()` on insert; reject mismatches |
| Updates | Client can PATCH `profiles` and call `cast_vote` | Allow users to update only their own profile row; make `cast_vote` a `security definer` function that checks `auth.uid()` |
| Personal data | Emergency contact phone and email are stored | Restrict reads of `profiles` to the owner; do not expose it to other users |
| XSS | User text is escaped with `esc()` before insertion into HTML | Keep using it for every new field; consider moving to DOM text nodes |
| Session storage | Session JSON in localStorage | Acceptable for this scale; avoid adding third-party scripts that could read it |
| Spam and abuse | No limit on reports | Add rate limit (for example one report per user per locality per 10 minutes) in a database function or Edge Function |
| Third-party scripts | Loaded from cdnjs and Google Fonts | Pin versions (already pinned) and consider Subresource Integrity hashes |

## 7. Performance

- The file is about 100 KB before external libraries. three.js r128 is the heaviest dependency; the 3D background runs a continuous animation loop, which can drain battery on low-end phones.
- Recommended: lazy-load three.js only when WebGL is available and the user has not asked for reduced motion; pause the background loop when the tab is hidden; skip it when `prefers-reduced-motion` is set (the code already slows motion but still renders).
- Report polling every 20 seconds is acceptable for the prototype; use Supabase Realtime later to cut requests.
- The map makes one batched weather request for 113 points; cache it for 10 minutes.

## 8. Error Handling

| Situation | Current behaviour | Recommended |
| --- | --- | --- |
| Forecast API fails | Falls back to built-in sample numbers (visibility, humidity, temperature by hour) and continues | Show a clear banner "Live weather unavailable" and do not present a score, or label it as sample data |
| AQI API fails | Uses AQI 120 silently | Show "AQI unavailable" |
| Chosen hour missing in forecast | Message asking for a nearer date | Keep |
| Supabase request fails | Error text under the form or a toast | Keep; add retry |
| Session expired | Logs out with a message | Keep |
| Route inquiry save fails | Ignored | Acceptable |

## 9. Browser and Device Support

- Modern Chrome, Edge, Safari and Firefox with ES2017, `fetch`, `async/await`, `color-mix()`, `backdrop-filter`, CSS grid.
- WebGL needed for the 3D scenes. If three.js is not loaded the app still works.
- Web Audio is needed for the fog horn; failures show a toast.
- Touch targets are at least 44 px; bottom navigation appears below 820 px width.

## 10. Testing Plan

| Area | Test |
| --- | --- |
| Auth | Sign up, log in, wrong password message, log out, expired session |
| Route check | Same start and end rejected; past and future date limits; time interpolation; failed weather API; failed AQI API |
| Scoring | Unit-test `score()` with boundary values (200, 500, 1000, 2000 m; humidity 90; AQI 200 and 300) |
| Reports | Submit, appears for second user within about 20 s, expires after 3 hours, filters work |
| Votes | One vote per user per report; vote counts update; cleared status when cleared exceeds still |
| Map | Layer switching, marker colours, popup content |
| SOS | Each `tel:` link opens the dialer; family contact add, edit, remove |
| Security | Try reading another user's profile and inserting a report with someone else's `user_id`; both must fail |
| Accessibility | Keyboard navigation, screen reader labels, reduced-motion mode |

## 
