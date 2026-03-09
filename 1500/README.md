# 1500 — CI/CD Integration

## Overview

Fly.io integrates with any CI/CD system via `flyctl` and a **deploy token**. The pattern is:

1. Create a deploy token for your app
2. Store it as a secret in your CI/CD system
3. Add a deploy step that runs `fly deploy`

---

## Step 1: Create a Deploy Token

```bash
fly tokens create deploy -a my-app -x 30d   # Expires in 30 days
fly tokens create deploy -a my-app           # Never expires
```

Copy the token value — you'll only see it once.

---

## Step 2: Add Token to CI/CD

In your CI/CD provider, add the token as a secret named `FLY_API_TOKEN`.

---

## GitHub Actions

### Basic deploy workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Fly.io

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: superfly/flyctl-actions/setup-flyctl@master
      
      - name: Deploy
        run: fly deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

### With test step before deploy

```yaml
name: Test & Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: fly deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

### Preview environments (per PR)

```yaml
name: PR Preview

on:
  pull_request:
    types: [opened, synchronize, closed]

jobs:
  preview:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      
      - name: Deploy Preview
        if: github.event.action != 'closed'
        run: |
          fly apps create my-app-pr-${{ github.event.number }} || true
          fly deploy -a my-app-pr-${{ github.event.number }} --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
      
      - name: Destroy Preview
        if: github.event.action == 'closed'
        run: fly apps destroy my-app-pr-${{ github.event.number }} -y
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

---

## GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - deploy

deploy:
  stage: deploy
  image: ruby:slim
  before_script:
    - curl -L https://fly.io/install.sh | sh
    - export PATH="$HOME/.fly/bin:$PATH"
  script:
    - fly deploy --remote-only
  only:
    - main
  variables:
    FLY_API_TOKEN: $FLY_API_TOKEN
```

---

## CircleCI

```yaml
# .circleci/config.yml
version: 2.1

jobs:
  deploy:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - run:
          name: Install flyctl
          command: curl -L https://fly.io/install.sh | sh
      - run:
          name: Deploy
          command: ~/.fly/bin/fly deploy --remote-only
          environment:
            FLY_API_TOKEN: $FLY_API_TOKEN

workflows:
  deploy:
    jobs:
      - deploy:
          filters:
            branches:
              only: main
```

---

## Deploy Flags for CI

```bash
fly deploy --remote-only        # Build on Fly servers (no local Docker needed)
fly deploy --local-only         # Build locally (requires Docker on CI runner)
fly deploy --now                # Skip confirmation prompts
fly deploy --wait-timeout 120   # Fail if deploy takes > 120s
fly deploy --strategy rolling   # Explicit strategy
```

---

## Managing Secrets in CI

Set secrets from CI without exposing them in logs:

```bash
fly secrets set MY_KEY="${MY_KEY_FROM_CI}"
```

Or batch-import:
```bash
echo "KEY1=val1
KEY2=val2" | fly secrets import
```

---

## Tasks

### Task 1 — GitHub Actions deploy
1. Create a deploy token: `fly tokens create deploy -a my-app`.
2. Add it to your GitHub repo: Settings → Secrets → `FLY_API_TOKEN`.
3. Add `.github/workflows/deploy.yml` with the basic deploy workflow.
4. Push to main and watch the Actions tab.

### Task 2 — PR preview environment
1. Add the PR preview workflow.
2. Open a test PR.
3. Confirm a preview app was created: `fly apps list | grep pr-`.
4. Merge or close the PR and confirm the preview is destroyed.

---

## Next Steps

→ Continue to [`1600`](../1600/README.md) — Multi-Region & Distributed Apps
