# `changesets-action-playground-app`

This repository is an improved version of [changesets-action-playground-pat](https://github.com/marcalexiei/changesets-action-playground-pat).

## Overview

The main difference is how authentication is handled.  
Instead of relying on a Personal Access Token (PAT), this project uses a GitHub App.  

By doing so, the release workflow can generate tokens on demand using the [create-github-app-token Action](https://github.com/marketplace/actions/create-github-app-token), providing a more secure and maintainable setup.

## Usage

### Configuration

1. Install the app in the repository
2. Set the app id as GitHub actions `RELEASE_HELPER_APP_ID` variable
3. Set the app private key as GitHub actions `RELEASE_HELPER_PRIVATE_KEY` secret

### Implementation

1. Add the following steps before checkout action call

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

2. Ensure that `actions/checkout` is called with `persist-credentials: false`

   ```yml
   - name: Checkout
     uses: actions/checkout@v5
     with:
       persist-credentials: false
   ```

3. Changeset action must be called with `setupGitUser: false`

   As `GITHUB_TOKEN` use `steps.app-token.outputs.token`

   ```yml
   - name: Create Release Pull Request or Publish to npm
     id: changesets
     uses: changesets/action@v1
     with:
       commit: "chore: release"
       title: "chore: release"
       publish: "pnpm release"
       setupGitUser: false
     env:
       GITHUB_TOKEN: ${{ steps.app-token.outputs.token }}
       NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
   ```

## References

- [Configure Git CLI for a GitHub App’s bot user](https://github.com/marketplace/actions/create-github-app-token#configure-git-cli-for-an-apps-bot-user)
