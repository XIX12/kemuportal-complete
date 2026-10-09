# KeMU Student Portal — Updated deployment notes

## What changed
- Added the requested Kenya Methodist University address/contact block to the printable Result Slip and Fee Statement.
- Result slips now open a print-ready document; use the browser print dialog and choose **Save as PDF** if a PDF file is needed.
- Added a Fee Statement print action.
- Improved small-screen layout and horizontal scrolling for wide tables.
- Added configurable database storage: locally it uses `./data`; on Render set `DATA_DIR=/var/data` and mount a persistent disk at `/var/data`.

## Important: keeping student records on Render
Render's default filesystem is ephemeral. A persistent disk is required for this JSON-file database to survive restarts and deploys. Render persistent disks require a paid web-service instance; the free instance does not support attaching one. See https://render.com/docs/disks.

1. In Render, open the existing `kemu-student-portal-1` web service.
2. Change the service to a plan that supports persistent disks if it is currently Free.
3. Open **Disks** (or **Settings → Disks**) and add a disk:
   - Mount path: `/var/data`
   - Size: choose the smallest available size suitable for the portal.
4. Open **Environment → Environment Variables** and add:
   - Key: `DATA_DIR`
   - Value: `/var/data`
5. Save changes and let Render deploy.
6. After deployment, check logs for `Local database ready at /var/data/kemu-db.json`.

**Existing data warning:** the previous app wrote its database to `/app/data/kemu-db.json` on Render's ephemeral filesystem. That file is not automatically copied into a newly attached `/var/data` disk. Before deploying this update, preserve/export any important records from the current live instance; if you cannot export them, expect to verify and possibly re-enter student records after the persistent disk is configured. After the disk-backed deployment, keep `DATA_DIR=/var/data` in place and do not remove the disk.

## Push this update to GitHub (Git Bash)
Run these commands from the local project folder `kemuportal`:

```bash
git status
git add server.mjs public/app.js public/styles.css dist/app.js dist/styles.css DEPLOYMENT-UPDATE-GUIDE.md .gitignore
git commit -m "Improve responsive portal and printable student documents"
git push origin main
```

If Git says there is nothing to commit, check that you extracted this updated ZIP over/into the project folder you use for GitHub. Do not run `git init` again if the repository is already initialized.

Render should automatically deploy after the push if auto-deploy is enabled. Otherwise, open the Render service and choose **Manual Deploy → Deploy latest commit**.

## Verify after deployment
- Open `https://kemu-student-portal-1.onrender.com`.
- Log in as a student and open Results → **Print / Save result slip**.
- Open Fees → **Print fee statement**.
- On a phone, test navigation, student/course tables (tables may scroll horizontally), and the print buttons.
- As admin, add a test student, restart/redeploy the service only after the persistent disk is mounted, then confirm that the student remains. Delete the test student afterward if desired.

## Security reminder
Set a strong `KEMU_ADMIN_PASSWORD` in Render Environment Variables. Do not commit live database files, passwords, `.env` files, or student records to GitHub.
