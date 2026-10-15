# SpeechWave Auth Pages

Server-rendered authentication pages for the SpeechWave platform. Handles email verification, password reset, and OAuth interstitial flows triggered by Appwrite.

## Tech Stack

| Technology | Version |
|------------|---------|
| **Astro** | ^5.16.15 |
| **Tailwind CSS** | ^4.1.18 |
| **@astrojs/node** | ^9.5.2 (SSR adapter) |
| **Appwrite JS SDK** | Loaded via CDN (`appwrite@17.0.0`) |

## Pages

### `/verification`

Email verification page. When a user clicks the verification link in their email, this page:

1. Extracts `userId` and `secret` from URL query parameters
2. Calls Appwrite `account.updateVerification(userId, secret)`
3. Shows loading → success → redirect flow
4. Displays error state if verification fails or link is invalid/expired

### `/reset-password`

Password reset page. When a user clicks the password recovery link in their email, this page:

1. Extracts `userId` and `secret` from URL query parameters
2. Presents a form for new password + confirmation
3. Calls Appwrite `account.updateRecovery(userId, secret, password)`
4. Redirects on success, shows error on failure

### `/oauth-success`

Google OAuth success interstitial. When Appwrite redirects here after a successful OAuth flow, this page:

1. Shows a short success state matching the other auth pages
2. Preserves the original OAuth token callback query string (`userId` and `secret`)
3. Redirects to `appwrite-callback-693016d000015836421d://oauth-success` so the Flutter app can create the session

### `/oauth-failure`

Google OAuth failure interstitial. When Appwrite redirects here after a failed OAuth flow, this page:

1. Shows a short failure state matching the other auth pages
2. Preserves the original callback query string
3. Redirects to `appwrite-callback-693016d000015836421d://oauth-failure` so the Flutter SDK receives the failed callback

### `/` (index)

Root index page (minimal).

## Project Structure

```
auth_speechwave/
├── src/
│   ├── pages/
│   │   ├── index.astro            # Root page
│   │   ├── verification.astro     # Email verification flow
│   │   ├── reset-password.astro   # Password reset form
│   │   ├── oauth-success.astro    # OAuth success interstitial
│   │   └── oauth-failure.astro    # OAuth failure interstitial
│   └── styles/
│       └── global.css             # Global Tailwind styles
├── public/                        # Static assets (favicon, etc.)
├── astro.config.mjs               # Astro config with Node SSR adapter + Tailwind
├── package.json
└── tsconfig.json
```

## Setup

```bash
npm install
npm run dev       # Start dev server at localhost:4321
npm run build     # Build for production
npm run preview   # Preview production build
```

## Notes

- Both auth pages use `export const prerender = false` for server-side rendering (dynamic routes).
- The Appwrite client is configured inline in each page's `<script>` block via the CDN import.
- OAuth interstitial pages do not call Appwrite directly; they preserve Appwrite's OAuth token callback params and return to the native app callback scheme.
- `auth.speechwaveai.com` must be added as an allowed Web platform hostname in the Appwrite project when using the OAuth interstitial URLs.
- Pages are styled with Tailwind CSS v4 for a clean, minimal UI.
