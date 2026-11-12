# Doctor Bernd's Case Rounds: complete beginner's guide

**Doctor Bernd** is a voice chatbot. He plays a male senior consultant (attending) paediatrician who quizzes a medical student or resident on an infant's case: key findings, differential diagnosis, tests and treatment, and then, near the end, the final diagnosis. Afterwards he reviews the session with the learner in short bullet points and recommends improvements.

**What he does**
- Reads the case file you choose and works out the final diagnosis himself, but keeps it secret until the learner commits to an answer.
- Asks one question at a time and **adapts**: strong answers get harder follow-ups, weak answers get hints and simpler steps.
- Pitches the questions to the **level** you choose, from pre-clinical student to fellow.
- Raises one **red herring**, a tempting but misleading line of thought. Strong learners reject it with evidence; average learners follow it.
- **Never uses abbreviations.** Every term is spelled out in full.
- Speaks with a **male voice** and listens to spoken answers.
- **⏸ Hold** pauses, **▶ Continuation** resumes, **↺ Reset** clears the case rounds, and saying **"stop"** ends the session and starts the review.

---

## What's in the folder

| File | What it is |
|---|---|
| `index.html` | The chatbot page. This is your website. |
| `worker.js` | Code for your Cloudflare Worker. It holds **no** secrets. |
| `cases/cases.json` | The list of shared cases shown in the menu. |
| `cases/floppy-infant-6-weeks.txt` | Floppy infant case (diagnosis withheld). |
| `cases/fever-and-rash-5-months.txt` | Fever and rash case (diagnosis withheld). |
| `cases/case-template.txt` | Blank template for writing new cases. |
| `README.md` | This guide. |

## How the pieces fit together

```
Learner's browser  ──►  Cloudflare Worker  ──►  Claude
(GitHub Pages:          (worker.js: keeps        (does the
 index.html)             your API key secret)     thinking)
```

The website never sees your Anthropic key. Only the Worker has it, and the Worker only answers **your** GitHub website, and only when the right **access code** is given.

---

## Before you start: what you need

- A computer with **Google Chrome** or **Microsoft Edge** (best for voice).
- A **GitHub** account (github.com).
- Your **Cloudflare** account with the Worker you made before, **Infant Case Rounds** (dash.cloudflare.com).
- Your **Anthropic Console** account with some credit (console.anthropic.com).
- About **30 minutes** the first time.

Tip: keep this guide open in one window and the websites in another.

---

## PART 1: Unzip the files (2 minutes)

1. Find **doctor-bernd.zip** in your **Downloads** folder.
2. Unzip it:
   - **Windows:** right-click the file → **Extract All…** → **Extract**.
   - **Mac:** double-click the file.
3. You now have a folder called **doctor-bernd**. Open it and check that you can see `index.html`, `worker.js`, `README.md` and a folder called `cases`.

---

## PART 2: Check your Anthropic account (2 minutes)

The Claude API is paid separately from any Claude subscription.

1. Go to **https://console.anthropic.com** and log in.
2. In the left menu, click **Billing**. Check that you have credit (for example US $5 to $10).
3. Your Worker already has an API key, so you do not need a new one.
   *Only if you lost it:* left menu → **API Keys** → **Create Key** → give it a name → **copy the key immediately** (it starts with `sk-ant-` and is shown only once). Keep it in a safe place; never put it on GitHub.

---

## PART 3: Update your Cloudflare Worker (about 10 minutes)

You will reuse your existing Worker, **Infant Case Rounds**, and replace its code. Your saved variables (API key, access code and so on) are **kept** when you change the code. The new code also serves your Parent Conversation Simulator.

### 3.1 Open the Worker
1. Go to **https://dash.cloudflare.com** and log in.
2. In the left menu, click **Compute (Workers)** → **Workers & Pages**. (On some accounts it is just called **Workers & Pages**.)
3. Click your Worker, **infant-case-rounds**.

### 3.2 Back up the old code (safety step)
1. Click **Edit code** (top right, `</>` icon). A code editor opens.
2. Click inside the code. Press **Ctrl+A** (Windows) or **Cmd+A** (Mac) to select everything, then **Ctrl+C** / **Cmd+C** to copy.
3. Open **Notepad** (Windows) or **TextEdit** (Mac), paste (**Ctrl+V** / **Cmd+V**) and save the file as `old-worker-backup.txt` on your desktop.
   If anything ever goes wrong, you can paste this old code back.

