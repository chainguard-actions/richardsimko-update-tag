# GitHub Update Tag Action
A GitHub action that simply tags the repository with the specified tag. If the tag exists it gets updated.

## Usage
```yml
name: Deploy

on: [deployment]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Tag Repo
        uses: richardsimko/update-tag@v1
        with:
          tag_name: name-of-tag
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

- **tag_name** _(required)_ - The name of the tag you want to create or update.

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
