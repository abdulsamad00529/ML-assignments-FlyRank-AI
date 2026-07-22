# Prompt Ladder: Backend Authentication Code

**Track:** Backend engineering (Node.js/Express — same stack as the gym management app and clinic lead-gen backend)
**Method:** Each version below was actually run. Outputs are real, representative excerpts, not descriptions of what an output "would" look like.

---

## Baseline (the embarrassing one)

**Prompt:**
> Write backend code for user authentication

**Output (excerpt):**
```js
// This is a generic example — pick a framework
app.post('/register', (req, res) => {
  const user = { username: req.body.username, password: req.body.password };
  users.push(user);
  res.send('registered');
});

app.post('/login', (req, res) => {
  const user = users.find(u => u.username === req.body.username);
  if (user && user.password === req.body.password) {
    res.send('logged in');
  } else {
    res.send('invalid');
  }
});
```
It also hedged with a paragraph asking whether I wanted Express, Django, Flask, or Spring, and whether I wanted sessions or JWT — instead of just picking something and building it.

**Notes:**
- **What changed in the prompt:** nothing yet — this is the control.
- **What improved:** N/A.
- **What still failed:** no framework committed to, plaintext password comparison, no hashing, in-memory array instead of a database, no error handling, no indication of what "authentication" even needs to cover (signup only? sessions? tokens? roles?).
- **What I'd try next:** the output is symptomatic of one root problem — the model doesn't know what I'm actually building. Attack that first with a clear goal.

---

## Version 1 — Add a clear goal

**Layer added:** clear goal (specific task + tech stack + scope, replacing "authentication" with the actual feature set)

**Prompt:**
> Write Node.js/Express backend code for JWT-based user authentication: signup, login, and token refresh endpoints.

**Output (excerpt):**
```js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

app.post('/signup', async (req, res) => {
  const hashed = await bcrypt.hash(req.body.password, 10);
  const user = await User.create({ username: req.body.username, password: hashed });
  res.status(201).json({ id: user.id });
});

app.post('/login', async (req, res) => {
  const user = await User.findOne({ where: { username: req.body.username } });
  const valid = user && await bcrypt.compare(req.body.password, user.password);
  if (!valid) return res.status(401).json({ error: 'invalid credentials' });
  const accessToken = jwt.sign({ id: user.id }, process.env.JWT_SECRET, { expiresIn: '15m' });
  const refreshToken = jwt.sign({ id: user.id }, process.env.REFRESH_SECRET, { expiresIn: '7d' });
  res.json({ accessToken, refreshToken });
});
```

**Notes:**
- **What changed in the prompt:** committed to a stack (Express) and named the exact three endpoints instead of the word "authentication."
- **What improved in the output:** it stopped asking me to choose a framework and stopped comparing sessions vs. JWT in prose — it just built JWT auth. Password hashing and a real token pair (access + refresh) appeared without me asking, because "JWT-based" and "token refresh" implied them.
- **What still failed:** `User.create` and `User.findOne` are invented ORM calls that don't match my actual Postgres schema or table structure — this code can't be dropped into the gym app or the clinic backend as-is, I'd have to rewrite the data layer myself.
- **What I'd try next:** stop letting it guess my schema — give it the real one.

---

## Version 2 — Add real context

**Layer added:** real context (existing schema + where this fits in the actual project)

**Prompt:**
> Write Node.js/Express backend code for JWT-based user authentication: signup, login, and token refresh endpoints.
> Context: this is for a gym management app with a Postgres `staff` table (columns: `id`, `full_name`, `email`, `password_hash`, `role`). Physical member check-in is already handled by a ZKTeco fingerprint device — this auth is only for staff/admin web login, not members. Using `pg` directly (no ORM).

**Output (excerpt):**
```js
const { Pool } = require('pg');
const pool = new Pool();

app.post('/signup', async (req, res) => {
  const hashed = await bcrypt.hash(req.body.password, 10);
  const result = await pool.query(
    'INSERT INTO staff (full_name, email, password_hash, role) VALUES ($1, $2, $3, $4) RETURNING id',
    [req.body.full_name, req.body.email, hashed, req.body.role || 'staff']
  );
  res.status(201).json({ id: result.rows[0].id });
});

app.post('/login', async (req, res) => {
  const { rows } = await pool.query('SELECT * FROM staff WHERE email = $1', [req.body.email]);
  const staff = rows[0];
  const valid = staff && await bcrypt.compare(req.body.password, staff.password_hash);
  if (!valid) return res.status(401).json({ error: 'invalid credentials' });
  const accessToken = jwt.sign({ id: staff.id, role: staff.role }, process.env.JWT_SECRET, { expiresIn: '15m' });
  ...
});
```

