# Survivor Huddle

**Live:** https://thomaspwc.github.io/survivor-huddle/

One shared NFL survivor entry, four people. Everyone ranks five teams (5 pts for #1 down to 1 pt for #5), picks are sealed until all four have submitted, then the combined ranking is revealed. Lines (spread, moneyline, total) load live from ESPN's public odds feed every time the page opens, and the week advances automatically.

Static single-page site, hosted on GitHub Pages from the `gh-pages` branch. Personal project. No company infrastructure.

Files:

- `index.html` — the whole app (one file, no build step)
- `database.rules.json` — Firebase Realtime Database rules that keep picks sealed until all four are in, then lock them
- `firebase.json` — optional, only if you ever want to deploy the rules with the Firebase CLI instead of pasting them

## Deploy

Edit `index.html`, commit, and push the same commit to both branches:

```sh
git push origin master
git push origin master:gh-pages
```

GitHub Pages redeploys within a minute or two. Nothing needs to change week to week; the page pulls the current week and lines on its own.

## Turning on group sharing (one-time, about 10 minutes)

Until this is done the page runs in local-only mode: everything works but picks stay on each person's own device and a yellow banner says so. Sharing needs a free Firebase Realtime Database.

### 1. Create the free Firebase project

1. Go to https://console.firebase.google.com and click **Add project**. Name it anything (e.g. `survivor-huddle`). Google Analytics can be turned off.
2. In the left menu open **Build → Realtime Database → Create Database**. Pick any location, choose **Start in locked mode**, click Enable.
3. On the **Rules** tab, replace the contents with the contents of `database.rules.json` from this repo and click **Publish**.
4. Copy the database URL shown at the top of the **Data** tab. It looks like `https://survivor-huddle-xxxxx-default-rtdb.firebaseio.com`.
5. Click the gear next to **Project Overview → Project settings**. Scroll to **Your apps**, click the **</>** (Web) icon, give it a nickname, skip Hosting, click Register app.
6. Copy the `firebaseConfig` object it shows you.

### 2. Paste the config into the page

Open `index.html`, find `const FIREBASE_CONFIG = {` near the top of the script, and replace the commented-out lines with your values. Make sure `databaseURL` is included (add it from step 4 if the console didn't show it):

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

### 3. Push

Commit and push to both branches as in **Deploy** above. The banner disappears and the four of you are sharing one board.

## How it works for the group

- Each person opens the link on their phone and taps their name once. The page remembers who they are on that device.
- They tap teams in order of confidence, reorder if needed, and hit **Submit picks**. Unsubmitted picks auto-save on that device; submitted picks go to the shared database.
- Until all four have submitted, nobody can see anyone else's picks (enforced by the database rules, not just the UI). The board shows who's in and who's still out.
- Anyone can hit **Edit picks** to change their mind, before or after the reveal. Before the reveal the ballot is rewritten in place and stays sealed. After the reveal (the rules lock the week's `picks` once all four `submitted` stamps exist) an edit is saved as its own mini-week `weeks/<key>-r<slot>-<n>` holding `picks/0` plus four `submitted` stamps, which is what makes it readable to everyone. The latest revision wins, the board marks that person ✎ edited, and there's a cap of 4 edits per person per week.
- When the fourth ballot lands, everyone's page updates live: the group pick, the full point ranking, and each person's number on every team.
- The ◀ ▶ arrows next to the week let you look back at last week's result or ahead at next week's lines.
- **Our picks so far**: the sidebar lists each week and the team the group actually submitted. Those teams are crossed out in the matchups and can't be picked again. After a week reveals, the results card offers a one-tap **Mark X as our Week N pick**; or tap **Mark pick / Change** on any week row and pick from the team grid. Picks are shared through the database at `weeks/pick-w<N>/names/0` (a pseudo-week, so the published rules already cover it). `SEED_PICKS` in `index.html` is the fallback starting point.
- Player names are stored at `weeks/profile/names/<slot>` (another pseudo-week) so they carry across weeks.

## Database rules

`database.rules.json` is what's published in the Firebase console. If it ever changes, paste its contents into **Realtime Database → Rules** and click **Publish**. The app deliberately keeps all shared data under `weeks/<key>/...` so the rules haven't needed to change since the first publish.

## Notes

- The Firebase free tier (Spark) is far more than enough for four people.
- The `apiKey` in a Firebase web config is not a secret; it identifies the project. Access is controlled by the rules file. It is fine for it to be in this public repo.
- Anyone with the link can pick any slot. That's fine for a friend group; if a stranger ever gets the link, rotate to a new Firebase project.