### 3.3 Paste in the new code
1. On your computer, open `worker.js` from the doctor-bernd folder in a plain text editor:
   - **Windows:** right-click `worker.js` → **Open with** → **Notepad**.
   - **Mac:** right-click → **Open With** → **TextEdit**.
2. Select everything (**Ctrl+A** / **Cmd+A**) and copy it (**Ctrl+C** / **Cmd+C**).
3. Go back to the Cloudflare code editor. Click inside the code, select everything (**Ctrl+A** / **Cmd+A**) and press **Delete**. The editor is now empty.
4. Paste (**Ctrl+V** / **Cmd+V**).
5. Click the blue **Deploy** button (top right). Confirm if it asks.

### 3.4 Check the variables
1. Go back to the Worker's main page (click the Worker name at the top, or the back arrow).
2. Click **Settings** → **Variables and Secrets**.
3. Check that these three exist. If one is missing, click **+ Add**, choose the **Type**, type the **Name** exactly, paste the **Value**, and save.

| Name (type exactly) | Type | Value |
|---|---|---|
| `ANTHROPIC_API_KEY` | Secret | your key from console.anthropic.com |
| `ACCESS_CODE` | Secret | a passphrase you choose, for example `rounds-2026` |
| `ALLOWED_ORIGINS` | Text | `https://YOUR-GITHUB-USERNAME.github.io` |

**About ALLOWED_ORIGINS:**
- Write only `https://` + your GitHub username + `.github.io`. No repository name, no `/` at the end.
- ✅ `https://jsmith.github.io` ❌ `https://jsmith.github.io/doctor-bernd/`
- The name must end with an **S**.
- If it already has your github.io address (from the Parent Conversation Simulator), **leave it**. All your repositories share that one address.

**Optional model variables (Text):** `MODEL_FAST` = `claude-haiku-5-5`, `MODEL_BALANCED` = `claude-sonnet-5-5`, `MODEL_THOROUGH` = `claude-opus-5-5`. These are already built in, so you can leave them out. If you set `MODEL_BALANCED = claude-sonnet-5-5` before, that's fine.

4. If you added or changed anything, click **Deploy**. **Variables only work after you deploy.**

### 3.5 Test the Worker
1. On the Worker's main page, find its address. It looks like `https://infant-case-rounds.YOUR-SUBDOMAIN.workers.dev`. **Copy it** and paste it into Notepad; you need it in Part 4.
2. Paste the address into a new browser tab and press **Enter**. You should see text like:
   `{"ok":true,"service":"Claude chatbot proxy (Doctor Bernd and other GitHub Pages chatbots) v2","apiKeySet":true,"accessCodeSet":true,...}`
   - `"apiKeySet":false` → add the `ANTHROPIC_API_KEY` secret (step 3.4) and Deploy.
   - `"allowedOrigins":"NOT SET..."` → add `ALLOWED_ORIGINS` (step 3.4) and Deploy.
   - Text without **"Doctor Bernd"** → the old code is still running. Repeat step 3.3 and click **Deploy**.

---

## PART 4: Put the chatbot on GitHub (about 10 minutes)

### 4.1 Create a new repository
A repository ("repo") is a project folder on GitHub.
1. Go to **https://github.com** and sign in.
2. Click the **+** at the top right → **New repository**.
3. **Repository name:** `doctor-bernd`
4. Choose **Public**. Free GitHub Pages needs a public repository. The page contains no secrets.
5. Leave **Add a README file** **unticked**.
6. Click the green **Create repository** button.

> Already have a `doctor-bernd` repository from before? Either open it and upload the new files over the old ones (GitHub replaces files with the same name), or delete it first: repository → **Settings** → scroll to the bottom → **Delete this repository**.

### 4.2 Upload the files
1. On the new empty repository page, click the blue link **uploading an existing file**.
2. Open your unzipped **doctor-bernd** folder on your computer.
3. Select **index.html**, **README.md**, **worker.js** and the **cases** folder (hold **Ctrl** on Windows or **Cmd** on Mac to select several), then drag them all into the GitHub upload box.
   Dragging the `cases` folder keeps its files inside `cases/`, which is required.
4. Wait until all files appear in the list under the box (you should see `cases/cases.json` and the case files).
5. Scroll down and click the green **Commit changes** button.

### 4.3 Put your Worker address into the page (recommended)
Then learners only need to type the access code.
1. In your repository, click **index.html**.
2. Click the **pencil icon ✏️** (Edit this file), top right of the file.
3. Press **Ctrl+F** / **Cmd+F** and search for `WORKER_URL`. You will find:
   ```js
   WORKER_URL: "",
   ```
