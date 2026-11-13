# 🌫️ FogRoute

**Making every journey visibility aware.**

FogRoute is a web app for Kanpur and nearby districts that helps you plan travel in winter fog. Pick a route and a time, get a fog-risk score with visibility, humidity, temperature and air quality, see a better time to leave, and check live fog reports from other commuters on a map.

Built by **Team Vynora** (Vishesh, Vaishnavi, Tulsi) using plain HTML, CSS and JavaScript.

---

## ✨ Features

- **Route risk check:** choose a start, a destination, a date and time. Get a risk level (Low / Moderate / High) with a score out of 100.
- **Hourly risk strip** around your chosen time, plus a suggestion for the next low-risk hour.
- **Safer alternate route** suggestion via a named corridor hub, viewable on the map, with a button to open alternatives in Google Maps.
- **Destination fog alert** when a dense-fog report is active near where you are going.
- **Community fog reports:** report Low, Medium or Dense fog for any locality, with an optional landmark. Reports are shared live.
- **Verify tab:** vote *Fog still exists* or *Fog cleared*. Reports older than 3 hours are shown as old.
- **Live fog map** of 113 localities, coloured by visibility and reports, with Street, Dark and Satellite views.
- **Emergency SOS:** one-tap calls to 112, 108, 101, 1033 and 1073, a saved family emergency contact, a fog horn sound and a strobe beacon.
- **Accounts and profile:** sign up, log in, change your name, see how many reports you shared.
- **Light and dark themes**, a 3D animated sky that follows current Kanpur visibility, and a reduced-motion friendly design.

## 🧰 Tech Stack

| Part | Technology |
|---|---|
| Front-end | HTML, CSS, vanilla JavaScript (single file) |
| Backend | [Supabase](https://supabase.com) (Auth, Postgres, REST) |
| Weather and AQI | [Open-Meteo](https://open-meteo.com) forecast and air-quality APIs |
| Map | [Leaflet](https://leafletjs.com) 1.9.4 with Esri tiles |
| 3D visuals | [three.js](https://threejs.org) r128 |
| Font | Bricolage Grotesque (Google Fonts) |

## 🚀 Getting Started

### Run locally

1. Download or clone this repo.
2. Open `index.html` in a browser. On Android you can open it from a file manager or use a code editor's preview.
3. You need an internet connection for weather, maps, fonts and Supabase.

A simple local server also works:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

### Set up your own Supabase backend

1. Create a project at [supabase.com](https://supabase.com).
2. Create these tables (see `TRD` for columns):
   - `fog_reports`
   - `profiles`
   - `route_inquiries`
3. Create a function `cast_vote(rid, v)` that records one vote per user per report and returns `true` or `false`.
4. Add a trigger that creates a `profiles` row for every new user.
5. **Turn on Row Level Security for every table** and add policies so users can only change their own data.
6. In the code, replace the two values near the top of the script:

   ```js
   var SB  = 'https://YOUR-PROJECT.supabase.co';
   var KEY = 'YOUR-ANON-PUBLIC-KEY';
   ```

   Use only the **anon (public)** key. Never put the `service_role` key in this file.
7. In Supabase → Authentication → URL Configuration, add the address where your site is hosted.
8. Email confirmation can be turned off for testing. If it is on, new users must confirm their email before logging in.

### Deploy

It is a static site, so any static host works.

- **Render:** New → Static Site → connect the repo → publish directory `.`
- **Vercel / Netlify:** import the repo, no build command needed.
- **GitHub Pages:** enable Pages on the main branch.

## 📁 Project Structure

```
fogroute/
├── index.html    # app (HTML, CSS and JS inline)
├── favicon.ico
├── README.md
├── PRD.md          # product requirements
└── TRD.md          # technical requirements
 

```

Inside the script, the main parts are:

| Function / variable | What it does |
|---|---|
| `PL` | All 113 localities with coordinates |
| `score()` | Computes the fog risk score |
| `getAlternateRoute()` | Picks a safer corridor hub |
| `loadReps()` | Loads reports from Supabase |
| `loadMap()` / `drawMap()` | Builds the live map |
| `initSky()` | 3D background scene |
| `submitAuth()` | Sign up and log in |

## 🧮 How the Risk Score Works

| Input | Effect |
|---|---|
| Visibility 2 km+ / 1–2 km / 500 m–1 km / 200–500 m / under 200 m | Base 5 / 20 / 50 / 75 / 92 |
| Humidity 90% or more | +5 |
| AQI above 200 / above 300 | +3 / +5 |
| Active Dense reports at start or destination | +8 each (max +16) |

Below 35 is **Low**, below 65 is **Moderate**, otherwise **High**. If a Dense report is active near the destination, the score is at least 78.

> This is a simple heuristic to help you decide, not a scientific forecast. Open-Meteo visibility is a model estimate, and community reports can be wrong.

## 🔐 Security Notes

- The Supabase anon key in the front-end is public by design. Your data is protected only by **Row Level Security**, so make sure it is enabled.
- User-entered text is escaped before being shown.
- Emergency contact numbers are personal data. Allow only the owner to read their own profile row.

## 🙏 Credits

- Weather and air quality: [Open-Meteo](https://open-meteo.com)
- Map tiles: Esri
- Maps library: Leaflet
- 3D: three.js

## 👥 Team

**Team Vynora** — Vishesh · Vaishnavi · Tulsi

© 2026 Team Vynora
