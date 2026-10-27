# Orbit — Student Dashboard

Orbit is a responsive student dashboard and planner for keeping academic work organised in one place. It is built with HTML, CSS and JavaScript and can run as a standalone webpage in a modern browser.

## Features

- **Personal dashboard** — an overview of attendance, today's classes, pending assignments and upcoming deadlines.
- **Attendance tracking** — mark scheduled class hours as present or absent and review monthly and subject-wise attendance.
- **Weekly timetable** — create recurring classes by weekday, subject, time and room.
- **Assignment planner** — record assignments, due dates and submission status.
- **Subject management** — manage the subjects used by attendance, the timetable and assignments.
- **Personalised greeting** — uses the student's saved name and the device's local time.
- **Backup and restore** — export academic data to a JSON file and import a backup.
- **Supabase cloud sync** — when configured, sign-in lets a student load and save their own data online and access it from another device.
- **Responsive layout** — designed for laptop and phone screens.
- **Dark Orbit branding** — dark interface with violet accents.

## What students need

- A modern browser such as Chrome, Edge, Firefox or Safari.
- The Orbit website link.
- The Supabase project URL and publishable/anon key supplied by the website administrator, if these are not already configured in the deployed app.
- An email address to create an account.
- Internet access for account creation and cloud sync.

## Enrolling as a student

1. Open the Orbit website using the link shared by your college, class or website administrator.
2. Open **Settings & backup**.
3. In **Cloud sync (Supabase)**, enter the Supabase project URL and publishable/anon key supplied by the administrator, if the app has not already been configured on your device. These are project connection details, not your personal login credentials.
4. Select **Save cloud config**.
5. Enter your own email address and choose a password.
6. Select **Create account**.
7. If Supabase email confirmation is enabled, open the confirmation email and verify your address.
8. Return to Orbit, enter the same email and password, and select **Sign in**.
9. Orbit should load your academic records from your Supabase account. You can then add subjects, build your timetable, record attendance and add assignments.

**Important:** Use your own account. Do not share your password with classmates or use another student's account. If sign-up or sign-in fails, contact the website administrator and share the error message shown in Settings.

## Using Orbit

### 1. Set up your profile and subjects

- Go to **Settings & backup**.
- Enter your name and select **Save name**.
- Add your subjects. Subject names are used throughout attendance, assignments and the timetable.

### 2. Create your weekly timetable

- Open **Timetable** and select **Add class**.
- Choose the subject, weekday, start time, end time and room/location if needed.
- Add each recurring class.
- Orbit uses the timetable to generate class sessions for a selected date.

### 3. Record attendance

- Open **Attendance**.
- Choose the date and, if needed, filter by subject.
- Mark each scheduled class as **Present** or **Absent**.
- Use **Monthly summary** and **By subject** to review your records. Attendance percentages depend on the sessions you have marked.

### 4. Manage assignments

- Open **Assignments** and select **New assignment**.
- Enter the title, subject and due date.
- Use the filters to find pending, submitted or overdue tasks.
- Mark an assignment as submitted when you have completed it.

### 5. Back up your work

- In **Settings & backup**, select **Export backup (.json)** to download a copy.
- To restore a backup, select **Import backup** and choose a valid Orbit JSON backup.
- Keep backups somewhere safe. A backup is an extra safeguard and is not a substitute for checking that cloud sync completed successfully.

## Using Orbit on another device

1. Open the same deployed Orbit website on the other device.
2. If the project details are not preconfigured, enter the same Supabase project URL and publishable/anon key in Settings and save them.
3. Sign in with the **same student account**.
4. Orbit should retrieve that account's data from Supabase.

The Supabase project configuration is stored in browser storage on the device in the current standalone build. It may need to be entered again on a different browser or device. Student records should be retrieved from Supabase after sign-in; do not assume a change has synced until the app reports a successful sync.

## For the website administrator

### Deploying the website

