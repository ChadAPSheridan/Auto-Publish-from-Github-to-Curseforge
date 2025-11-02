# Choosing the Right Workflow

This guide helps you choose the best workflow configuration for your specific needs.

## Decision Tree

```
Start: What are you publishing?
    │
    ├─ WoW Addon
    │   │
    │   ├─ Simple addon (single .toc file, lua files)?
    │   │   └─→ Use BigWigsMods Packager (release-bigwigs.yml)
    │   │       ✓ Automatic .toc processing
    │   │       ✓ Version substitution
    │   │       ✓ Multi-platform support
    │   │
    │   └─ Complex addon (multiple libraries, custom build)?
    │       └─→ Use Basic Workflow (release-basic.yml)
    │           ✓ Full control over packaging
    │           ✓ Custom build steps
    │
    ├─ Minecraft Mod
    │   │
    │   ├─ Java Edition (Fabric/Forge)?
    │   │   └─→ Use Minecraft Workflow (release-minecraft.yml)
    │   │       ✓ Gradle build integration
    │   │       ✓ JAR packaging
    │   │       ✓ Multi-version support
    │   │
    │   └─ Bedrock Edition?
    │       └─→ Consider other platforms (not primarily CurseForge)
    │
    └─ Other Game/Platform
        └─→ Use Basic Workflow (release-basic.yml)
            ✓ Adapt to your needs
            ✓ Change game_endpoint
```

## Workflow Comparison

| Feature | Basic Workflow | BigWigsMods Packager | Minecraft Workflow |
|---------|---------------|---------------------|-------------------|
| **Best For** | Any addon/mod | WoW addons | Minecraft Java mods |
| **Complexity** | Medium | Low | Medium |
| **Flexibility** | High | Medium | Medium |
| **Build Steps** | Manual | Automatic | Gradle |
| **Version Substitution** | Manual | Automatic | Build system |
| **Multi-Platform** | Manual setup | Built-in | Manual setup |
| **Learning Curve** | Moderate | Easy | Moderate |
| **Customization** | Full control | Limited | Good control |

## Detailed Comparison

### Basic Workflow (release-basic.yml)

**Pros:**
- ✅ Works for any game/platform
- ✅ Full control over packaging
- ✅ Easy to understand and modify
- ✅ No dependencies on third-party actions
- ✅ Can handle complex file structures

**Cons:**
- ❌ More manual configuration required
- ❌ Need to handle version substitution yourself
- ❌ Must manually set up multi-platform publishing
- ❌ More verbose workflow file

**Use When:**
- Publishing to a game without specialized tools
- Need complete control over the build process
- Have complex or unusual packaging requirements
- Want to minimize external dependencies

**Example Use Cases:**
- Custom addons with complex build processes
- Multi-game addons
- Addons requiring pre-processing steps
- Projects with unconventional structures

### BigWigsMods Packager (release-bigwigs.yml)

**Pros:**
- ✅ Extremely simple setup (just a few lines)
- ✅ Automatic .toc file processing
- ✅ Automatic version substitution (@project-version@)
- ✅ Built-in multi-platform support (CurseForge, WoWInterface, Wago, GitHub)
- ✅ Industry standard for WoW addons
- ✅ Handles dependencies automatically

**Cons:**
- ❌ Only for WoW addons
- ❌ Less flexibility for custom packaging
- ❌ Requires specific file structure
- ❌ Depends on third-party action

**Use When:**
- Publishing WoW addons
- Following standard WoW addon structure
- Want the simplest possible setup
- Need multi-platform publishing

**Example Use Cases:**
- Standard WoW addons
- AddOns following community conventions
- Projects wanting minimal maintenance
- Addons published to multiple platforms

### Minecraft Workflow (release-minecraft.yml)

**Pros:**
- ✅ Integrates with Gradle build system
- ✅ Automatic JAR creation
- ✅ Supports version variables
- ✅ Can run tests before publishing
- ✅ Handles dependencies via Gradle

