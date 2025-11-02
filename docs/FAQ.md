# Frequently Asked Questions (FAQ)

## General Questions

### Q: Do I need to pay for GitHub Actions?
**A:** GitHub Actions is free for public repositories. For private repositories, you get 2,000 free minutes per month, which is more than enough for typical addon publishing workflows.

### Q: Can I publish to multiple platforms (CurseForge, Modrinth, WoWInterface)?
**A:** Yes! You can add multiple upload steps to your workflow to publish to different platforms simultaneously. See the [examples](../examples/) for ideas on how to structure this.

### Q: Will this work with private repositories?
**A:** Yes, GitHub Actions works with both public and private repositories. However, be aware of the free minutes limit for private repositories.

### Q: How long does the automation take to run?
**A:** Typically 1-3 minutes from creating a release to having your addon live on CurseForge, depending on the size of your addon and build complexity.

## Setup Questions

### Q: Where do I find my CurseForge project ID?
**A:** Your project ID can be found in:
1. Your project URL on CurseForge
2. Your project settings under "About Project"
3. The API console at https://console.curseforge.com/

### Q: Can I use the same API token for multiple projects?
**A:** Yes, one CurseForge API token can be used for all your projects. However, you may want to create separate tokens for different purposes for better security and tracking.

### Q: Do I need to create a new workflow file for each addon?
**A:** Yes, each repository should have its own workflow file. However, you can copy the same template and just change the project ID and addon-specific settings.

### Q: What if I don't use GitHub releases?
**A:** The workflows provided are triggered by GitHub releases. If you prefer to use tags or direct commits, you'll need to modify the workflow trigger. See the [GitHub Actions documentation](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) for other trigger options.

## Workflow Questions

### Q: Can I test the workflow without publishing to CurseForge?
**A:** Yes! You can:
1. Use the `workflow_dispatch` trigger to manually run workflows
2. Add a `dry_run` parameter to skip the actual upload
3. Test with a separate test project on CurseForge

### Q: How do I publish alpha or beta versions?
**A:** Set the `release_type` parameter in your workflow:
```yaml
release_type: alpha  # or 'beta' or 'release'
```

You can also automatically determine this based on your tag name (e.g., tags containing "alpha" → alpha release).

### Q: Can I customize the packaging process?
**A:** Absolutely! The packaging step in the workflow can be customized to:
- Include/exclude specific files
- Run build scripts
- Process files before packaging
- Generate documentation

### Q: What if my addon has dependencies?
**A:** Include installation and building of dependencies in your workflow:
```yaml
- name: Install dependencies
  run: npm install  # or pip install, etc.

- name: Build
  run: npm run build
```

## Version Management

### Q: How do I handle version numbers?
**A:** Best practices:
1. Use semantic versioning (e.g., v1.2.3)
2. Use the GitHub release tag as the version number
3. Use variable substitution in your addon files (e.g., `@project-version@` for WoW addons)

### Q: Can I publish the same version to multiple game versions?
**A:** Yes, specify multiple game versions in the workflow:
```yaml
game_versions: 1.20.1,1.20.2,1.19.4
```

### Q: What happens if I create a release with an existing version number?
**A:** CurseForge typically won't allow duplicate version numbers. The workflow will fail with an error. Make sure to use unique version tags for each release.

## Security Questions

### Q: Is it safe to store my API token in GitHub Secrets?
**A:** Yes, GitHub Secrets are encrypted and only exposed to workflows in your repository. They are not visible in logs or to anyone browsing your repository.

### Q: What permissions does the API token need?
**A:** The CurseForge API token needs permission to upload files to your projects. Generate it from the CurseForge console with the appropriate scopes.

### Q: Can someone steal my token from the workflow logs?
**A:** No, GitHub automatically redacts secrets from workflow logs. If you accidentally print a secret, it will appear as `***` in the logs.

### Q: Should I commit the workflow file to my repository?
**A:** Yes, workflow files should be committed. They don't contain secrets (those are in GitHub Secrets), and being in version control helps track changes to your automation.

## Troubleshooting

### Q: The workflow runs but nothing uploads to CurseForge. What's wrong?
**A:** Common causes:
1. Incorrect project ID
2. Invalid or expired API token
3. Malformed package file
4. Network issues

Check the workflow logs in the Actions tab for specific error messages.

### Q: My packaged file is empty or missing files. Why?
**A:** Review your packaging commands:
1. Ensure you're copying the right files
2. Check your exclusion patterns (e.g., `-x "*.git*"`)
3. Verify the directory structure in your zip file
4. Test the packaging locally first

### Q: The workflow fails with "Resource not accessible by integration". What does this mean?
**A:** This typically means:
1. You haven't set up the required secrets
2. The secret name in the workflow doesn't match the actual secret name
3. Permissions issue with your GitHub token (if you're using repository operations)

### Q: How do I debug workflow failures?
**A:** 
1. Go to the Actions tab in your repository
2. Click on the failed workflow run
3. Expand the failed step to see detailed logs
4. Look for error messages and stack traces
5. Add debug output to your workflow if needed:
```yaml
- name: Debug info
  run: |
    echo "Tag: ${{ github.event.release.tag_name }}"
    ls -la
```

## Platform-Specific Questions

### Q: Does this work for WoW Classic?
**A:** Yes! Just specify the appropriate game versions for Classic (e.g., 1.14.3, 1.14.4).

### Q: Can I use this for Minecraft Bedrock Edition?
**A:** CurseForge primarily supports Java Edition. For Bedrock, consider publishing to other platforms.

### Q: What about publishing to WoWInterface?
**A:** The BigWigsMods packager supports WoWInterface. Set your WoWInterface API token as a secret and configure it in the packager.

### Q: Does this work for other games on CurseForge?
**A:** Yes! As long as the game is supported by CurseForge, you can publish addons/mods for it using similar workflows. Just change the `game_endpoint` parameter.

## Advanced Usage

### Q: Can I run additional checks before uploading?
**A:** Yes! Add steps before the upload:
```yaml
- name: Run tests
  run: npm test

- name: Lint code
  run: npm run lint

- name: Upload to CurseForge
  # Only runs if previous steps succeed
```

### Q: How do I publish different builds for different game versions?
**A:** You can:
1. Create separate jobs for each game version
2. Use matrix builds to test multiple versions
3. Build conditionally based on the tag or branch

### Q: Can I schedule releases automatically?
**A:** Yes, use the `schedule` trigger:
```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
```

However, you'll still need to manage version numbers appropriately.

### Q: How do I rollback a release?
**A:** 
1. Delete the GitHub release (this won't remove it from CurseForge)
2. Manually mark the CurseForge version as alpha or delete it
3. Create a new release with a fixed version

## Getting Help

### Q: Where can I get more help?
**A:** 
- Check the [main README](../README.md) for detailed instructions
- Review the [example workflows](../examples/)
- Look at the [workflow diagrams](WORKFLOW_DIAGRAM.md)
- Visit the [CurseForge Discord](https://discord.gg/curseforge)
- Check [GitHub Community Forum](https://github.community/)
- Open an issue in this repository

### Q: I found a bug in the workflow. How do I report it?
**A:** Open an issue in this repository with:
1. Description of the problem
2. Your workflow file (with sensitive info removed)
3. Error messages from the workflow logs
4. Steps to reproduce

### Q: Can I contribute improvements to this guide?
**A:** Absolutely! Pull requests are welcome. See the main README for contribution guidelines.
