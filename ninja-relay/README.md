# ninja-relay

**A GitHub repository that quietly saves a small copy of poe.ninja's economy
prices, once an hour, so that other tools can read them.**

You do not need to know how to code to set this up. You need a web browser and
about ten minutes. Every step below is a thing you click.

---

## What is this actually for?

Some programs cannot open poe.ninja. Not because it is down — because they are
running somewhere with a locked-down internet connection that only allows a
short list of websites. (AI assistants running in a sandbox are the usual
example, but corporate networks do this too.)

`raw.githubusercontent.com` is almost always on that allowed list.

So this repository acts as a **relay**:

```
  poe.ninja  ──►  GitHub runs a script once an hour  ──►  saves the prices here
                                                                    │
                                        your tool reads them from ──┘
                                        raw.githubusercontent.com
```

Nothing runs on your computer. Nothing costs money. You never have to touch it
again once it is set up.

---

## Before you start: the honest bit

The prices are **poe.ninja's data**, not yours and not mine. This repository
just carries it. Three rules come with that, and the setup below builds all
three in:

1. **You must say who you are.** poe.ninja asks that anything talking to their
   API identifies itself and gives a way to make contact. That is Step 5, and
   the script flatly refuses to run until you have done it. This is not
   red tape — if something here ever misbehaves, that contact string is how
   they reach *you* instead of just blocking you.
2. **Slowly.** This asks poe.ninja for data once an hour. Their numbers only
   update about every 15 minutes, so hourly is already generous. Please do not
   speed it up.
3. **A snapshot, not a mirror.** poe.ninja ask people not to use the API to
   re-create their website. This saves a few fields per item and throws away
   the rest, on purpose.

It only ever touches poe.ninja's four **public, documented** economy addresses.
poe.ninja also have addresses for builds, characters and profiles — those are
**internal and explicitly not for outside use**, and this does not go near them.
Please do not add them.

---

## Setup

### Step 1 — Make a GitHub account

If you already have one, skip to Step 2.

Go to **https://github.com/signup** and follow it through. It is free. The free
plan is all you need.

### Step 2 — Create a new, **public** repository

1. Go to **https://github.com/new**
2. **Repository name**: type `ninja-relay` (any name works, but the rest of
   this guide assumes that one).
3. Choose **Public**.

   > **This must be Public.** Two reasons: on public repositories GitHub runs
   > scheduled jobs like this one for free, and the `raw.githubusercontent.com`
   > links work without a password. On a private repository you would pay for
   > the minutes and your tool could not read the files.
   >
   > There is nothing private in here — no passwords, no tokens, no account
   > details. Just prices.

4. Tick **Add a README file**. (You will replace it in a moment; this just
   makes the repository exist so you have somewhere to put things.)
5. Click the green **Create repository** button.

You are now looking at your new, empty repository.

### Step 3 — Add the four files

You will do this four times, once per file. The steps are identical each time.

For each file:

1. Click the **Add file** button (top right of the file list), then choose
   **Create new file** from the little menu.
2. In the box at the top that says **Name your file...**, type the filename
   *exactly* as given below — including any `/` slashes. **The slashes matter.**
   As you type a `/`, GitHub will turn the part before it into a folder. That is
   correct and expected.
3. Click into the big empty area underneath and paste the file's contents.
4. Click the green **Commit changes...** button (top right), then the green
   **Commit changes** button in the box that pops up.

The four files:

| # | Type this exact filename | Paste in the contents of |
|---|---|---|
| 1 | `snapshot.py` | `snapshot.py` |
| 2 | `.github/workflows/snapshot.yml` | `.github/workflows/snapshot.yml` |
| 3 | `LICENSE` | `LICENSE` |
| 4 | `README.md` | this file (replaces the placeholder from Step 2) |

> **On file 2:** the filename starts with a dot and has two slashes in it:
> `.github/workflows/snapshot.yml`. Type it character for character. GitHub
> will show it turning into folders as you go. If you get this one wrong,
> nothing will ever run — it is the only filename that has to be perfect.

> **If you already have `README.md`** from Step 2, don't use "Create new file"
> for it. Click on `README.md` in the file list, then the **pencil** icon
> (top right), select everything in the box and paste over it, then
> **Commit changes**.

### Step 4 — Turn on Actions

1. Click the **Actions** tab along the top of your repository.
2. If you see a green button saying **I understand my workflows, go ahead and
   enable them**, click it.
3. If you instead see a workflow called **Hourly poe.ninja snapshot** in the
   left-hand list, it is already on. Move to Step 5.

### Step 5 — Set your contact string (required)

