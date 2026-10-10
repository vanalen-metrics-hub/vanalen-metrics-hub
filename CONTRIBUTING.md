# Contributing

## Branches

- `main`: demo-ready only. Merged from `dev` by the tech lead.
- `dev`: integration branch. All feature PRs target `dev`.
- Work branches: `feat/short-name`, `fix/short-name`, `docs/short-name`, `chore/short-name`

## Flow

1. `git checkout dev && git pull`
2. `git checkout -b feat/quote-library`
3. Commit small, clear messages: `feat: add quote tagging script`
4. Push, open a PR into `dev`, get 1 approval + passing checks, squash merge
5. Never push directly to `main` or `dev`. Branch rules block it.

## Data rule
This repo is public. Real Van Alen data never goes in it. Use fake sample data.
