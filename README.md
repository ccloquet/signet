# Signet

A minimal document-attestation tool. Each person registers a **function** (role/identity) and a personal password; from those, plus a date and a document reference, they can generate a 6-character code to write on a document. Anyone can later look up who a code belongs to — without ever learning the password.

This is a static site: two files, no server, no build step.

- `index.html` — the app
- `registry.json` — the shared list of registered people (starts empty: `{}`)

## How it works

- **Register**: derives a secret key from the password using PBKDF2 (the password itself is never stored or sent anywhere).
- **Sign**: computes `HMAC-SHA256(key, date + document reference)`, encoded as 6 alphanumeric characters.
- **Verify**: tries every entry in the registry and reports which one produces a matching code.

Because this is a fully static site (no backend), there's one manual step: when someone registers, the app shows a small JSON snippet for their entry. An admin adds that snippet into `registry.json` and pushes the change, so everyone's copy of the site can verify that person's codes. Until an entry is added to `registry.json`, it's only usable for verification on the same device/browser (it's kept in that browser's local storage).

## Deploy to GitHub Pages

1. Create a new GitHub repository (or use an existing one) and add `index.html` and `registry.json` to it.
2. Push to GitHub.
3. In the repo, go to **Settings → Pages**.
4. Under **Build and deployment**, set **Source** to "Deploy from a branch", pick the branch (e.g. `main`) and folder `/ (root)`, then save.
4. GitHub will publish the site at `https://<your-username>.github.io/<repo-name>/` within a minute or two.

To add someone to the shared registry later: open `registry.json`, add their `"slug": {...}` entry (comma-separated with existing entries), commit, and push. GitHub Pages redeploys automatically.

## Notes

- Works in any modern browser (uses the standard Web Crypto API) — no dependencies to install.
- Serve it over `https://` (which GitHub Pages does automatically) or `http://localhost` during local testing — the clipboard and crypto APIs used here require a "secure context" and won't work when opening `index.html` directly as a `file://` URL. Use e.g. `python3 -m http.server` in the folder to test locally.
- This attests who generated a code and when, tied to a document reference — it does not prove the document's content wasn't changed afterward, since the code doesn't depend on the file's contents, only on the reference string you type in (e.g. its filename).
