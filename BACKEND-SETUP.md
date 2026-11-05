# Macklywise: secure backend + EcoCash setup

Everything below is free to start. Do the steps in order. Nothing here can be tested from inside Claude, so test each step in the sandbox before using real money.

## 0. What protects the money (read this once)
| Attack | Defence |
|---|---|
| Hacker changes the price in the browser | Prices are read from the database on the server. The browser only sends *which* product and how many. |
| Hacker redirects the payout to their wallet | The payout wallet is set only inside your **Paynow** account (protected by your Paynow login). No page, no admin panel and no API in this project can change it. |
| Fake "payment successful" message | Only the server marks a payment paid, and only after it asks Paynow directly and checks the signature, reference and exact amount. A forged webhook is ignored. |
| Stealing files without paying | Files live in a **private** bucket with no read access. The only door is the `download` function: logged in + owner of a *paid* payment + link that expires in 60 s + 10-download cap + audit log. |
| Hacked admin login | Admin = a role in the database **and** a 2-step code (authenticator app). Without the code the database refuses every admin write. There is no PIN in the code any more. |
| Bots and spam | Cloudflare Turnstile on sign-up, login, comments, orders and every payment. Comments, orders and payments cannot be inserted by direct API calls at all. |
| Someone reading other people's data | Row Level Security on every table. Members see only their own orders and payments. Anonymous comments never reveal the author. |
| Brute force and flooding | Supabase Auth rate limits + our own limits (5 payments per 10 min, 5 comments per minute, 30 downloads per hour). |
| Stolen or leaked keys | Only public keys are in the repo. Secrets live in Supabase's encrypted secrets store. |

No system is "unhackable". These layers close the common ways websites lose money. Keep your own accounts safe (see section 7).

## 1. Create the Supabase project
1. supabase.com > New project (choose the region closest to you, e.g. Cape Town/Frankfurt). Save the database password in a password manager.
2. SQL Editor > New query > paste all of `supabase/schema.sql` > Run.
3. Optional starter products: paste `supabase/seed.sql` > Run.
4. Authentication > Providers > Email: turn on **Confirm email**. Set minimum password length to **10**. Authentication > Sign In / Providers: turn **off** anonymous sign-ins.
5. Authentication > Settings > **Multi-factor**: make sure TOTP is enabled.
6. Authentication > URL Configuration: Site URL `https://macklywise.com`, add the same as a redirect URL.
7. Authentication > Attack Protection: enable **CAPTCHA protection**, choose **Cloudflare Turnstile**, paste the Turnstile **secret** from step 2 below. If your plan offers "leaked password protection", switch it on.

## 2. Cloudflare Turnstile (the "I am human" check)
1. dash.cloudflare.com > Turnstile > Add widget. Domain: `macklywise.com`. Mode: **Managed**.
2. Copy the **Site key** (public) and **Secret key** (private).
3. Put the Site key in `assets/config.js` as `turnstileSiteKey`.

## 3. Paynow (EcoCash payments)
1. Register a merchant account at paynow.co.zw and create an **Integration** (get *Integration ID* and *Integration Key*). In the dashboard set your **settlement / payout** details to the EcoCash number or bank you want paid, for now +263786425697 (the Paynow account holder must own and verify it). This is the *only* place the payout destination is set.
2. Turn on 2-step login on your Paynow account.
3. Confirm with Paynow that your account is enabled for EcoCash and for the currency you charge (the code defaults to USD; change `CURRENCY` below if you use ZWG).
4. Use Paynow's **test mode** first. In test mode Paynow only accepts the merchant's own email as `authemail`, so set `PAYNOW_TEST_EMAIL` while testing and remove it when you go live.

## 4. Deploy the server functions
Install the Supabase CLI (supabase.com/docs/guides/cli), then in this folder:
```
supabase login
supabase link --project-ref YOUR_PROJECT_REF
supabase secrets set TURNSTILE_SECRET=... PAYNOW_ID=... PAYNOW_KEY=... ALLOWED_ORIGIN=https://macklywise.com CURRENCY=USD
supabase functions deploy secure-write
supabase functions deploy pay-initiate
supabase functions deploy pay-status
supabase functions deploy download
supabase functions deploy pay-webhook --no-verify-jwt
```
(`pay-webhook` must be public because Paynow calls it; it rejects anything without a valid Paynow signature and re-checks with Paynow itself.)

## 5. Connect the website
Project Settings > API: copy **Project URL** and the **anon public** key into `assets/config.js`. Never copy the `service_role` key anywhere.

## 6. Make yourself admin
1. Open the live site, Sign up with your real email, confirm the email.
2. SQL Editor: `update public.profiles set role = 'admin' where id = (select id from auth.users where email = 'YOUR_EMAIL');`
3. On the site click the Macklywise.com logo 4 times, enter your email and password. First time you will scan a QR code with Google Authenticator or Authy. Save the backup secret shown under the QR code.
4. Admin > Products: for each panel set a **cover image**, a **locked product file** and the **price**. Admin > **Direct Sales** shows every EcoCash payment, its status and download count.

## 7. Hardening checklist (do all of these)
- **Cloudflare in front of the site** (free): move your DNS to Cloudflare, proxy the records (orange cloud), SSL mode *Full (strict)*, turn on *Always Use HTTPS*, HSTS, *Bot Fight Mode*, DNSSEC, and a **rate-limiting rule** on `/` and the Supabase function paths. Add a header rule: `X-Frame-Options: DENY` and `Permissions-Policy: camera=(), microphone=(), geolocation=()`.
- **GitHub**: turn on 2-step login, use a passkey or authenticator app, protect the `main` branch, enable Dependabot and secret scanning. Do not give anyone else write access.
- **Spaceship** (domain): turn on 2-step login and **domain lock**; add SPF, DKIM and DMARC records for sales@ and enquiry@ so nobody can send mail as you.
- **Supabase**: 2-step login on your Supabase account, keep the project's database password private, review Auth > Logs weekly.
- **Paynow**: check Direct Sales against your Paynow transaction list weekly. Any payment marked "Amount mismatch" needs a manual look.
- Never share the service_role key, Paynow key or Turnstile secret, and never paste them into chat, email or the repository.