This is the one piece of information only you can supply.

1. Click the **Settings** tab of your repository (top right, next to Insights).
2. In the left sidebar, click **Secrets and variables**, then **Actions**.
3. Click the **Variables** tab. (Not Secrets — **Variables**. This is not
   secret information; poe.ninja is meant to be able to read it.)
4. Click the green **New repository variable** button.
5. **Name**: `POE_NINJA_CONTACT`
6. **Value**: something that identifies you and could be used to reach you.
   Any one of these is fine:
   - your GitHub username, e.g. `github.com/yourname`
   - an email address, e.g. `you@example.com`
   - a Discord handle, e.g. `discord:yourname`
7. Click **Add variable**.

That's it. The script will now put that string into every request it makes.

> Skipping this does not "mostly work" — the script stops with a large message
> in the log telling you to come back and do it.

### Step 6 — Run it once, by hand

Don't wait an hour to find out whether it worked.

1. Click the **Actions** tab.
2. In the left sidebar, click **Hourly poe.ninja snapshot**.
3. On the right you will see a grey bar saying *"This workflow has a
   workflow_dispatch event trigger."* Click the **Run workflow** button on it.
4. A small panel opens. Leave the league box empty. Click the green
   **Run workflow** button inside it.
5. Wait about five seconds and refresh the page. A run appears with a spinning
   yellow dot. It takes roughly half a minute.

**A green tick** means it worked. Click the run to see a short summary — which
league it used, how many prices it saved.

