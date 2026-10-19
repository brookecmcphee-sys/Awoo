# Launching Awoo

This guide gets Awoo live on your own web address. It takes about an hour and is free apart from the domain (about US$15 a year). No coding is needed.

**You'll set up:**

1. **Firebase**: sign-in and the database (free)
2. **Your keys** in `public/config.js`
3. **Netlify**: hosts the site and updates it whenever this repository changes (free)
4. **Your domain**, for example `getawoo.app`

---

## 1. Set up Firebase

Stay on the free **Spark** plan. No card is needed, because photos are stored compressed in the database.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with your Google account.
2. Click **Create a project**, name it **Awoo**, turn Google Analytics off, and click **Create project**.
3. On the project home page, click the **web icon `</>`**. Name the app **Awoo website**, leave "Firebase Hosting" unticked, and click **Register app**.
4. Firebase shows a code block containing `const firebaseConfig = { ... }`. Keep this page open for step 2.
5. In the left menu, go to **Build > Authentication > Get started**, then the **Sign-in method** tab:
   - **Email/Password**: switch on the first toggle and click **Save**.
   - **Add new provider > Google**: switch it on, choose your support email, and click **Save**.
6. Go to **Build > Firestore Database > Create database**:
   - Location: **australia-southeast1 (Sydney)**. This can't be changed later.
   - Choose **Start in production mode**, then **Create**.
7. Open Firestore's **Rules** tab. Delete what's there, paste in the whole of [`firestore.rules`](../firestore.rules), and click **Publish**.

## 2. Add your keys to `config.js`

You can do this on the GitHub website, with nothing to download.

1. In this repository, open **public/config.js** and click the **pencil icon** (Edit).
2. Replace each `PASTE_...` value with the matching value from Firebase's `firebaseConfig`. Keep the quote marks:

   ```js
   firebase: {
     apiKey: "AIzaSy...",
     authDomain: "awoo-12345.firebaseapp.com",
     projectId: "awoo-12345",
     storageBucket: "awoo-12345.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abc123"
   }
   ```

3. Click **Commit changes**.

These keys are designed to be public. The Firestore rules are what protect the data.

## 3. Publish with Netlify

1. Create a free account at [netlify.com](https://www.netlify.com), using **Sign up with GitHub**.
2. Click **Add new site > Import an existing project > GitHub**, and choose the **Awoo** repository.
3. Leave the build settings empty and click **Deploy**. The included `netlify.toml` tells Netlify to publish the `public` folder.
4. After a minute you get an address like `random-name-123.netlify.app`.
5. In Firebase, go to **Authentication > Settings > Authorized domains > Add domain** and add that Netlify address. Without this, Google sign-in shows an "isn't approved" message.

From now on, **every change you commit to this repository goes live automatically** within a minute or two.

## 4. Connect your domain

1. Buy your domain from Porkbun, Cloudflare or Namecheap. On 18 Oct 2026, `getawoo.app` and `awoopets.app` looked free, while `awoo.app` was registered and listed for sale.
2. In Netlify, open your site, then **Domain management > Add a domain**, and type your domain.
3. Choose **Set up Netlify DNS**. Netlify shows four nameservers.
4. At your registrar, change the domain's **nameservers** to Netlify's four.
5. Wait up to a day (usually under an hour). Netlify adds HTTPS automatically, which `.app` domains require.
6. In Firebase **Authorized domains**, add your domain and its `www.` version.

## 5. Test with your partner

1. On your phone, open your address, tap **Create an account**, choose **Cat** or **Dog**, and add your pet.
2. Add it to your home screen:
   - **iPhone (Safari):** Share, then **Add to Home Screen**
   - **Android (Chrome):** ⋮ menu, then **Add to Home screen**
3. Go to **Routine** and copy your **invite code**.
4. Your partner creates their own account, taps **Join with their code** on the welcome screen, and pastes it in.

Checklist:

- [ ] Both of you can sign in
- [ ] You both see the same pets
- [ ] A treat logged on one phone appears on the other
- [ ] Photos upload and appear in the Pack
- [ ] The emergency button opens Google Maps

## Before you invite the public

- [ ] Replace `[your contact email]` in `public/privacy.html` with a dedicated address, such as hello@yourdomain.
- [ ] Get a trademark check on the name in Australia (IP Australia, classes 9 and 42). A US brand called Awoo Pets sells collars and leashes.
- [ ] Claim matching Instagram and TikTok handles.
- [ ] Consider making this repository **private** (Settings > General > Danger Zone > Change visibility). Netlify keeps working after you do.

## Changing things later

| To change | Edit |
| --- | --- |
| The app's name | `name` in `public/config.js` (also update `public/manifest.webmanifest`) |
| Wording, screens, features | `public/index.html` |
| Who can read or write data | `firestore.rules`, then paste it into Firebase again |
| Privacy policy | `public/privacy.html` |

## Free limits

Firebase's free plan allows 50,000 reads and 20,000 saves a day, plus 1 GiB of storage. That suits a beta of a few hundred active users. Firebase emails you if you get close. Upgrading means adding a card, and costs stay small at that size.
