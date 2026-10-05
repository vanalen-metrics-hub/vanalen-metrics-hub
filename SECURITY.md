# Security

This repository is public.

- Never commit secrets. Use `.env` locally and GitHub Actions secrets in CI.
- Never commit Van Alen data (spreadsheets, partner info, community quotes).
  Use fake sample data only. The `data/` folder is gitignored.
- If a secret is committed: tell the tech lead immediately, ROTATE the key first,
  then clean history. Deleting the file does not remove it from git history.