1. Put the app HTML file in a GitHub repository.
2. Keep the branding files alongside the HTML if the file references them:
   - `icon.png` — sidebar icon
   - `logo.png` — optional full logo used on the dashboard
   - `favicon.png` — browser-tab icon, if referenced by the HTML
3. Enable **GitHub Pages** in the repository settings, selecting the branch and folder containing the website.
4. Open the published Pages URL and test it on desktop and mobile.
5. Configure Supabase before inviting students to create accounts.

The actual asset filenames must match the filenames referenced by the HTML. If your version uses different names, update either the files or the references.

### Configure Supabase

1. Create a project in Supabase.
2. In the project's SQL Editor, create the table and Row Level Security policies required by the app.
3. In the app's Settings, use the project URL and the publishable/anon key.
4. Never place a Supabase **service-role key** in a browser application or commit it to GitHub.
5. Test account creation, email confirmation (if enabled), sign-in, data loading, saving, sign-out and sign-in on a second device before launch.

The current app code expects a table named `student_planner_data` with:
- `user_id` — the authenticated student's UUID, unique per row
- `payload` — a JSONB object containing the student's Orbit data
- `updated_at` — the last update timestamp

A compatible starting SQL schema is:

```sql
create table if not exists public.student_planner_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  payload jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.student_planner_data enable row level security;

create policy "Students can read their own Orbit data"
on public.student_planner_data
for select
to authenticated
using (auth.uid() = user_id);

create policy "Students can insert their own Orbit data"
on public.student_planner_data
for insert
to authenticated
with check (auth.uid() = user_id);

create policy "Students can update their own Orbit data"
on public.student_planner_data
for update
to authenticated
using (auth.uid() = user_id)
with check (auth.uid() = user_id);
```

Run the policies only once; if equivalent policies already exist, review them rather than creating duplicates. Verify the table and policies in Supabase before using real student data.

### Important deployment note about project configuration

The current standalone HTML asks for the Supabase project URL and public key in each browser unless you change the app to ship those public project settings with the website. For a smoother student onboarding experience, the administrator can configure the project URL and publishable/anon key in the deployed app. These values are intended to be public, but **Row Level Security is essential**. Never embed a service-role key.

### Data storage and privacy

- The app's cloud model stores one JSON payload per authenticated user in `student_planner_data`.
- Row Level Security must restrict each signed-in user to rows where `user_id = auth.uid()`.
- Cloud access requires a valid Supabase configuration and authentication.
- A successful sign-in alone does not prove that the table or policies are configured correctly; test actual reads and writes.
- The app's local browser storage is used for Supabase connection settings in the current implementation. If you modify storage behaviour, test that student records are not unintentionally cached locally.
- Avoid entering sensitive information that is not necessary for academic planning.

## Troubleshooting

**“Enter and save your Supabase URL and anon/public key first.”**
Ask the website administrator for the correct project URL and public key. Save them in Settings before signing up or signing in.

**Email confirmation required**
Open the confirmation message from Supabase, verify the email address, then return to Orbit and sign in.

**Sign-in works but records do not load**
The administrator should check that `student_planner_data` exists, that its column names match the schema, that Row Level Security is enabled, and that the select/insert/update policies use `auth.uid() = user_id`.

**Changes are not appearing on another device**
Confirm that the app reports a successful sync, both devices use the same Supabase project, and you are signed in to the same student account.

**I am changing browsers or devices**
The project URL and public key may need to be configured again. Sign in with the same account to load the cloud data.

**I need to move or recover data**
Use **Export backup (.json)** before switching devices or making major changes. Importing a backup replaces the current in-memory workspace in the current implementation, so verify the account and backup before importing.

## Technology

- HTML
- CSS
- JavaScript
- Supabase Auth
- Supabase Postgres with Row Level Security
- GitHub Pages (optional static hosting)

## Security reminder

The Supabase publishable/anon key is designed for use in client-side applications when the database is protected by correct Row Level Security policies. A service-role key bypasses those protections and must never be exposed in this repository or webpage.

---

**Orbit — Student Dashboard**  
Organise your classes, track attendance and stay ahead of deadlines.