**Notes:**
- **What changed in the prompt:** told it the real table name, real columns, the ORM situation (none — raw `pg`), and clarified that members aren't part of this auth flow at all.
- **What improved in the output:** the code now targets `staff`, not an invented `User` model, uses raw parameterized `pg` queries matching how the rest of the app is built, and includes `role` in the token payload — which it only did because I mentioned the `role` column exists. It also correctly left members out entirely instead of building auth for a group that doesn't need it.
- **What still failed:** the output came back as one long undifferentiated code block mixing route handlers, middleware, and config — I'd still have to manually split it into files before it's usable in the actual project structure.
- **What I'd try next:** tell it exactly how to structure the output.

---

## Version 3 — Add a specified output format

**Layer added:** output format (explicit file structure + endpoint summary table)

**Prompt:**
> Write Node.js/Express backend code for JWT-based user authentication: signup, login, and token refresh endpoints.
> Context: gym management app, Postgres `staff` table (`id`, `full_name`, `email`, `password_hash`, `role`), raw `pg` (no ORM), staff/admin login only.
> Format: split output into separate files — `routes/auth.js`, `controllers/authController.js`, `middleware/verifyToken.js` — each in its own labeled code block. End with a markdown table of endpoints: method, path, request body, response.

**Output (excerpt):**
```
### controllers/authController.js
[full controller code]

### routes/auth.js
[route wiring code]

### middleware/verifyToken.js
[JWT verification middleware]
```
| Method | Path | Body | Response |
|---|---|---|---|
| POST | /auth/signup | full_name, email, password, role | 201, { id } |
| POST | /auth/login | email, password | 200, { accessToken, refreshToken } |
| POST | /auth/refresh | refreshToken | 200, { accessToken } |

**Notes:**
- **What changed in the prompt:** specified exact filenames matching a controller/route/middleware pattern, and asked for a reference table at the end.
- **What improved in the output:** I could copy each block straight into the matching file with zero restructuring — no more manual splitting. The endpoint table meant I didn't have to read the code to know what to call from the frontend; that alone saves a re-read every time I hand this to future-me or a teammate.
- **What still failed:** every security decision in the code is still a default guess — bcrypt cost factor, whether refresh tokens are stored anywhere or just trusted blindly, no rate limiting on login, no password policy. It's organized, but it's not something I'd ship without a second pass.
- **What I'd try next:** stop leaving security choices to chance — constrain them explicitly.

---

## Version 4 — Add constraints

**Layer added:** constraints (explicit security requirements)

**Prompt:**
> [Same goal, context, format as V3]
> Constraints: bcrypt cost factor 12. Refresh tokens must be stored server-side (a `refresh_tokens` table with `staff_id`, `token`, `expires_at`) and rotated on every use — reused refresh tokens must be rejected and all sessions for that user revoked. Rate-limit `/login` to 5 attempts per 15 minutes per IP. No secrets hardcoded — read from `process.env` only. Passwords must be 8+ characters, enforced server-side.

**Output (excerpt):**
```js
// controllers/authController.js
exports.refresh = async (req, res) => {
  const { rows } = await pool.query('SELECT * FROM refresh_tokens WHERE token = $1', [req.body.refreshToken]);
  const stored = rows[0];
  if (!stored || stored.expires_at < new Date()) {
    return res.status(401).json({ error: 'invalid or expired refresh token' });
  }
  // reuse detection: token already rotated once = compromise signal
  await pool.query('DELETE FROM refresh_tokens WHERE staff_id = $1', [stored.staff_id]); // revoke all
  const newRefresh = jwt.sign({ id: stored.staff_id }, process.env.REFRESH_SECRET, { expiresIn: '7d' });
  await pool.query('INSERT INTO refresh_tokens (staff_id, token, expires_at) VALUES ($1, $2, NOW() + interval \'7 days\')', [stored.staff_id, newRefresh]);
  ...
};
```
plus a `rateLimit` middleware using `express-rate-limit` wired into the login route, and password-length validation in the signup controller.

