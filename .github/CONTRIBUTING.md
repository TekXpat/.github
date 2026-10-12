# Contributing

This is how changes get into a TekXpat repository. If a repository has its own `CONTRIBUTING.md`, that one wins. Its README covers setup.

## Branches

```
feature/<short-description>  →  dev  →  main
```

- **`main`** is what is released. It only moves by merging `dev`.
- **`dev`** is where finished work comes together and is checked before release.
- **`feature/*`** is where you work. Branch from `dev`, and give it a name a stranger could understand: `feature/invoice-pdf-export`, not `feature/fix2`.

Nobody commits directly to `dev` or `main`, including maintainers and including one-line fixes. This holds whether or not the repository enforces it.

**Why:** `dev` and `main` must always be in a state someone can deploy. A direct commit skips the two things that keep them that way, CI and a second pair of eyes. A one-line fix is exactly what people don't bother to check.

## Pull requests

Every change reaches `dev` through a pull request, and so does every promotion from `dev` to `main`. A pull request can be merged when:

1. **CI is green.** Not "green apart from a flaky test". If a test is flaky, fix it or raise an issue for it; don't merge over it.
2. **One other person has approved it.**
3. **The description explains why.** The diff already shows *what* changed. The reviewer, and whoever runs `git blame` in a year, needs the reason, the alternatives you ruled out, and anything you're unsure about.

Keep pull requests small. A 200-line PR gets a real review; a 2,000-line one gets a skim and an approval. If a change is large, split it into steps that each make sense alone.

Open a draft PR early if you want feedback on direction. It's much cheaper to change course on day one than on day five.

## Commits

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add PDF export to invoices
fix: stop double-charging on payment retry
docs: explain tenant setup in README
refactor: extract currency formatting
test: cover expired-token refresh
chore: bump dependencies
```

One commit, one idea. The subject line says what the commit does, in the present tense, in under about 72 characters.

**Why:** small, labelled commits make history readable, let a bad change be reverted without dragging good ones with it, and make release notes nearly write themselves.

## Reviewing

As a reviewer you are checking that:

- the change does what the description says, and the description's reasoning holds up
- it is tested where it matters (see below)
- it doesn't break one of the rules below
- someone new to the code could follow it

Be direct and specific, and say whether a comment blocks the merge or is a suggestion. As the author, respond to every comment, even if only to say "done" or "I disagree, because…".

## Testing

- New behaviour comes with tests.
- A bug fix comes with a test that fails without the fix. That is how we know it was the bug, and that it stays fixed.
- Test behaviour, not implementation details. A refactor shouldn't need test changes.

## Rules we don't bend

Each of these guards against a failure that is easy to cause and hard to notice.

1. **An environment variable that sets a public URL fails the build if it isn't set.** Never fall back to a placeholder domain. A wrong canonical URL quietly removes a whole site from search results, and nothing warns you.
2. **Fonts are self-hosted, never loaded from `next/font/google`.** Build environments can't always reach external font services, and when they can't the build fails with an error that tells you nothing.
3. **No handler reports success when a downstream call failed.** If a payment, webhook or email fails, the user is told and offered another way forward. A silent failure is worse than a visible error.
4. **Money is a decimal with an explicit currency, never a float.** Floats can't represent most decimal amounts exactly, and the rounding errors add up.
5. **Secrets are read at runtime and never prefixed `NEXT_PUBLIC_`.** That prefix compiles the value into the JavaScript sent to every visitor's browser.
6. **Multi-tenant data is accessed through the tenant-scoped client, never the raw ORM.** A query missing its tenant filter is a data breach, not a bug, and it looks completely normal in review.

## Environment variables

Any new or renamed variable goes into `.env.example` in the same pull request, with a comment saying what it is for. Never put real values in `.env.example`.

## Security

Never report a vulnerability in a public issue. See [SECURITY.md](SECURITY.md).