**Cons:**
- ❌ Only for Minecraft Java Edition
- ❌ Requires Gradle setup
- ❌ More complex than basic workflow
- ❌ Longer build times

**Use When:**
- Publishing Minecraft Fabric or Forge mods
- Already using Gradle for builds
- Need to run tests before publishing
- Want dependency management

**Example Use Cases:**
- Fabric mods
- Forge mods
- Mods with complex dependencies
- Projects requiring automated testing

## Special Considerations

### Publishing to Multiple Platforms

**If you need to publish to:**

1. **Only CurseForge**
   - Any workflow works fine
   - Basic workflow gives most control

2. **CurseForge + WoWInterface (WoW)**
   - Use BigWigsMods packager
   - Add both API tokens as secrets

3. **CurseForge + Modrinth (Minecraft)**
   - Use Minecraft workflow
   - Add upload step for Modrinth
   - Example:
   ```yaml
   - name: Upload to Modrinth
     uses: RubixDev/modrinth-upload@v1
   ```

4. **CurseForge + GitHub Releases**
   - Any workflow
   - Add upload step for GitHub:
   ```yaml
   - name: Upload to GitHub Release
     uses: softprops/action-gh-release@v1
   ```

### Version Management Strategies

**Automatic Version from Tag:**
- All workflows support this
- Use `${{ github.event.release.tag_name }}`
- Best for most projects

**Automatic Version from File:**
- BigWigsMods: Reads from .toc
- Minecraft: Uses build.gradle or gradle.properties
- Basic: Need to implement yourself

**Manual Version:**
- Can hardcode in workflow
- Not recommended
- Use only for special cases

### Build Complexity

**Simple (no build needed):**
- Basic workflow
- BigWigsMods packager
- Just package and upload

**Medium (some processing):**
- Basic workflow with custom steps
- Run scripts before packaging

**Complex (full build system):**
- Minecraft workflow (Gradle)
- Custom workflow with build tools
- May need Docker for complex dependencies

## Migration Guide

### From Manual Uploads to Automation

1. **Identify your current process:**
   - What files do you currently upload?
   - How do you create the package?
   - What metadata do you set?

2. **Choose the matching workflow:**
   - Match your current process to a workflow
   - Start with the closest example

3. **Test thoroughly:**
   - Use a test project first
   - Verify packaging is identical
   - Check metadata is correct

### From One Workflow to Another

**Basic → BigWigsMods:**
- Remove manual packaging steps
- Ensure .toc file is correct
- Add packager configuration
- Much simpler!

**Basic → Minecraft:**
- Set up Gradle build
- Configure build.gradle
- Remove manual JAR creation

**BigWigsMods → Basic:**
- Add manual packaging steps
- Handle version substitution manually
- More control but more work

## Quick Reference

### I want the simplest setup possible
→ **BigWigsMods Packager** (if WoW) or **Basic Workflow**

### I need maximum flexibility
→ **Basic Workflow**

### I'm publishing a Minecraft mod
→ **Minecraft Workflow**

### I need to publish to multiple platforms
→ **BigWigsMods Packager** (WoW) or **Basic Workflow** with custom steps

### I have complex build requirements
→ **Custom workflow** based on **Basic Workflow**

### I'm new to GitHub Actions
→ Start with **BigWigsMods Packager** (WoW) or **Basic Workflow**

### I want to minimize maintenance
→ **BigWigsMods Packager** (WoW) or **Minecraft Workflow**

## Still Not Sure?

1. Start with the **Basic Workflow** - it's the most flexible
2. Try your chosen workflow with a test project
3. Iterate and customize as needed
4. Check the [FAQ](FAQ.md) for specific questions
5. Ask for help by opening an issue

## Need a Custom Solution?

If none of these workflows fit your needs:

1. Start with the closest workflow
2. Review the [GitHub Actions documentation](https://docs.github.com/en/actions)
3. Check the [CurseForge Upload action docs](https://github.com/itsmeow/curseforge-upload)
4. Modify step-by-step to match your requirements
5. Share your custom workflow to help others!
