# ClassBook — Classroom Booking System

Node.js/Express backend + vanilla JS frontend. The backend also serves the frontend, so it deploys as **one service**.

```
classbook/
├── package.json          ← root: `npm install` + `npm start` work for deployment
├── backend/
│   ├── server.js         ← API + serves frontend, all security middleware
│   ├── config.js         ← reads environment variables
│   ├── database.js       ← in-memory data + timetable seed
│   └── .env.example      ← copy to .env for local use
└── frontend/public/      ← index.html, app.js, api.js, style.css
```

## Run locally

```bash
cd backend
npm install
node server.js        # open http://localhost:5000
```
(Optional: copy `.env.example` to `.env` and fill it.) Without a `.env`, the dev password for all demo accounts is `pass123`.

Old two-terminal way still works: backend on 5000, then `cd frontend && npm install && npx serve public -p 3000`.

## Deploy (Render / Railway / any Node host)

| Setting | Value |
|---|---|
| Build command | `npm install` |
| Start command | `npm start` |
| Env: `NODE_ENV` | `production` |
| Env: `JWT_SECRET` | long random string (`node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`) |
| Env: `SEED_PASSWORD` | password for all demo accounts (8+ chars, **required** in production) |
| Env: `SEMESTER_START` / `SEMESTER_END` / `SEMESTER_LABEL` / `HOLIDAYS` | optional — defaults are AY 2026-27 Sem I (6 Jul – 21 Nov 2026) and its holidays |

Use the host's HTTPS URL. Data is in memory, so it resets on every restart (same as before).

## Accounts

Teachers: `amk dsp sab psg kvw vki stb gmv spm gzz mam adl rjd smp npr ppr pap vsj kvj`
Students: `aiml_a1 aiml_a2 … aiml_f1 aiml_f2`
Password: whatever `SEED_PASSWORD` is set to (dev default `pass123`). Credentials are no longer shown on the login page.

## Security measures

- Login token lives in an **httpOnly, SameSite=Strict cookie** (+ `Secure` on HTTPS) — not in localStorage, not readable by JavaScript, not in the response body
- Logout revokes the token on the server; sessions expire after 8 hours
- JWT secret and demo password come from environment variables, nothing hard-coded
- Passwords stored only as bcrypt hashes; login timing doesn't reveal valid usernames
- Login rate limiting (8 failed tries / 15 min per account, 60 per IP) + general API rate limit
- Helmet security headers + strict Content-Security-Policy (no inline scripts / inline handlers)
- All dynamic text is HTML-escaped on the frontend (stops script injection via subject/names)
- Server-side input validation (types, lengths, dates, divisions, days, slots)
- Booking rules enforced on the server (IST): date must match the chosen weekday, no holidays, no past dates, and no slot that has already started today
- CSRF protection (origin check + SameSite cookie), CORS closed by default
- API responses are never cached; errors never leak stack traces
- Demo credentials removed from the login page, console output and served files

Note: your own browser's Inspect → Network tab will always show what *you* typed in the login request. That can't be hidden from the person typing it; **HTTPS** (automatic on hosting sites) is what protects it from everyone else.

## API

| Method | Route | Description |
|---|---|---|
| POST | /api/auth/login | Login (sets session cookie) |
| POST | /api/auth/logout | Logout (revokes session) |
| GET | /api/auth/me | Current user |
| GET | /api/rooms | List all rooms |
| GET | /api/rooms/availability?day=&slot= | Room availability for a slot |
| GET | /api/bookings | All bookings (optional `?day=`) |
| GET | /api/bookings/mine | My bookings (teacher) |
| GET | /api/bookings/weekly-count | Extra bookings used this week (teacher) |
| POST | /api/bookings | Create a booking (teacher) |
| DELETE | /api/bookings/:roomId/:day/:slot | Cancel a booking (teacher) |
| POST / DELETE | /api/waitlist | Join / leave waitlist (teacher) |
| GET | /api/waitlist/mine | My waitlist entries (teacher) |
| GET | /api/notifications | My notifications |
| PUT | /api/notifications/read-all | Mark all read |
| GET | /api/meta | Days, slots, semester info |
| GET | /api/health | Health check |
