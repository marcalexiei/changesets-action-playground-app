# `changesets-action-playground-app`

This repository is an improved version of [changesets-action-playground-pat](https://github.com/marcalexiei/changesets-action-playground-pat).

## Overview

The main difference is how authentication is handled.  
Instead of relying on a Personal Access Token (PAT), this project uses a GitHub App.  

By doing so, the release workflow can generate tokens on demand using the [create-github-app-token Action](https://github.com/marketplace/actions/create-github-app-token), providing a more secure and maintainable setup.

## References

- [Configure Git CLI for a GitHub App’s bot user](https://github.com/marketplace/actions/create-github-app-token#configure-git-cli-for-an-apps-bot-user)
