# Quick Start Guide

**Want to get started immediately?** Follow these 5 quick steps:

## 1. Get Your CurseForge API Token
- Go to https://console.curseforge.com/
- Generate a new API token
- Copy it (you won't see it again!)

## 2. Add Token to GitHub
- Go to your repo → Settings → Secrets and variables → Actions
- Create new secret named `CURSEFORGE_TOKEN`
- Paste your API token

## 3. Choose Your Workflow Template
Pick the right workflow for your project:
- **WoW Addon**: Use `examples/release-bigwigs.yml`
- **Minecraft Mod**: Use `examples/release-minecraft.yml`
- **Other/Generic**: Use `examples/release-basic.yml`

## 4. Configure the Workflow
- Copy your chosen workflow to `.github/workflows/release.yml`
- Replace `YOUR_PROJECT_ID_HERE` with your CurseForge project ID
- Update game versions and other settings as needed

## 5. Create a Release
```bash
git tag v1.0.0
git push origin v1.0.0
```
Then create a GitHub release with that tag!

---

**That's it!** Your addon will automatically publish to CurseForge when you create releases.

For detailed instructions, see the [full README](README.md).

## Finding Your CurseForge Project ID

Your project ID is in your CurseForge project URL:
```
https://www.curseforge.com/wow/addons/your-addon-name
                                      ↑ This part or check project settings
```

Or find it in your project's "About" section on CurseForge.

## Common Issues

**Workflow doesn't run?**
- Make sure the file is in `.github/workflows/`
- Ensure it has a `.yml` extension
- Verify you created a release (not just a tag)

**Authentication error?**
- Double-check your secret is named correctly
- Verify the token hasn't expired
- Make sure you're using the token in the workflow correctly

**Files missing from upload?**
- Review the packaging commands in your workflow
- Check the zip file includes all necessary files
- Ensure you're excluding unnecessary files (.git, etc.)

For more troubleshooting, see the [full README](README.md#troubleshooting).
