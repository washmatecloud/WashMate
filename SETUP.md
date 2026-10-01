# WashMate Cloud — setup and publishing guide

This project is a responsive React website backed by Supabase. Inventory, orders, stock movements and manager-only expenses are stored in a shared cloud database. Prices use Nigerian naira (NGN).

## What you need
- A Supabase account (free tier is enough to begin): https://supabase.com/dashboard
- A GitHub account: https://github.com/
- A Vercel account: https://vercel.com/
- A computer for the initial setup. After publishing, staff can use the site from their phones.

## 1. Create the Supabase project
1. Sign in to Supabase and choose **New project**.
2. Name it `washmate-cloud`; create a strong database password and save it privately.
3. Select a region suitable for your business and wait for provisioning.
4. Open **SQL Editor → New query**.
5. Open `supabase/schema.sql` from this project, copy all its contents into the SQL editor, and run it.
6. Open **Project Settings → API** (or **Connect**) and copy the Project URL and the publishable key (or legacy anon key). These two values are intended for the browser when row-level security is correctly enabled. NEVER use the `service_role` or secret key in frontend code.

## 2. Create your first manager
1. Run the site locally (step 4) or deploy it (step 3) and use **Create an account** with the owner's email.
2. If Supabase email confirmation is enabled, confirm the email using the link sent to the owner.
3. In Supabase, open **SQL Editor** and run this query, replacing the email with the exact owner email:
   ```sql
   update public.profiles p
   set role = 'manager'
   from auth.users u
   where p.id = u.id and lower(u.email) = lower('OWNER_EMAIL_HERE');
   ```
4. Sign out and sign in again. The owner should now see Expenses, Reports, and Staff & Settings.
5. Have the receptionist create their own account. New accounts are receptionists by default. Do not promote them to manager unless you want them to see financial reports and expenses.

**Important:** Only run the manager-promotion query for a person you trust. Keep Supabase dashboard access and database credentials private.

## 3. Publish without buying a domain
Vercel provides a `*.vercel.app` address, so you do not need to buy a domain to start.
1. Create a GitHub repository and upload this project folder.
2. In Vercel, choose **Add New → Project**, import the GitHub repository, and deploy.
3. Add these Environment Variables in the Vercel project settings:
   - `VITE_SUPABASE_URL` = your Supabase Project URL
   - `VITE_SUPABASE_ANON_KEY` = your Supabase publishable key (or legacy anon key)
4. Redeploy after adding environment variables.
5. Open the generated `https://your-project.vercel.app` address on the owner's phone and receptionist's phone.
6. In Supabase → Authentication → URL Configuration, set **Site URL** to your Vercel URL. Add the same URL to **Redirect URLs**. For email confirmation, use the deployed address and test the confirmation flow.
7. In Supabase → Authentication → Providers → Email, choose whether to require email confirmation. For a small shop, keeping confirmation enabled is safer; ensure the owner and staff can access their inboxes.

## 4. Run locally (optional)
Install Node.js LTS from https://nodejs.org/. In this folder:
```bash
npm install
cp .env.example .env
```
Edit `.env` with your project URL and publishable/anon key, then:
```bash
npm run dev
```
Open the local URL printed by Vite. To check the production build:
```bash
npm run build
```

## 5. How the roles work
- **Manager:** inventory, stock movements, orders, expenses, reports, staff role review.
- **Receptionist:** inventory, stock movements, and laundry orders. No expenses or financial reports.
- Every staff member signs in with their own Supabase Auth account.
- Supabase Row Level Security policies enforce access in the database, not only by hiding buttons in the interface.
- The stock movement uses a database function and row lock to keep stock calculations consistent and prevent a stock-out below zero.

## 6. Data and operational notes
- Cloud records are shared between authenticated staff accounts in the same Supabase project.
- Export CSV from Orders or Reports for a spreadsheet snapshot.
- Supabase free-plan projects may pause when inactive and have usage limits; check the current Supabase dashboard and plan details.
- Use a strong unique password, enable multi-factor authentication on owner email accounts, and limit manager access.
- This is a starter business application, not a substitute for independent backups, payment reconciliation, or audited accounting.
- Do not store card details or sensitive customer information in order notes.
- Before going live, test with sample records: add a product, record stock-in, record stock-out, create an order, update payment and status, and add an expense from the manager account.

## Troubleshooting
- **“Missing Supabase environment variables” screen:** Add both Vercel environment variables and redeploy.
- **“permission denied” or no records:** Confirm `schema.sql` ran successfully and that your user has a row in `public.profiles`.
- **Owner cannot see Expenses:** Run the manager-promotion SQL and sign out/in again.
- **Email confirmation not working:** Check Supabase Authentication URL Configuration and your email spam folder.
- **Stock movement function error:** Confirm the SQL schema was run and that the selected item exists.