4. Click between the two quotation marks and paste your Worker address:
   ```js
   WORKER_URL: "https://infant-case-rounds.YOUR-SUBDOMAIN.workers.dev",
   ```
   Keep both quotation marks and the comma.
5. Click **Commit changes…** (top right) → **Commit changes**.

### 4.4 Switch on GitHub Pages
1. In the repository, click **Settings** (gear icon, top menu).
2. In the left menu, click **Pages**.
3. Under **Build and deployment** → **Source**, choose **Deploy from a branch**.
4. Under **Branch**, choose **main** and **/ (root)**, then click **Save**.
5. Wait 1 to 3 minutes, then refresh the page. A box appears: **"Your site is live at https://YOUR-USERNAME.github.io/doctor-bernd/"**. Click **Visit site**.

---

## PART 5: First run (5 minutes)

1. Open `https://YOUR-USERNAME.github.io/doctor-bernd/` in **Chrome** or **Edge**.
2. Click **⚙ Connection** (top right):
   - The Worker address should already be filled in. If not, paste it.
   - Type your **access code** (your `ACCESS_CODE` from Cloudflare).
   - Click **Test connection**. You should see **✔ Connected. Access code accepted.**
   - Click **Save and close**. Each browser only needs this once.
3. **Step 1, Patient case file:** click **Choose file** and pick the case from your computer (.txt, .md, .docx or .pdf), **or** choose a shared case from the list.
4. **Step 2, Learner and level:** type the learner's name (optional) and choose their **seniority**. Choose the quiz length. Keep **Include a red herring** ticked.
5. **Step 3, Voice and quality:** choose the accent and Doctor Bernd's voice. Voices marked **(male)** are best. Click **🔊 Test Doctor Bernd's voice** to hear him. Leave quality on **Balanced**.
6. Click **▶ Start case rounds**. When the browser asks to use the microphone, click **Allow**.

---

## PART 6: During the case rounds

- Doctor Bernd reads the case (10 to 30 seconds), greets the learner and asks his first question **aloud**.
- When he finishes speaking, the microphone turns **red**: answer out loud. After a short pause (2.5 seconds by default) the answer is sent. You can also click 🎤 to send at once, or **type** and press **Enter**.
- The bar at the top shows the phase (Case summary → Key findings → Differential diagnosis → Tests → Treatment → Final diagnosis → Extension), the question number, the difficulty and the time.
- **📄 Show the patient case file** lets the learner reread the case. (It is hidden if the file itself names the diagnosis.)

| Button | What it does |
|---|---|
| **⏸ Hold** | Pauses everything: voice, microphone and timer. |
| **▶ Continuation** | Carries on exactly where you paused (Doctor Bernd repeats the sentence he was saying). |
| **↺ Reset** | Clears the conversation. Choose **Restart this case** (same case and level, from the beginning) or **Back to setup** (pick a new case or level). |
| **⏹ End and review** | Ends now and goes to the review. |
| Say **"stop"** | Same as End and review. It must be said on its own: "I would stop the feeds" does **not** end the session. |

---

## PART 7: The review

After "stop" (or when Doctor Bernd finishes his questions), he reviews the session and speaks a short summary. The review is a short point-form summary:
- an **overall score** out of 100 and a band (Excellent, Good, Average, Needs more work)
- **final diagnosis**: what the learner said compared with the correct answer
- **red herring**: how the learner handled it
- up to 3 **strengths** and up to 4 **recommended improvements**, each with one way to practise
- one **focus for the next session**
- fold-out sections: question by question, Doctor Bernd's answer key, and the full transcript

Buttons: **🔊 Hear summary**, **📋 Copy report**, **⬇ Download report**, **🖨 Print or save as PDF**, **↺ Same case again**, **New case or level**.

---

## PART 8: Check your Parent Conversation Simulator still works

Both chatbots share one Worker. After updating the Worker, open your Parent Conversation Simulator page and start a short session.
- **It works:** you're done.
- **It shows an error:** go back to Cloudflare → your Worker → **Edit code**, paste the code from `old-worker-backup.txt`, and click **Deploy**. Then tell Claude the error message, and it will make one Worker that serves both chatbots.

---

## Adding your own cases

**For yourself:** just click **Choose file** each time.

