# GitHub Setup Guide

Step-by-step for turning this folder into a real GitHub repo, from zero.

## 1. Create the repo on GitHub

1. Go to github.com, log in, click the **+** in the top right -> **New repository**.
2. Repository name: something clear, e.g. `mydfir-30day-soc-lab`.
3. Description: one line, e.g. "Hands-on SOC/DFIR home lab — MyDFIR 30-Day Challenge."
4. Set to **Public** (so employers can see it).
5. **Do NOT** check "Add a README file" — you already have one and don't want a
   conflict.
6. Click **Create repository**. Leave the page open — it shows you the exact commands
   for the next step (they'll match what's below).

## 2. Get Git installed and configured (one-time, skip if already done)

On your Windows host (not inside a VM), download and install Git from
https://git-scm.com if you don't already have it. Then in a terminal:

```bash
git config --global user.name "Your Name"
git config --global user.email "the-email-you-used-for-github@example.com"
```

## 3. Put all the files in one local folder

Arrange your local folder to match this structure exactly:

```
mydfir-30day-soc-lab/
├── README.md
├── PROGRESS.md
├── GITHUB_SETUP_GUIDE.md
├── SCREENSHOT_CHECKLIST.md
├── network-setup/
│   ├── README.md
│   ├── netplan-fix.md
│   ├── nat-vs-hostonly.md
│   ├── dhcp-hostonly-fix.md
│   └── kali-adapter-fix.md
├── elastic-stack-deployment/
│   ├── README.md
│   ├── agent-install-enroll.md
│   ├── cert-trust-issue.md
│   └── service-verification.md
├── attack-simulation/
│   ├── README.md
│   └── hydra-ssh-bruteforce.md
├── detection-validation/
│   ├── README.md
│   ├── discover-query-validation.md
│   └── geoip-map-investigation.md
├── dashboards/
│   ├── README.md
│   └── failed-ssh-logins-timeline.md
└── screenshots/
    ├── README.md
    └── (your .png files go here — see SCREENSHOT_CHECKLIST.md)
```

Every file above was generated for you in this conversation — download them all and
place them into folders with these exact names.

## 4. Initialize Git and push

Open a terminal, `cd` into the top-level `mydfir-30day-soc-lab` folder, then:

```bash
git init
git add .
git commit -m "Initial commit: Days 13-14 - networking, Elastic Stack, attack simulation"
git branch -M main
git remote add origin https://github.com/<your-username>/mydfir-30day-soc-lab.git
git push -u origin main
```

Replace `<your-username>` with your actual GitHub username — the exact URL is also
shown on the "Create repository" page from Step 1 if you want to copy it directly.

If it asks you to log in, GitHub now requires a **Personal Access Token** instead of
your password for command-line pushes:
1. GitHub -> Settings -> Developer settings -> Personal access tokens -> Tokens
   (classic) -> Generate new token
2. Give it `repo` scope, generate, copy the token immediately (you won't see it again)
3. When Git prompts for a password, paste the token instead

## 5. Adding screenshots later

Every time you add new screenshots to `/screenshots/`:

```bash
git add screenshots/
git commit -m "Add screenshots for [section]"
git push
```

## 6. Updating as you complete more days

As you finish more of the 30 days:

```bash
# edit PROGRESS.md, add new day entries
# add any new folders/files for that day's work
git add .
git commit -m "Add Day 15 progress"
git push
```

## 7. Final polish before sharing the link with employers

- Make sure `README.md` renders cleanly on the GitHub repo's main page (GitHub
  auto-displays it) — check formatting looks right in the browser, not just locally.
- Pin this repo on your GitHub profile: Profile -> Customize your pins -> select it.
- Add a one-line repo description and topics (e.g. `soc`, `dfir`, `elastic-stack`,
  `homelab`) via the gear icon next to "About" on the repo page — this helps it show up
  in searches and gives recruiters instant context.
