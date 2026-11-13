# FogRoute – Product Requirements Document (PRD)

**Team:** Vynora (Vishesh, Vaishnavi, Tulsi)\
**Status:** Hackathon prototype, working end to end

## 1. Overview

FogRoute is a  web app that helps people in Kanpur and nearby districts travel safely in winter fog. A user picks a start point, a destination, a date and a time. FogRoute then shows a fog-related risk score (0–100), the forecast visibility, humidity, temperature and air quality (AQI), an hourly risk strip, and a suggestion for a safer departure time. Commuters can also report fog on the ground, verify each other's reports, see everything on a live map, and reach emergency numbers in one tap.

Tagline: *Making Every Journey Visibility Aware.*

## 2. Problem Statement

North India gets dense fog every winter. Road accidents, pile-ups and delays rise sharply when visibility drops below a few hundred metres. Commuters currently have these problems:

- General weather apps show city-wide data, not locality-level road conditions.
- Visibility numbers alone do not tell a driver whether a trip is risky or when to leave.
- There is no easy way for commuters to share what they see on the road right now.
- During a roadside emergency in low visibility, people struggle to find the right helpline quickly.

## 3. Goals and Non-Goals

**Goals**

1. Give a clear, understandable risk level for a specific route at a specific time.
2. Suggest a better departure time when risk is high.
3. Combine forecast data with live community reports so the picture reflects the ground.
4. Make emergency help reachable in one tap, including a saved family contact.
5. Work well on a phone screen and on low-end devices, with no app install.

**Non-Goals **

- Turn-by-turn navigation (the app hands off to Google Maps for that).
- Coverage outside Kanpur and the nearby districts listed in the app.
- Official government or IMD warnings.
- Native Android/iOS apps and push notifications.

## 4. Target Users

| Persona | Need |
| --- | --- |
| Daily commuter (bike or car) | Know if the morning trip is safe and when to leave |
| Highway or intercity traveller | Check fog on the way to Lucknow, Unnao, Fatehpur, Etawah and similar |
| Family member of a traveller | Confidence that the traveller can reach help fast |
| Local volunteer or resident | Report and confirm fog so others benefit |

## 5. Key User Journeys

1. **Check a route:** Sign in → choose From and To → pick date and time (or tap *Use current time*) → tap *Check route* → read risk, hourly strip and tip.
2. **Avoid a hazard:** If dense fog is reported near the destination, a red alert appears and the user can open alternative routes in Google Maps.
3. **Report fog:** Reports tab → choose locality, optional landmark, intensity (Low / Medium / Dense) → submit. It is shared with all users.
4. **Verify a report:** Verify tab → vote *Fog still exists* or *Fog cleared* on a fresh report.
5. **Explore the map:** Map tab → see every locality coloured by live visibility and reports.
6. **Get help:** SOS tab → tap a helpline number, or the saved family contact, to open the dialer.

## 6. Functional Requirements

### 6.1 Accounts and Profile

- FR-1: Users can create an account with name, email and password (min 8 characters) and log in.
- FR-2: Fog data and reporting are visible only after login.
- FR-3: Profile shows name, email, member-since date and number of reports shared.
- FR-4: Users can change their display name; reports show this name.
- FR-5: Users can save, update and remove one family emergency contact (relation/name and phone).
- FR-6: Users can log out. An expired session sends the user back to login.

### 6.2 Route Risk Check

- FR-7: Users choose a start and a destination from 113 predefined localities (Kanpur areas plus nearby towns). Same start and destination is rejected.
- FR-8: Users choose a date (from 7 days back to 10 days ahead) and a time.
- FR-9: The app shows risk level (Low / Moderate / High), score out of 100, visibility, humidity, temperature and AQI.
- FR-10: The app shows an hourly risk strip around the selected time.
- FR-11: If the risk is not low, the app looks up to 12 hours ahead for the first time window that is low risk and suggests it.
- FR-12: Active Dense fog reports at the start or destination raise the score.
- FR-13: If a Dense, unresolved report exists at or within about 5 km of the destination, a high-risk alert is shown and the score is at least 78.
- FR-14: A safer alternate corridor (via a named hub) is suggested for the route and can be drawn on the map.
- FR-15: Each check is saved as a route inquiry for analytics.