**For everyone (shared list):**
1. In your GitHub repository, click the **cases** folder → **Add file** → **Upload files**.
2. Drag in your case file and click **Commit changes**.
3. Click `cases.json` → ✏️ and add a line for it. For example:
   ```json
   [
     { "title": "Floppy infant, 6 weeks old", "file": "floppy-infant-6-weeks.txt" },
     { "title": "Fever and rash, 5 months old", "file": "fever-and-rash-5-months.txt" },
     { "title": "Vomiting infant, 3 weeks old", "file": "vomiting-infant-3-weeks.txt" }
   ]
   ```
   Every line ends with a comma **except the last one**. `file` must match the file name exactly. **Never put the diagnosis in the title or the file name**, because learners can see them.
4. Click **Commit changes**, wait a minute, and reload the chatbot page.

**Writing good cases:** copy `cases/case-template.txt`. Leave the diagnosis out (Doctor Bernd works it out). Hide the real clue naturally in the history. Use **invented or fully de-identified** details only; never upload real patient records.

---

## Troubleshooting

| Problem or message | What to do |
|---|---|
| "…is not in ALLOWED_ORIGINS" | In Cloudflare set `ALLOWED_ORIGINS` to exactly `https://YOUR-USERNAME.github.io` (no path, no `/` at the end), then **Deploy**. |
| "Wrong access code." | Retype it in ⚙ Connection (capital letters matter). To change it, edit the `ACCESS_CODE` secret in Cloudflare and Deploy. |
| "ANTHROPIC_API_KEY is not set" | Add the secret in Cloudflare → Settings → Variables and Secrets, then Deploy. |
| "…looks like old Worker code" | Repeat Part 3, step 3.3, and click **Deploy**. |
| An error mentioning "model" | Set the Text variable `MODEL_BALANCED` to `claude-sonnet-5-5` (or delete it), then Deploy. |
| "credit balance is too low" | Add credit at console.anthropic.com → Billing. |
| "Could not reach the Worker" | Check the Worker address in ⚙ Connection. Open it in a tab: it should show `"ok":true`. |
| "Doctor Bernd could not start" | Press **↺ Reset → Restart this case**. If it keeps failing, choose **Thorough** quality or shorten the file. |
| The microphone doesn't work | Use Chrome or Edge. Click the 🔒 icon left of the web address → **Microphone: Allow** → reload. You can always type. |
| No sound from Doctor Bernd | Tick **Doctor Bernd speaks aloud**, turn up the volume, click **🔊 Test Doctor Bernd's voice**. |
| His voice sounds female | Choose a voice marked **(male)**. If there are none: **Windows** Settings → Time & language → Speech → Add voices (for example "Microsoft Ryan" or "Guy"). **Mac** System Settings → Accessibility → Spoken Content → System voice → Manage Voices (for example "Daniel"). In Chrome, "Google UK English Male" is usually available. |
| A scanned PDF won't load | It is a picture of text. Save the case as .txt or .docx instead. |
| My new shared case isn't in the list | Check `cases.json` for a missing comma (paste it into https://jsonlint.com), wait a minute, and reload with **Ctrl+F5** / **Cmd+Shift+R**. |
| The page didn't change after an edit | GitHub Pages takes 1 to 3 minutes. Then reload with **Ctrl+F5** (Windows) or **Cmd+Shift+R** (Mac). |

## Cost
GitHub Pages and Cloudflare Workers are free at this size. You pay Anthropic per use. A standard 10-question session with its review on **Balanced** usually costs a few cents. **Fast** is cheaper; **Thorough** costs more. **Restart this case** reuses Doctor Bernd's answer key, so it costs less than a new case.

## Tips for teachers
- **Group teaching:** project the page and let learners take turns. Use **Hold** for discussion, then **Continuation**.
- **Comparing learners fairly:** use the same case, level and length for everyone. The red herring verdict ("Rejected with good reasoning", "Hesitated", "Accepted it") helps separate strong from average reasoning.
- Download each learner's report to track progress over time.
- The answer key is kept in the page's memory during a session. A technically skilled learner could find it with the browser's developer tools, so for formal assessment, run the session on your own computer.

---

## Appendix: build a new Worker from scratch (only if you lose yours)
1. Cloudflare → **Workers & Pages** → **Create** → **Create Worker** → name it `infant-case-rounds` → **Deploy**.
2. Click **Edit code**, replace everything with `worker.js` (as in step 3.3) → **Deploy**.
3. **Settings** → **Variables and Secrets** → add the three variables from step 3.4 → **Deploy**.
4. Test it as in step 3.5, then put the new address into `index.html` (step 4.3) and ⚙ Connection.
