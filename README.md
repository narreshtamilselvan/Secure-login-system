# SecureGate — Secure Login System

A runnable cybersecurity internship project demonstrating account registration, Argon2id password hashing, SQLite parameterized queries, session authentication, CSRF protection, login rate limiting, TOTP authenticator-app 2FA, one-time recovery codes, and a dark security dashboard.

## Requirements

- Node.js 20 or newer
- npm
- An authenticator app (optional until enabling 2FA)

## Run locally

1. Extract the ZIP and open a terminal in the project folder.
2. Install dependencies:

   ```bash
   npm install
   ```

3. Create your local environment file:

   **Windows PowerShell**
   ```powershell
   Copy-Item .env.example .env
   ```
   **macOS/Linux**
   ```bash
   cp .env.example .env
   ```

4. Edit `.env` and set `SESSION_SECRET` to a unique random string of at least 32 characters. Generate one with:

   ```bash
   node -e "console.log(require('node:crypto').randomBytes(48).toString('hex'))"
   ```

   Put the generated value in `.env`; never commit `.env` to Git.

5. Start the application:

   ```bash
   npm start
   ```

6. Open `http://127.0.0.1:3000` in your browser. Register an account, sign in, open **2FA**, and follow the QR-code setup flow.

For development auto-restart: `npm run dev`.

## Main routes

| Route | Purpose |
| --- | --- |
| `/` | Project overview |
| `/register` | Create an account |
| `/login` | Sign in |
| `/2fa/verify` | Second-factor verification |
| `/dashboard` | Protected account dashboard |
| `/security/2fa` | Configure or disable 2FA |
| `POST /logout` | Destroy server-side session |

## Security implementation

- **Passwords:** Argon2id hashes are stored; plaintext passwords are never persisted.
- **SQL injection:** All dynamic SQL values use SQLite prepared/parameterized statements.
- **Sessions:** Server-side SQLite session store, HttpOnly/SameSite cookies, session regeneration after authentication, and session destruction on logout.
- **CSRF:** Synchronizer token checked on every state-changing POST.
- **Rate limits:** Separate limits for login, registration, and second-factor verification.
- **2FA:** TOTP secret setup, QR code, verification, and eight cryptographically random one-time recovery codes. Recovery codes are stored as Argon2id hashes and consumed once.
- **Headers:** Helmet security headers and a restrictive Content Security Policy.
- **Audit:** Basic authentication and 2FA events recorded in SQLite.

## Test

```bash
npm test
```

The included Node test suite checks recovery-code generation and normalization. After installing dependencies, run the app and manually verify:
1. Invalid/short password registration is rejected.
2. Duplicate username/email registration is rejected.
3. Wrong password does not authenticate.
4. Dashboard redirects when signed out.
5. POST requests without a valid CSRF token are rejected.
6. Login attempts are rate-limited.
7. Enabling 2FA requires a valid TOTP code.
8. Recovery codes work once and then fail.
9. Logout prevents access to protected pages.

## Database

The app creates the configured SQLite database and tables automatically on first launch. The default path is `./data/app.sqlite`. The session store creates a separate `sessions.sqlite` file in the same directory.

## Production hardening checklist

This is an educational starter, not a third-party security audit or a production certification.

- Serve only behind HTTPS in production; set `NODE_ENV=production` so session cookies use `Secure`.
- If behind a trusted reverse proxy, configure `TRUST_PROXY=1` only when the proxy is correctly configured.
- Use a strong secret manager for `SESSION_SECRET`; rotate it with an intentional session invalidation plan.
- Back up databases securely and restrict filesystem permissions.
- Add account recovery/change-password workflows, email verification, password breach checks, and administrator controls before real deployment.
- Consider per-account login throttling and monitoring; IP-based rate limits alone do not stop distributed attacks.
- For higher assurance, encrypt TOTP secrets at rest using a separately managed encryption key.
- Use a mature CSRF library and add integration tests for all routes before production use.
- Do not expose this development server directly to the public internet.
- Protect the recovery-code display flow from screenshots/logging and advise users to store codes securely.

## Important notes

- The app intentionally uses generic login failure messages.
- The TOTP setup key is displayed only during setup; keep it private.
- Recovery codes are shown once and cannot be retrieved later.
- A successful test suite covers only the assertions in that suite; it does not prove the application is free of vulnerabilities.
