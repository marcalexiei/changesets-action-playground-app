# `changesets-action-playground-app`

This repository is an improved version of [changesets-action-playground-pat](https://github.com/marcalexiei/changesets-action-playground-pat).

## Overview

The main difference is how authentication is handled.
Instead of relying on a Personal Access Token (PAT), this project uses a GitHub App.

By doing so, the release workflow can generate tokens on demand using the [create-github-app-token Action](https://github.com/marketplace/actions/create-github-app-token), providing a more secure and maintainable setup.

## Reason

When you use a repository’s `GITHUB_TOKEN` to perform actions,
any events triggered by that token, except for `workflow_dispatch` and `repository_dispatch`, will **not** start a new workflow run.

This behavior prevents recursive workflow executions.

For example, if a workflow uses the repository’s `GITHUB_TOKEN` to push code changes,
it will not trigger another workflow run even if the repository has a workflow configured to run on `push` events.

According to this, marking a GitHub Actions workflow as required in [rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) becomes tricky:\
you can't merge PRs opened by GitHub Actions bot without a bypass rule.

## Usage

### Create a GitHub App

1. Create a new GitHub App.

2. Configure the following permissions:
   - **Contents** → Read & Write
   - **Metadata** → Read-only (automatically granted when modifying contents, listed here for completeness)
   - **Pull requests** → Read & Write

3. Fill all other required information.\
   No callback URl or webhook are required to be active.

4. Generate a new private key and store it securely (for example, in a password manager).

---

### Configure the App in the Repository

1. Install the GitHub App on the target repository.
2. Add the following to your GitHub Actions settings:
   - **Variable:** `RELEASE_HELPER_APP_ID` → set to the App ID.
   - **Secret:** `RELEASE_HELPER_PRIVATE_KEY` → set to the App’s private key.

### Use app in the workflows

1. Add the following steps before checkout action call:

   ```yml
   - name: Create access token for GitHub App
     uses: actions/create-github-app-token@v2
     id: app-token
     with:
      app-id: ${{ vars.RELEASE_HELPER_APP_ID }}
      private-key: ${{ secrets.RELEASE_HELPER_PRIVATE_KEY }}

   - name: Get GitHub App User ID
      id: app-user-id
      run: echo "user-id=$(gh api "/users/${{ steps.app-token.outputs.app-slug }}[bot]" --jq .id)" >> "$GITHUB_OUTPUT"
      env:
         GH_TOKEN: ${{ steps.app-token.outputs.token }}

   - name: Configure git user for GitHub App
      run: |
         git config --global user.name '${{ steps.app-token.outputs.app-slug }}[bot]'
         git config --global user.email '${{ steps.app-user-id.outputs.user-id }}+${{ steps.app-token.outputs.app-slug }}[bot]@users.noreply.github.com'
   ```

2. Ensure that `actions/checkout` is called with `persist-credentials: false`:

   ```yml
   - name: Checkout
     uses: actions/checkout@v5
     with:
       persist-credentials: false
   ```

3. Changeset action must be called with `setupGitUser: false`.

   As `GITHUB_TOKEN` use `steps.app-token.outputs.token`.

   ```yml
   - name: Create Release Pull Request or Publish to npm
     id: changesets
     uses: changesets/action@v1
     with:
       commit: 'chore: release'
       title: 'chore: release'
       publish: 'pnpm release'
       setupGitUser: false
     env:
       GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
       # ...
   ```

## References

- [When `GITHUB_TOKEN` triggers workflow runs](https://docs.github.com/en/actions/concepts/security/github_token#when-github_token-triggers-workflow-runs)
- [Configure Git CLI for a GitHub App’s bot user](https://github.com/marketplace/actions/create-github-app-token#configure-git-cli-for-an-apps-bot-user)
