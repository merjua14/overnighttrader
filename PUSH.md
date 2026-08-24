# Pushing this to GitHub

The git repo in this folder is **already initialized and committed** on branch `main`. You only need to point it at GitHub and push.

## Step 1: create the empty repo

Go to [github.com/new](https://github.com/new) and create `overnighttrader` under the `merjua14` account.

**Leave "Add a README file", ".gitignore", and "Choose a license" all unchecked.** If GitHub creates any of those, your first push will be rejected as a non-fast-forward and you will have to merge unrelated histories to fix it.

## Step 2: push

```bash
git remote add origin https://github.com/merjua14/overnighttrader.git
git push -u origin main
```

If it asks for a password, GitHub does not accept account passwords over HTTPS anymore. Use a personal access token as the password ([github.com/settings/tokens](https://github.com/settings/tokens), classic token with `repo` scope), or switch the remote to SSH:

```bash
git remote set-url origin git@github.com:merjua14/overnighttrader.git
```

## If you would rather use the GitHub CLI

`gh` does both steps at once, from inside this folder:

```bash
gh repo create merjua14/overnighttrader --public --source=. --remote=origin --push
```

## After it is up

The repo URL in POST.md is already set to `github.com/merjua14/overnighttrader`, so the thread copy is ready to use as-is.

Two things worth doing on the GitHub page itself, since they are the first thing anyone sees:

- **Description:** `Buy at the close, sell at the open, automatically. One paste, no code. Read RISKS.md first.`
- **Topics:** `trading`, `robinhood`, `mcp`, `claude`, `automation`, `overnight-returns`

## Making changes later

```bash
git add -A
git commit -m "what you changed"
git push
```

One caution: the paste block lives in two places, inside README.md and as prompts/SETUP.txt. They are byte-identical right now. If you edit one, edit the other, or delete SETUP.txt and let the README be the single source.
