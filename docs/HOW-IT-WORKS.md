# How Awoo works

A technical overview for anyone working on the code.

## Stack

- **Front end:** one static page, `public/index.html`, with plain HTML, CSS and JavaScript. There is no framework and no build step.
- **Back end:** Firebase Authentication (email/password and Google) and Cloud Firestore, loaded from Google's CDN using the v10 "compat" SDK.
- **Hosting:** Netlify publishes `public/` (see `netlify.toml`). Any static host works.
- **Settings:** `public/config.js` sets `window.APP_CONFIG` (app name and Firebase keys). If the keys are missing, the app runs in **preview mode** with sample data and saves nothing.

## App structure

All logic sits in one script at the bottom of `index.html`, wrapped in an IIFE.

| Part | Purpose |
| --- | --- |
| `S` | The app's state: signed-in user, household id, pets, logs, Pack posts, boops, current screen |
| `init()` / `subscribe()` | Start Firebase, watch sign-in, then subscribe live to the user's profile, their household's pets and logs, the Pack and boops |
| `render()` | Redraws the current screen from `S`. Screens are functions in `VIEWS` (`today`, `pack`, `health`, `routine`, `memories`, `nearby`, `account`) |
| `FORMS`, `LOGS` | Bottom-sheet forms for editing pets, vet, insurance, reminders and logging walks, meals, meds and play |
| `treatCard()`, `treatSheet()` | The shared treat tracker and daily limit |
| `saveDog()` / `writeDog()` | Saves a pet. Only changed top-level fields are written, so two people editing different things at once don't overwrite each other |
| `addLog()` | Every walk, treat, meal and so on is its own document, so simultaneous logging never conflicts |

The welcome, paywall and sign-in screens are static HTML, so typing is never interrupted by a redraw.

## Data model (Firestore)

```
users/{uid}                     Private settings: handle, plan, trialStart, following[], hid (household id)
households/{hid}                { members: [uid, ...], created }
households/{hid}/dogs/{petId}   A pet (cat or dog): name, species, breed, dates, routine, done ticks,
                                reminders, vaccines, weights, vet, insurance, emergency, photos,
                                milestones, team, treatLimit
households/{hid}/logs/{logId}   One event: { type: walk|play|meal|meds|weight|photo|note|treat|reminder,
                                dogId, time, by, byUid, note?, qty?, km?, minutes?, kg? }
pack/{uid}                      Public Pack profile and up to 30 posts: { handle, dogName, species, fur,
                                photoId, posts: [...] }
boops/{uid}                     { p: { "<authorUid>_<postId>": true } }
images/{imageId}                { owner, data: "data:image/jpeg;base64,...", created }
```

(The collection is called `dogs` for historical reasons. It holds cats too.)

**Photos** are resized in the browser (up to 1080px, JPEG) and stored as data URLs in `images`, each well under Firestore's 1 MiB document limit. This keeps the project on Firebase's free plan, since Cloud Storage requires the paid Blaze plan. Pages reference photos as `img:<imageId>` and load them on demand (`hydrate()`).

## Sharing a household

1. On sign-up, the app creates `households/{hid}` with the user as the only member and stores `hid` on `users/{uid}`.
2. The **invite code** is the household id: 24 random characters, shown on the Routine screen.
3. A partner who enters the code is added to `members` (the rules allow someone to add **only themselves**, up to 6 members), and their profile's `hid` is set.
4. Both accounts then subscribe to the same `households/{hid}/dogs` and `logs`, so changes appear live on every phone.

## Security rules

See `firestore.rules`. In short:

- `users/{uid}`: only that user.
- `households/{hid}` and everything under it: only members. Anyone signed in can read a household document they know the id of, and can add themselves to `members`, but nothing else.
- `pack/{uid}`, `boops/{uid}`: everyone signed in can read; only the owner can write.
- `images/{id}`: everyone signed in can read one by id (no listing); only the uploader can create or delete; under 1 MB.

## Running it locally

Serve the `public` folder with any static server, for example:

```
npx serve public
```

Then open the address it prints. Add `localhost` to Firebase's Authorized domains to test sign-in locally.

## Known limits

- No push notifications yet. Reminders show when the app is open.
- Payments are simulated (`profile.plan`). Stripe is the planned web payment provider.
- Account deletion is handled by email. There is no in-app delete-account button yet.
- The Pack is visible to everyone signed in. There is no private or friends-only posting yet.
- Each Pack profile keeps the latest 30 posts.