### 6.3 Community Fog Reports

- FR-16: Users can submit a report with locality, optional landmark (max 50 characters) and intensity.
- FR-17: Reports are shared live with all users and refresh automatically about every 20 seconds.
- FR-18: Reports count as fresh for 3 hours; older ones are shown faded as old.
- FR-19: Reports can be filtered by All / Dense / Medium / Low.

### 6.4 Verification

- FR-20: Users can vote *Fog still exists* or *Fog cleared* on fresh reports, once per report.
- FR-21: A report is treated as cleared when cleared votes exceed still votes.
- FR-22: A vote bar shows the split between the two votes.

### 6.5 Live Map

- FR-23: Map shows all localities as markers coloured green (clear, 2 km or more), amber (mist or medium report) or red (dense fog, or visibility below 500 m).
- FR-24: Tapping a marker shows visibility, temperature, humidity and active report count.
- FR-25: Users can switch between Street, Dark and Satellite map styles.

### 6.6 Emergency SOS

- FR-26: One-tap dial links for 112, 108, 101, 1033 (NHAI highway) and 1073 (road accident and towing).
- FR-27: A large speed-dial card for the saved family contact; if none is saved, a prompt links to the profile.
- FR-28: A fog horn sound and a full-screen strobe beacon.
- FR-29: Short fog-driving safety tips.

### 6.7 Experience

- FR-30: Light and dark themes, remembered across visits.
- FR-31: Animated 3D scene in the background reflects current Kanpur visibility and weather.
- FR-32: Bottom tab bar on phones, top navigation on larger screens.

## 7. Risk Scoring Rules (Product View)

The score is a simple, explainable heuristic, not a scientific model.

| Input | Effect |
| --- | --- |
| Visibility 2 km or more | Base 5 |
| 1–2 km | Base 20 |
| 500 m–1 km | Base 50 |
| 200–500 m | Base 75 |
| Below 200 m | Base 92 |
| Humidity 90% or more | +5 |
| AQI above 200 / above 300 | +3 / +5 |
| Active Dense reports on the route's two ends | +8 each, max +16 |

Levels: below 35 is Low, 35–64 is Moderate, 65 and above is High. The score is capped at 100.

## 8. Non-Functional Requirements

- **Performance:** First screen usable within a few seconds on a mid-range phone over 4G.
- **Accessibility:** Keyboard focus outlines, labelled form fields, 44 px or larger touch targets, and a reduced-motion setting that switches off animations.
- **Responsiveness:** Layout works from small phones to desktop.
- **Reliability:** If weather data cannot be fetched, the app must not break. (See open issues for how this should be shown to users.)
- **Privacy:** Only necessary data is stored: account details, reports, route inquiries and an optional emergency contact.
- **Cost:** Runs on free tiers (vercel hosting, Supabase free plan, Open-Meteo).

## 9. Success Metrics

| Metric | Target for pilot |
| --- | --- |
| Registered users | 200+ in the first winter week |
| Route checks per active user per week | 3+ |
| Reports submitted per day in peak fog | 20+ |
| Share of reports with at least one vote | 40%+ |
| Users who save an emergency contact | 30%+ |

## 10. Risks and Open Issues

1. **Demo data fallback.** If the weather API fails, the app silently falls back to built-in sample numbers. Users must be told clearly, or the app should refuse to give a score.
2. **Alternate routes are rule-based.** Suggested corridors and their safety bonus are fixed values written in the code, not computed from live road data.

## 11. Roadmap

**Next (v1.1):** honest data labels, offline/failed-data banner, report rate limit, guest read-only mode, Hindi language option.\
**Later (v2):** real road-routing with fog exposure per segment, push alerts for saved routes, report photos, moderation tools, more cities.

## 12. Release Criteria

- Sign-up, login and logout work on Android Chrome.
- Route check works for any two localities with live data.
- Reports and votes appear for a second user within about 20 seconds.
- SOS links open the phone dialer.
- Database security rules are enabled on all tables.
