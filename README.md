# Heat Death Holdings

A pixel-art idle game. You build an energy empire up the Kardashev scale, from a hamster wheel to zero-point energy, then collapse the universe for dark matter and start again stronger. Players can sign in with Google to keep their save in the cloud.

**Play:** <https://imsoderp.github.io/heat-death-holdings/>

```
index.html           the whole game
firebase-config.js   this project's Firebase settings (public by design)
firestore.rules      who may read and write cloud saves
firebase.json        config for the Firebase CLI (optional)
.nojekyll            tells GitHub Pages to serve the files as they are
```

The game works without signing in: it saves in the player's browser every 5 seconds. Signing in also keeps a copy in Firebase.

---

## One-time setup

### 1. Turn on GitHub Pages

Repo **Settings → Pages → Build and deployment**: Source **Deploy from a branch**, Branch **main**, folder **/ (root)**, **Save**. After a minute or two the game is live at the link above. Each push to `main` updates it.

### 2. Firebase (project `heat-death-holdings`)

In <https://console.firebase.google.com>:

1. **Authentication → Sign-in method → Google → Enable**, choose a support email, **Save**.
2. **Authentication → Settings → Authorized domains → Add domain**: `imsoderp.github.io`
3. **Firestore Database → Create database**: production mode, a location near your players.
4. **Firestore Database → Rules**: replace everything with the contents of `firestore.rules`, then **Publish**.

### 3. Check it works

Open the game → **Settings** → **Sign in with Google**. The Cloud save section should say "Signed in as …", then "Cloud copy: universe #1 … saved just now". In the Firebase console, **Firestore Database → Data** should show a `saves` collection with one document.

---

## Bringing over a game from the Claude version

In the Claude version, open **Settings → Copy save code**. In this version, paste it into **Settings → Save codes** and click **Load save code** twice. Sign in afterwards and it goes to the cloud.

## How saving works

- The game saves in the browser every 5 seconds, always.
- Signed-in players also back up to the cloud every 2 minutes, right after a Big Crunch, and when they leave the tab.
- On opening the game (or returning to the tab), it compares the cloud copy with the browser's and keeps the one with more energy banked all-time. If the cloud copy replaces a browser game, the old one is kept, and **Settings → Undo last cloud load** brings it back.
- **Erase everything** clears both the browser and the cloud copy.

## Editing `firebase-config.js`

Keep it as a single `window.FIREBASE_CONFIG = { ... };` assignment. Don't paste the `import ... from "firebase/app"` snippet from the Firebase console into it: that code can't run in a plain script tag, and the game would report that cloud save isn't set up. The game loads the Firebase SDK and handles sign-in itself.

## Free-tier limits

On the free Spark plan Firestore includes **50,000 document reads and 20,000 writes per day**. An active player writes about 30 times an hour, so that covers roughly 600 hours of play per day across all players. If the daily quota runs out, cloud saves pause until it resets, and the game keeps saving in browsers and shows a note in Settings. To go further, move the project to the pay-as-you-go **Blaze** plan, or back up less often by changing `setInterval(cloudAutosave, 120000)` in `index.html` (milliseconds).

Current limits are on <https://firebase.google.com/pricing>.

## Privacy and security

- Saves contain only game progress. Players' Google names and emails stay in Firebase Authentication, which only the project owner can see.
- The Firestore rules let each signed-in player read and write only their own save (`saves/<their user id>`), cap its size, and close everything else.
- Optional hardening: to stop other websites from using this config, restrict the API key. In Google Cloud console go to **APIs & Services → Credentials**, open the browser key, set application restrictions to **Websites**, and list `imsoderp.github.io/*`, `heat-death-holdings.firebaseapp.com/*` (sign-in uses it) and `localhost/*` if you test locally.

## Testing on your own computer

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>. `localhost` is normally already an authorized domain in Firebase.

## Hosting on Firebase instead (optional)

```bash
npm install -g firebase-tools
firebase login
firebase use --add      # pick heat-death-holdings
firebase deploy
```

This serves the game at `https://heat-death-holdings.web.app` and also uploads `firestore.rules`.

## Troubleshooting

| What you see | Fix |
|---|---|
| "Cloud save isn't set up on this copy of the game." | `firebase-config.js` is missing, has a typo, or contains extra code. See "Editing `firebase-config.js`". |
| "Sign-in isn't enabled for this web address yet." | Add the domain under Authentication → Settings → Authorized domains (step 2). |
| "Google sign-in isn't switched on for this game yet." | Enable the Google provider (step 2). |
| "Your browser blocked the sign-in window." | Allow pop-ups for the site, then click Sign in again. |
| "The cloud refused this save." | Publish the rules from `firestore.rules` (step 2). |
| "The game's free cloud quota is used up for today." | See Free-tier limits above. |
| The site shows a 404 | Pages isn't on yet, or the first build is still running (step 1). |
