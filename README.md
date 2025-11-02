# Auto-Publish from GitHub to CurseForge

A comprehensive walkthrough on how to automatically package and publish your addons to CurseForge using GitHub Actions.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Get Your CurseForge API Token](#step-1-get-your-curseforge-api-token)
- [Step 2: Add the API Token to GitHub Secrets](#step-2-add-the-api-token-to-github-secrets)
- [Step 3: Create the GitHub Actions Workflow](#step-3-create-the-github-actions-workflow)
- [Step 4: Configure Your Addon Metadata](#step-4-configure-your-addon-metadata)
- [Step 5: Trigger a Release](#step-5-trigger-a-release)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Overview

This guide will show you how to set up automatic publishing of your addon to CurseForge whenever you create a new release on GitHub. This eliminates manual uploading and ensures your addon is always up-to-date on CurseForge.

**What you'll accomplish:**
- Automatic packaging of your addon when you create a GitHub release
- Automatic upload to CurseForge with release notes
- Version synchronization between GitHub and CurseForge

## Prerequisites

Before you begin, make sure you have:

1. **A GitHub repository** containing your addon code
2. **A CurseForge project** for your addon (create one at [CurseForge Authors](https://authors.curseforge.com/))
3. **Basic knowledge** of Git and GitHub
4. **Your addon properly structured** (e.g., for WoW addons: `.toc` file, for Minecraft mods: proper mod structure)

## Step 1: Get Your CurseForge API Token

To allow GitHub to publish to CurseForge on your behalf, you need an API token.

1. Navigate to [CurseForge Core API Console](https://console.curseforge.com/)
2. Log in with your CurseForge account
3. Click on **"Generate API Token"** or go to the API Tokens section
4. Create a new token with a descriptive name (e.g., "GitHub Actions Publisher")
5. **Copy the token immediately** - you won't be able to see it again!

![CurseForge API Token Generation](docs/images/curseforge-api-token.png)
*Screenshot: Generating an API token in the CurseForge Console*

**Important:** Keep this token secure and never commit it directly to your repository.

**Useful Links:**
- [CurseForge API Documentation](https://support.curseforge.com/en/support/solutions/articles/9000197321-curseforge-api)
- [CurseForge API Console](https://console.curseforge.com/)

## Step 2: Add the API Token to GitHub Secrets

GitHub Secrets allow you to store sensitive information securely.

1. Go to your GitHub repository
2. Click on **Settings** (top navigation bar)
3. In the left sidebar, click **Secrets and variables** → **Actions**
4. Click **New repository secret**
5. Name the secret `CURSEFORGE_TOKEN` (or whatever name you prefer)
6. Paste your CurseForge API token in the value field
7. Click **Add secret**

![GitHub Secrets Configuration](docs/images/github-secrets.png)
*Screenshot: Adding a secret in GitHub repository settings*

**Useful Links:**
- [GitHub Encrypted Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## Step 3: Create the GitHub Actions Workflow

GitHub Actions workflows are defined in YAML files stored in the `.github/workflows` directory.

### 3.1 Create the Workflow Directory

In your repository, create the directory structure:
```
.github/
  workflows/
```

### 3.2 Create the Workflow File

Create a file named `.github/workflows/release.yml` with the following content:

```yaml
name: Release to CurseForge

on:
  release:
    types: [published]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Package addon
        run: |
          # Create a directory for the packaged addon
          mkdir -p release
          # Copy addon files (adjust this based on your addon structure)
          # Example for WoW addon:
          cp -r . release/YourAddonName
          cd release
          zip -r ../YourAddonName-${{ github.event.release.tag_name }}.zip YourAddonName -x "*.git*" "*.github*" "release/*"
        
      - name: Upload to CurseForge
        uses: itsmeow/curseforge-upload@v3
        with:
          token: ${{ secrets.CURSEFORGE_TOKEN }}
          project_id: YOUR_PROJECT_ID_HERE
          game_endpoint: wow  # Change to 'minecraft' for Minecraft mods
          file_path: YourAddonName-${{ github.event.release.tag_name }}.zip
          changelog: ${{ github.event.release.body }}
          changelog_type: markdown
          game_versions: 1.14.3,1.14.4  # Update with your supported versions
          release_type: release  # Can be 'release', 'beta', or 'alpha'
```

### 3.3 Customize the Workflow

You need to customize several values in the workflow:

1. **`YourAddonName`**: Replace with your actual addon name
2. **`YOUR_PROJECT_ID_HERE`**: Replace with your CurseForge project ID (find it in your project URL or settings)
3. **`game_endpoint`**: Set to `wow`, `minecraft`, or other supported game
4. **`game_versions`**: List the game versions your addon supports
5. **Packaging commands**: Adjust the `zip` command based on your addon structure

**Finding Your CurseForge Project ID:**
- Go to your project on CurseForge
- Look at the URL: `https://www.curseforge.com/wow/addons/YOUR-ADDON-NAME`
- Or find it in your project settings under "About Project"

![Finding CurseForge Project ID](docs/images/curseforge-project-id.png)
*Screenshot: Locating your project ID on CurseForge*

**Useful Links:**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [CurseForge Upload Action](https://github.com/itsmeow/curseforge-upload)

### 3.4 Alternative: Using BigWigsMods/packager

For WoW addons, you can use the popular BigWigsMods packager:

```yaml
name: Release to CurseForge

on:
  release:
    types: [published]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Needed for proper packaging

      - name: Package and release
        uses: BigWigsMods/packager@v2
        env:
          CF_API_KEY: ${{ secrets.CURSEFORGE_TOKEN }}
        with:
          args: -p YOUR_PROJECT_ID
```

**Useful Links:**
- [BigWigsMods Packager Documentation](https://github.com/BigWigsMods/packager)
- [WoW Addon Packaging Guide](https://github.com/BigWigsMods/packager/wiki)

## Step 4: Configure Your Addon Metadata

For the automation to work properly, your addon should have proper metadata files.

### For WoW Addons

Ensure you have a `.toc` file with proper version information:

```toc
## Interface: 110002
## Title: Your Addon Name
## Version: @project-version@
## Notes: Description of your addon
## Author: Your Name
```

The `@project-version@` will be automatically replaced by the packager with your release tag.

### For Minecraft Mods

Ensure your `fabric.mod.json` or `mods.toml` (for Forge) has proper version information:

```json
{
  "schemaVersion": 1,
  "id": "yourmod",
  "version": "${version}",
  "name": "Your Mod Name",
  "description": "Mod description"
}
```

## Step 5: Trigger a Release

Now that everything is configured, you can trigger a release:

### 5.1 Create a Git Tag

```bash
git tag v1.0.0
git push origin v1.0.0
```

### 5.2 Create a GitHub Release

1. Go to your repository on GitHub
2. Click on **Releases** (right sidebar)
3. Click **Create a new release** or **Draft a new release**
4. Choose your tag (e.g., `v1.0.0`)
5. Enter a release title (e.g., "Version 1.0.0")
6. Write release notes in the description (these will be used as the CurseForge changelog)
7. Click **Publish release**

![Creating a GitHub Release](docs/images/github-release.png)
*Screenshot: Creating a new release on GitHub*

### 5.3 Monitor the Workflow

1. Go to the **Actions** tab in your repository
2. You should see your workflow running
3. Click on it to see detailed logs
4. If everything succeeds, your addon will be published to CurseForge!

![GitHub Actions Workflow](docs/images/github-actions.png)
*Screenshot: Monitoring the workflow in GitHub Actions*

## Troubleshooting

### Workflow Fails with "Authentication Error"
- **Solution**: Double-check that your `CURSEFORGE_TOKEN` secret is correctly set and matches the name used in your workflow file.

### "Project ID not found" Error
- **Solution**: Verify your CurseForge project ID is correct. You can find it in your project's URL or settings.

### Package/ZIP is Empty or Missing Files
- **Solution**: Review your packaging commands in the workflow. Make sure you're including all necessary files and excluding unnecessary ones (`.git`, etc.).

### Wrong Game Version Listed
- **Solution**: Update the `game_versions` field in your workflow to match the versions your addon supports.

### Changelog Not Appearing
- **Solution**: Make sure you're writing release notes when creating the GitHub release. These notes become the CurseForge changelog.

### Workflow Doesn't Trigger
- **Solution**: Ensure the workflow file is in `.github/workflows/` and is named with a `.yml` or `.yaml` extension. Also verify the `on: release: types: [published]` trigger is correct.

## Additional Resources

### Documentation
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [CurseForge API Documentation](https://support.curseforge.com/en/support/solutions/articles/9000197321-curseforge-api)
- [GitHub Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

### Tools and Actions
- [CurseForge Upload Action](https://github.com/itsmeow/curseforge-upload)
- [BigWigsMods Packager](https://github.com/BigWigsMods/packager) (for WoW addons)
- [GitHub CLI](https://cli.github.com/) (for managing releases from command line)

### Community Resources
- [CurseForge Discord](https://discord.gg/curseforge)
- [GitHub Community Forum](https://github.community/)
- [WoW Interface Forums](https://www.wowinterface.com/forums/)

### Example Repositories
- [Example WoW Addon with Auto-Publishing](https://github.com/BigWigsMods/BigWigs)
- [Example Minecraft Mod with Auto-Publishing](https://github.com/FabricMC/fabric-example-mod)

## Contributing

If you find any issues with this guide or have suggestions for improvement, please open an issue or submit a pull request!

## License

This documentation is provided as-is for educational purposes. Feel free to use and adapt it for your own projects.
