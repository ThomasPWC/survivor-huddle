# Survivor Huddle

One shared NFL survivor entry, four people. Everyone ranks five teams (5 pts for #1 down to 1 pt for #5), picks are sealed until all four have submitted, then the combined ranking is revealed to everyone. Lines (spread, moneyline, total) load live from ESPN's public odds feed every time the page opens, and the week advances automatically.

Files:

- `index.html` — the whole app (one file, no build step)
- `database.rules.json` — Firebase Realtime Database rules that keep picks sealed until all four are in, then lock them
- `firebase.json` — Firebase Hosting + rules config, so one command deploys both

## One-time setup (about 10 minutes)

### 1. Create the free Firebase project

1. Go to https://console.firebase.google.com and click **Add project**. Name it anything (e.g. `survivor-huddle`). Google Analytics can be turned off.
2. In the left menu open **Build → Realtime Database → Create Database**. Pick any location, choose **Start in locked mode**, click Enable.
3. Copy the database URL shown at the top of the Data tab. It looks like `https://survivor-huddle-xxxxx-default-rtdb.firebaseio.com`.
4. Click the gear next to **Project Overview → Project settings**. Scroll to **Your apps**, click the **</>** (Web) icon, give it a nickname, skip Hosting for now, click Register app.
5. Copy the `firebaseConfig` object it shows you.

### 2. Paste the config into the page

Open `index.html`, find `const FIREBASE_CONFIG = {` near the top of the script, and replace the commented-out lines with your values. Make sure `databaseURL` is included (add it from step 3 if the console didn't show it):

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "survivor-huddle-xxxxx.firebaseapp.com",
  databaseURL: "https://survivor-huddle-xxxxx-default-rtdb.firebaseio.com",
  projectId: "survivor-huddle-xxxxx",
  appId: "1:1234:web:abcd"
};
```

Optionally fill in the four names in `DEFAULT_NAMES` on the next line so the slots are pre-labelled.

### 3. Publish the rules and the page

From this folder, in a terminal:

```bash
npx firebase-tools login
```

```bash
npx firebase-tools use --add
```

(pick the project you just made, alias it `default`), then

```bash
npx firebase-tools deploy
```

That uploads the database rules and hosts the page. The output ends with a **Hosting URL** like `https://survivor-huddle-xxxxx.web.app`. That's the link to text everyone.

Re-run `npx firebase-tools deploy` any time you change `index.html`. Nothing needs to change week to week.

### Alternative hosting

If you'd rather not use the Firebase CLI, the rules can be pasted by hand into **Realtime Database → Rules** in the console, and `index.html` can be hosted anywhere static (Netlify Drop, GitHub Pages, Cloudflare Pages). The page only needs the database, not Firebase Hosting.

## How it works for the group

- Each person opens the link on their phone and taps their name once. The page remembers who they are on that device.
- They tap teams in order of confidence, reorder if needed, and hit **Submit picks**. Unsubmitted picks auto-save on that device; submitted picks go to the shared database.
- Until all four have submitted, nobody can see anyone else's picks (enforced by the database rules, not just the UI). The board shows who's in and who's still out.
- Anyone can hit **Edit picks** to change their mind until the fourth person submits. After that, the week is locked.
- When the fourth ballot lands, everyone's page updates live: the group pick, the full point ranking, and each person's number on every team.
- The ◀ ▶ arrows next to the week let you look back at last week's result or ahead at next week's lines.

## Notes

- The Firebase free tier (Spark) is far more than enough for four people.
- The `apiKey` in a Firebase web config is not a secret; it identifies the project. Access is controlled by the rules file.
- Anyone with the link can pick any slot. That's fine for a friend group; if a stranger ever gets the link, just rotate to a new Firebase project.