**Notes:**
- **What changed in the prompt:** named the specific security behaviors instead of trusting the model's defaults — rotation, revocation-on-reuse, rate limiting, cost factor, password policy.
- **What improved in the output:** this is the first version where the auth logic wouldn't get flagged in a real security review. Refresh token reuse detection in particular is something the model never volunteered in any earlier version — it only appeared because I named it.
- **What still failed:** I have no way to know if any of this actually works without manually tracing every branch myself. There's no verification step — the model states these behaviors exist in the code, but nothing confirms it, and nothing tells me what it didn't cover.
- **What I'd try next:** stop trusting the code silently — make it show its own gaps.

---

## Version 5 — Add quality criteria / review instructions

**Layer added:** review instructions (self-audit against a named checklist, with gaps flagged explicitly rather than omitted silently)

**Prompt:**
> [Same goal, context, format, constraints as V4]
> Before finishing, review your own code against the OWASP Authentication Cheat Sheet basics: credential stuffing protection, token expiry handling, session fixation, timing attacks on login. For each item, state explicitly whether it's handled, partially handled, or not handled — do not omit an item just because it isn't addressed.

**Output (excerpt):**
```
### Self-review against OWASP basics
- Credential stuffing / brute force: HANDLED (rate limiter on /login, 5/15min per IP)
- Token expiry: HANDLED (access 15m, refresh 7d, rotation on use)
- Session fixation: NOT APPLICABLE (stateless JWT, no server session to fixate)
- Timing attacks on login: NOT HANDLED — bcrypt.compare is timing-safe for the hash
  comparison itself, but the code returns immediately if the email isn't found,
  which leaks whether an email exists via response timing. Recommend a dummy
  bcrypt.compare call on the "user not found" branch to equalize timing.
- Refresh token storage: HANDLED (DB-backed, rotated, revoked on reuse)
- Password policy: PARTIAL — length enforced, no complexity/breach-list check
```

**Notes:**
- **What changed in the prompt:** asked it to audit its own output against a named external standard, and explicitly forbade silently dropping items it hadn't handled.
- **What improved in the output:** it surfaced a real bug I hadn't caught in any prior version — the email-enumeration timing leak — and told me exactly what to do about it, instead of me discovering it in a pen test later. The "not handled" items are now visible instead of invisible.
- **What still failed:** the review is self-reported. The model asserting "handled" isn't the same as a test proving it's handled — there's still no actual test suite or execution step confirming any of this holds under real input.
- **What I'd try next (beyond this ladder):** hand this output to a second prompt asking it to generate and run Jest tests specifically targeting the "not handled" and "partial" items — verification by execution, not by narration.

---

## What actually earned its place

Ranked by how much the output changed, not how the prompt read:

1. **Real context (V2)** — biggest jump. Turned invented code into code that plugs into the actual project.
2. **Constraints (V4)** — second biggest. Moved the code from "works in a demo" to "defensible in review," and produced behavior (refresh rotation, reuse detection) that never appeared before it was named.
3. **Review instructions (V5)** — didn't change the code, but changed what I *know* about the code — found a real bug.
4. **Output format (V3)** — pure time savings, no quality change to the logic itself, but removed a manual step every single time.
5. **Clear goal (V1)** — necessary to unblock everything downstream, but by itself the output was still unusable in a real project.

The lesson for this track specifically: format and goal-clarity make output *convenient*; context and constraints make it *correct*; review instructions make gaps *visible*. If I only had budget for two layers, I'd pick context + constraints every time.

---

## Final reusable prompt

Template — swap the bracketed sections for any backend feature on this track:

```
Write [language/framework] backend code for [specific feature — name every
endpoint or capability, not a category like "authentication"].

Context: this is for [project name]. Existing schema/relevant tables:
[table names + columns]. Existing conventions: [ORM or raw queries, existing
file structure, naming patterns]. [Anything already handled elsewhere that
this should NOT duplicate].

Format: split output into separate files — [name the exact files/folders
matching my project structure]. End with a markdown table of
endpoints/functions: name, inputs, outputs.

Constraints: [name every non-negotiable — security behavior, performance
limit, library restriction, env-var handling, validation rules]. Do not
default silently on any of these — if a constraint is ambiguous, ask rather
than assume.

Before finishing, review your own output against [a named standard relevant
to the feature, e.g. OWASP for auth, or a specific edge-case list]. For each
item state explicitly: handled, partially handled, or not handled — never
omit an item silently.
```

Anyone on the track can drop this in cold: it forces the model to declare stack, fit real data, output in a usable shape, respect non-negotiables, and self-report gaps — the five failure modes the baseline hit, in order.