**A red cross** means something went wrong. Click the run, then click the
`snapshot` box, and read the log. See [Troubleshooting](#troubleshooting).

### Step 7 — Look at your data

Click the **Code** tab. There is now a **`data`** folder. Inside:

| File | What's in it |
|---|---|
| `index.json` | **Start here.** Which league, when it was taken, what else exists, and anything that went wrong. |
| `currency.json` | Currency prices, from public stash listings. |
| `exchange.json` | Currency exchange rates. |
| `items.json` | Item prices, from public stash listings. |
| `http-cache.json` | Housekeeping. Lets the next run ask poe.ninja "has anything changed?" instead of re-downloading everything. Ignore it. |

Done. It will now update itself every hour.

---

## The link to hand to your tool

This is the thing you came for. Replace the two capitalised parts:

```
https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/data/index.json
```

- `YOUR-USERNAME` → your GitHub username
- `YOUR-REPO` → the repository name from Step 2 (`ninja-relay` if you followed along)

So if your username were `exampleuser` and you named it `ninja-relay`, it would be:

```
https://raw.githubusercontent.com/exampleuser/ninja-relay/main/data/index.json
```

The other files follow the same pattern — just swap the last part:

```
https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/data/currency.json
https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/data/exchange.json
https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/data/items.json
```

**Give your tool the `index.json` link.** It describes the others, so one link
is enough.

> **How to get the link without typing it:** in the **Code** tab, click into the
> `data` folder, click `index.json`, then click the **Raw** button (top right of
> the file view). The address bar now shows exactly the right URL. Copy it.

> Note: `main` in the URL is the branch name. GitHub uses `main` for all new
> repositories, so this will be right unless you deliberately changed it.

---

## What the data looks like

`index.json` is the table of contents:

```json
{
  "ok": true,
  "generatedAt": "2026-08-23T14:17:03Z",
  "league": "Mercenaries",
  "leagueSource": "discovered from /leagues",
  "files": [
    { "file": "items.json", "category": "items", "records": 1200, "status": "updated" }
  ],
  "errors": []
}
```

The price files each hold a list of trimmed records:

```json
{
  "league": "Mercenaries",
  "recordCount": 1200,
  "records": [
    {
      "name": "Mageblood",
      "base": "Heavy Belt",
      "chaos": 96000,
      "divine": 480,
      "listings": 42,
      "id": "mageblood",
      "link": "https://poe.ninja/poe1/economy/mercenaries/items/mageblood"
    }
  ]
}
```

`listings` is how many listings the price is based on — treat a price built on
three listings with more suspicion than one built on three hundred. `link` is
best-effort; `id` is poe.ninja's own identifier if you want to build your own.

Records are sorted most-expensive first, and the list is capped (1200 per file
by default) so the repository stays small. The cheap tail is what gets dropped.

---

## When a new league starts

**Normally: do nothing.** The script asks poe.ninja which leagues exist and
picks the current temporary challenge league by itself, precisely so that it
does not break every three months. Within an hour of poe.ninja listing the new
league, your data will be for the new league. You can check which one it chose
in `index.json` under `"league"`.

**If you want to pin it to a specific league** (for example to keep watching
Standard, or because the auto-detection guessed wrong during the messy first
hours of a league launch):

1. **Settings** → **Secrets and variables** → **Actions** → **Variables** tab
2. **New repository variable**
3. Name: `POE_NINJA_LEAGUE`
4. Value: the exact league name, e.g. `Standard`
5. **Add variable**

To go back to automatic, delete that variable (bin icon next to it).

**For a one-off run against a different league**, you don't need a variable at
all: **Actions** → **Hourly poe.ninja snapshot** → **Run workflow**, and type
the league name into the box before clicking the green button.

---

## Turning it off

**Pause it** (keeps everything, easy to undo):

**Actions** tab → **Hourly poe.ninja snapshot** in the sidebar → the **`...`**
menu at the top right → **Disable workflow**. The same menu re-enables it.

**Get rid of it entirely:**

**Settings** → scroll to the very bottom → **Danger Zone** →
**Delete this repository**.

---

## Troubleshooting

**"STOP: no contact string is set"** in the log
Step 5 was missed, or the variable name is misspelled. It must be exactly
`POE_NINJA_CONTACT`, under the **Variables** tab (not Secrets).

**Red cross, and the log mentions HTTP 403 or 429**
poe.ninja is rate-limiting or blocking. Don't retry in a loop. Leave it alone
for a few hours; the next hourly run will try again on its own.

**Green tick, but `index.json` has things in `"errors"`**
Working as designed. One category failing does not sink the run — the rest of
the data is still good, and the failure is written down so you can see it. If
the same category fails for days, poe.ninja may have changed something.

**`"league"` looks wrong just after a league launch**
poe.ninja sometimes lists the new league before there is data in it. Give it a
few hours, or pin it with `POE_NINJA_LEAGUE` as above.

**Nothing has run for weeks**
GitHub switches off scheduled jobs in repositories that have had no *human*
activity for 60 days — the hourly bot commits do not always count. GitHub emails
you before it happens. To fix: **Actions** tab → **Enable workflow**. To avoid
it, visit the repository and make any small edit every couple of months.

**The runs are green but `data` never appears**
Check the workflow file's name is exactly `.github/workflows/snapshot.yml`.
A typo there is by far the most common cause.

**The raw URL gives "404: Not Found"**
Either the username/repository in the URL is wrong, or the workflow has not
completed a successful run yet, or the repository is Private. Use the **Raw**
button trick above to get a URL you know is correct.

---

## For developers (and for AI assistants reading this later)

- `snapshot.py` is **standard library only**. No `pip install`, no
  `requirements.txt`, nothing to keep updated.
- **Every network call goes through one function**, `http_get_json()`. That is
  deliberate — it is the only part that cannot be tested where poe.ninja is
  unreachable, so it is kept small and everything else is tested around it.
- `python3 snapshot.py --selftest` runs the entire trim-and-write path against
  a built-in fake payload and asserts the output shape. It makes **no network
  calls** and takes about a second. The workflow runs it before every real
  fetch. Run it after any change you make.
- Other flags: `--league NAME` to force a league, `--data-dir PATH` to write
  somewhere else, `--summary` to print a report of the last run.
- Files are written with a temp-file-then-rename, so a crash can never leave a
  half-written JSON file for a consumer to trip over.
- A category file is only rewritten when its *contents* actually change — the
  timestamp alone is not enough. Otherwise the repository would collect a
  full-file diff every hour forever.
- The parsing is deliberately forgiving about field names (`chaosValue` /
  `chaosEquivalent` / `chaos` are all accepted for the same idea) because the
  API's exact shape is not a frozen contract.
- **Do not add the builds, characters, profiles or Path-of-Building
  endpoints.** They are internal and poe.ninja have explicitly said they are not
  for third-party use. This is not a technical limitation to route around.

---

## Licence and attribution

The **code** here is MIT licensed — see [LICENSE](LICENSE). Do what you like
with it.

The **data** is not mine to license. It belongs to **poe.ninja**, who collect
and publish it. This repository is only a courier.

If you use it, please:

- **Say where the numbers came from.** Credit poe.ninja anywhere you show them.
- **Keep the contact string honest.** It is the only channel poe.ninja has to
  reach you before they block you. A fake one converts "someone would have sent
  you an email" into "you are blocked and don't know why".
- **Don't speed up the schedule**, and don't widen it into a full mirror of
  their site.

This project is not affiliated with, endorsed by, or connected to poe.ninja or
Grinding Gear Games. Path of Exile is a trademark of Grinding Gear Games.
