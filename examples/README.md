# Example Workflow Files

This directory contains example GitHub Actions workflow files for automatically publishing to CurseForge.

## Available Examples

### 1. `release-basic.yml`
**Use case**: General-purpose workflow for simple addons or mods
**Features**:
- Manual packaging with zip
- Direct upload to CurseForge
- Customizable for any game

**Best for**: 
- Simple addons with straightforward file structures
- Projects that need custom packaging logic
- Learning how the process works from scratch

### 2. `release-bigwigs.yml`
**Use case**: World of Warcraft addons
**Features**:
- Uses the popular BigWigsMods packager
- Automatic .toc file processing
- Automatic version substitution
- Supports multiple distribution platforms (CurseForge, WoWInterface, Wago, GitHub)

**Best for**:
- WoW addon developers
- Projects that follow standard WoW addon structure
- Addons that need to be published to multiple platforms

### 3. `release-minecraft.yml`
**Use case**: Minecraft mods (Fabric/Forge)
**Features**:
- Java/Gradle build setup
- Automatic JAR building
- Upload to CurseForge

**Best for**:
- Minecraft mod developers
- Projects using Gradle build system
- Fabric or Forge mods

## How to Use These Examples

1. **Choose the appropriate example** based on your project type
2. **Copy the workflow file** to `.github/workflows/` in your repository
3. **Customize the placeholders**:
   - Replace `YOUR_PROJECT_ID_HERE` with your CurseForge project ID
   - Update `YourAddonName` or `yourmod` with your actual addon/mod name
   - Adjust `game_versions` to match your supported versions
   - Modify packaging commands if needed
4. **Set up your GitHub secret**:
   - Add `CURSEFORGE_TOKEN` to your repository secrets
5. **Test** by creating a new release

## Customization Tips

### Changing the Trigger
By default, these workflows trigger on published releases. You can modify the trigger:

```yaml
# Trigger on push to main branch
on:
  push:
    branches: [main]

# Trigger on tag creation
on:
  push:
    tags:
      - 'v*'

# Trigger manually
on:
  workflow_dispatch:
```

### Adding Multiple Games or Platforms
You can modify workflows to publish to multiple platforms:

```yaml
- name: Upload to CurseForge
  uses: itsmeow/curseforge-upload@v3
  with:
    token: ${{ secrets.CURSEFORGE_TOKEN }}
    project_id: ${{ secrets.CF_PROJECT_ID }}
    # ... other config

- name: Upload to Modrinth (for Minecraft)
  uses: RubixDev/modrinth-upload@v1
  with:
    token: ${{ secrets.MODRINTH_TOKEN }}
    # ... other config
```

### Conditional Release Types
You can set the release type based on the tag:

```yaml
- name: Determine release type
  id: release_type
  run: |
    if [[ "${{ github.event.release.tag_name }}" == *"alpha"* ]]; then
      echo "type=alpha" >> $GITHUB_OUTPUT
    elif [[ "${{ github.event.release.tag_name }}" == *"beta"* ]]; then
      echo "type=beta" >> $GITHUB_OUTPUT
    else
      echo "type=release" >> $GITHUB_OUTPUT
    fi

- name: Upload to CurseForge
  uses: itsmeow/curseforge-upload@v3
  with:
    release_type: ${{ steps.release_type.outputs.type }}
    # ... other config
```

## Need Help?

If you need help customizing these workflows for your specific use case:
1. Check the main [README](../README.md) for detailed instructions
2. Review the [GitHub Actions documentation](https://docs.github.com/en/actions)
3. Check the documentation for specific actions:
   - [itsmeow/curseforge-upload](https://github.com/itsmeow/curseforge-upload)
   - [BigWigsMods/packager](https://github.com/BigWigsMods/packager)
4. Open an issue in this repository with your question
