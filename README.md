<h1 align="center">Tracker</h1>

<p align="center">
  A local board that reads your Gmail and tracks your job applications.
</p>

<p align="center">
  <img src="docs/board.png" alt="The split view, with the applications grouped by status on the left and one of them open on the right" width="900">
</p>

## What It Does

Tracker groups the application emails in your inbox into one row per application.

- Sorts each application into Applied, In Progress, Accepted, or Rejected.
- Flags assessments, interviews, and checks as they arrive.
- Shows every email for an application in one place.
- Syncs only new mail after the first run.

It runs on your machine at <http://127.0.0.1:3939>. It only talks to Gmail and your AI provider.

## Installing

Requires [Node](https://nodejs.org) 22 or newer.

On Windows, double click `start.bat`. Everywhere else, run `npm run launch`. Either one installs,
builds, starts the server, and opens your browser.

Keep the server on `127.0.0.1`. It has no login, so anyone on your network could read your inbox.

## Setup

### 1. The Secret File

Copy `.env.example` to `.env.local` and set `APP_SECRET` to a random value:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Back up this file. Without it you will need to paste your API key in again.

### 2. A Google OAuth Client

Gmail access needs a Google Cloud project you own. It takes about fifteen minutes.

1. In the [Google Cloud Console](https://console.cloud.google.com/), create a new project.
2. Go to **APIs & Services** > **Library** and enable the **Gmail API**.
3. Go to **Google Auth Platform** > **Branding** > **Get started**. Enter a name and your email,
   choose **External**, and click **Create**.
4. Go to **Data access** > **Add or remove scopes** and manually add
   `https://www.googleapis.com/auth/gmail.readonly`. Then click **Update** and **Save**.
5. Go to **Audience** > **Publish app**. If you skip this, access expires every seven days.
6. Go to **Clients** > **Create client** > **Web application** and add this redirect URI exactly:

   ```
   http://127.0.0.1:3939/api/auth/google/callback
   ```

7. Copy the client ID and secret into `.env.local`, then restart the app:

   ```
   GOOGLE_CLIENT_ID="1234567890-abcdefg.apps.googleusercontent.com"
   GOOGLE_CLIENT_SECRET="GOCSPX-…"
   GOOGLE_REDIRECT_URI="http://127.0.0.1:3939/api/auth/google/callback"
   ```

### 3. Settings

Open Settings in the app and fill in:

- **Gmail Account.** Sign in. When Google says the app is not verified, click **Advanced** and
  continue. Tick the Gmail checkbox, or syncing will fail.
- **API Key.** From OpenRouter, Anthropic, or Gemini.
- **Read Emails From.** A start date, up to 12 months back.

Click **Save** to start the first sync. It reads about 250 emails and costs about 20 cents.

## Providers

Prices are per million tokens.

| Provider | Model | Input | Output |
| --- | --- | --- | --- |
| OpenRouter | `google/gemini-3.7-flash` | $0.375 | $1.875 |
| Anthropic | `claude-haiku-4-5` | $1.00 | $5.00 |
| Gemini | `gemini-3.7-flash` | $0.75 | $3.75 |

Settings shows your total spend.

## Privacy

Your mail and database stay on your machine. Your API key is stored encrypted. Gmail access is read
only.

To revoke access, use your [Google account's third party access page](https://myaccount.google.com/connections).

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `redirect_uri_mismatch` | Make the redirect URI match exactly. Use `127.0.0.1`, not `localhost`. |
| `invalid_client` | Copy the client ID and secret again, then restart. |
| `access_denied` | Publish the app (step 5). |
| `Gmail API has not been used in project …` | Enable the Gmail API (step 2). |
| `insufficient authentication scopes` | Log out, sign in again, and tick the Gmail checkbox. |
| **Reconnect Gmail** keeps showing | Sign in again. |
| Settings asks for `GOOGLE_CLIENT_ID` | Save `.env.local` and restart. |

## How The Code Is Organized

```
src/app/              Pages and API routes
src/components/       Board UI
src/lib/gmail/        Gmail client and email parsing
src/lib/llm/          One adapter per AI provider
src/lib/pipeline/     Sync: fetch, classify, and group emails into applications
prisma/               SQLite schema and migrations
scripts/              Launcher and check scripts
```

## Developing

```bash
npm install
npm run dev             # Starts the dev server
npm run check           # Runs typecheck, lint, and tests
```

CI runs `npm run check` and the build on every push and pull request.

Contributions are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md), and report vulnerabilities
through [SECURITY.md](./SECURITY.md).

## License

[MIT](./LICENSE)
