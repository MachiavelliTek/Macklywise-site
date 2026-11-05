# Macklywise: deploy guide (GitHub Pages)

## 1. Before you go live
- Your WhatsApp number (+263710490111) is already set. Testimonials stay hidden until you add the first one.
- If your domain is not exactly `macklywise.com`, search and replace it in every file.
- **Payments and member data need the backend.** Follow `BACKEND-SETUP.md` (Supabase + Turnstile + Paynow). Until `assets/config.js` is filled in, the "Pay now" button sends the customer to WhatsApp instead, and nothing pretends to take money.

## 2. Put it on GitHub
1. Create a new repository (public), e.g. `macklywise-site`.
2. Upload everything inside this folder to the repository root (on a phone: Add file > Upload files). `index.html` must be at the top level, with `CNAME`, `.nojekyll`, `assets/` and the page folders beside it.
3. Settings > Pages > Source: **Deploy from a branch** > `main` > `/ (root)` > Save.
4. Settings > Pages > Custom domain: `macklywise.com` > Save, then tick **Enforce HTTPS** when it becomes available.

## 3. Point Spaceship at GitHub
In Spaceship > Domain > DNS records:
- Four **A** records for host `@`: `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- One **CNAME** for host `www`: `YOUR-GITHUB-USERNAME.github.io`
(Check GitHub's docs "Managing a custom domain for your GitHub Pages site" if these ever change.) DNS can take from minutes to a few hours.
Email: sales@ and enquiry@macklywise.com need mailboxes or forwards in Spaceship, with the MX records they give you.

## 4. Get found on search engines
- Google Search Console: add `https://macklywise.com`, then submit `https://macklywise.com/sitemap.xml`.
- Bing Webmaster Tools: import from Google Search Console, then submit the same sitemap (Bing also feeds DuckDuckGo and Yahoo).
- Share the links on WhatsApp, LinkedIn and Facebook business pages so the preview image appears.

## What is in the pack
- Real page URLs: `/`, `/shop/`, `/services/`, `/testimonials/`, `/free-tools/`, `/about/`, `/contact/`, each with its own title, description, canonical link, social preview and structured data. The page content is already in the HTML, so search engines can read it without running JavaScript.
- Speed: CSS, JS and images are separate cached files. Poor connections, Data Saver or very low-memory phones automatically get a lighter version (no cloud animation or blur effects).
- Known limit: accounts, orders, comments, likes and admin uploads are saved in each visitor's own browser. A backend (Supabase or Firebase) is needed to share them between people.
