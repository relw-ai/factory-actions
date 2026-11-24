# factory-actions

**Dark Factory: verified by your CI.** One reusable workflow that runs the factory's checks for your stack
in **your** GitHub Actions (your packages, your services, your secrets stay with you), then proves the
result to the Dark Factory with **GitHub's OIDC identity token**.

## Use it

Add `.github/workflows/factory-verify.yml` to your repository:

```yaml
name: factory-verify
on: [push, pull_request]
jobs:
  verify:
    uses: relw-ai/factory-actions/.github/workflows/verify.yml@v1
    permissions:
      id-token: write    # lets GitHub vouch for this run to the factory
      contents: read
    with:
      stack: typescript  # or: python
```

Your repository must also be **registered** with the factory (ask Relw.AI). Unregistered repositories are refused.

## What runs

| Stack | Install (from your lockfile only) | Checks |
|---|---|---|
| `python` | `uv sync --frozen`, or `requirements.txt`, or `pyproject.toml` | `ruff check .`, `pytest -q` |
| `typescript` | `bun install --frozen-lockfile`, `pnpm install --frozen-lockfile` or `npm ci`, **install scripts off** | `tsc --noEmit`, `eslint .`, `vitest run` |

The inputs choose the **stack**, never the **commands**: a check that could be configured away would prove nothing.

## What leaves your CI

To `https://df.relw.ai/ci/report`, once per run:
- the repository, the commit, and the **sha256 of each tracked file** (no file contents);
- each check's **exit code** and **one summary line**, at most 200 characters (e.g. `3 passed`). Full logs stay in your CI;
- a GitHub OIDC token addressed to `df.relw.ai`, valid for about five minutes and usable once.

## What the factory can then say, and what it cannot

It can say: GitHub vouches that **this workflow, at this version, ran on this commit of this repository, and the checks
exited with these codes.** It cannot say the tests are good tests. That remains a review obligation.

Why an identity token and not GitHub "artifact attestations": attestations on private repositories need a paid GitHub
plan, tested 2026-11-23. The identity token works on every plan.
