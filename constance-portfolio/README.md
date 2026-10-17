# Constance Sophie Msuku | Portfolio

A one-page portfolio built with Vue 3 and Vite. Posts are Markdown files.

## Run it on your computer
1. Install Node.js (version 18 or newer) from nodejs.org.
2. Open a terminal in this folder.
3. Run `npm install`
4. Run `npm run dev` and open the address it prints.

## Add your photo
Save your professional photo as `public/photo.jpg` (square works best). Until then the site shows your initials.

## Edit your details
Open `src/data/site.js`. Change your headline, projects, skills, experience and links there.
Add your GitHub address to `github` to show the GitHub icon.

## Add a post
1. Create a file in `src/posts`, for example `my-first-fix.md`.
2. Start it with this block, then write your post below it:

```
---
title: How I fixed a DNS problem
date: 2026-10-20
summary: One sentence that shows on the card.
---

Write your post here. Use blank lines between paragraphs.
```

3. Save, commit and push to GitHub. Vercel updates the site in about a minute.

## Publish on Vercel
1. Push this folder to a new GitHub repository.
2. Go to vercel.com and sign in with GitHub.
3. Choose Add New, then Project, and import the repository.
4. Keep the defaults. Vercel detects Vite. Click Deploy.
5. In Project Settings, Domains, you can change the free address to something like constance-msuku.vercel.app.

## Before you share it
- Replace `public/Constance_Sophie_Msuku_CV.docx` with your latest CV, keeping the same file name.
- Check the page title and description in `index.html`.
- Delete or edit the welcome post in `src/posts`.
