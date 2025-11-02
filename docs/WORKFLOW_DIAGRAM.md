# Workflow Diagram

This document provides a visual representation of the automated publishing process.

## Process Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     GitHub Repository                            │
│                                                                  │
│  1. Developer creates a new release                             │
│     ├─ Assigns version tag (e.g., v1.0.0)                      │
│     ├─ Writes release notes                                     │
│     └─ Publishes release                                        │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   GitHub Actions Triggered                       │
│                                                                  │
│  2. Workflow starts automatically                               │
│     ├─ Event: release.published                                 │
│     └─ Runner: ubuntu-latest                                    │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Step 1: Checkout Code                          │
│                                                                  │
│  3. Actions checkout repository                                 │
│     └─ Uses: actions/checkout@v4                                │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                Step 2: Build/Package Addon                       │
│                                                                  │
│  4. Build and package your addon                                │
│     ├─ WoW: Use BigWigsMods packager or custom zip              │
│     ├─ Minecraft: Run Gradle build to create JAR                │
│     └─ Other: Custom packaging script                           │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│               Step 3: Upload to CurseForge                       │
│                                                                  │
│  5. Upload packaged addon                                       │
│     ├─ Uses CurseForge API                                      │
│     ├─ Authenticates with CURSEFORGE_TOKEN secret               │
│     ├─ Uploads ZIP/JAR file                                     │
│     ├─ Sets game versions                                       │
│     ├─ Sets release type (alpha/beta/release)                   │
│     └─ Includes changelog from GitHub release notes             │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CurseForge Platform                           │
│                                                                  │
│  6. Addon is now live on CurseForge                             │
│     ├─ Version matches GitHub release tag                       │
│     ├─ Changelog displays GitHub release notes                  │
│     ├─ Users can download the new version                       │
│     └─ Automatic notifications sent to followers                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Component Interaction

```
┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│   GitHub    │         │    GitHub    │         │  CurseForge │
│  Repository │────────▶│   Actions    │────────▶│   Platform  │
│             │         │              │         │             │
│  - Code     │         │  - Build     │         │  - Host     │
│  - Releases │         │  - Package   │         │  - Serve    │
│  - Tags     │         │  - Upload    │         │  - Notify   │
└─────────────┘         └──────────────┘         └─────────────┘
      │                        ▲                        │
      │                        │                        │
      │                  ┌──────────┐                  │
      │                  │ Secrets  │                  │
      └─────────────────▶│  Store   │◀─────────────────┘
                         │          │
                         │  Token   │
                         └──────────┘
```

## Sequence Diagram

```
Developer        GitHub          GitHub Actions      CurseForge
    │                │                  │                 │
    │ Create Release │                  │                 │
    │───────────────▶│                  │                 │
    │                │                  │                 │
    │                │  Trigger Workflow│                 │
    │                │─────────────────▶│                 │
    │                │                  │                 │
    │                │                  │ Checkout Code   │
    │                │◀─────────────────│                 │
    │                │                  │                 │
    │                │                  │ Build Package   │
    │                │                  │────────┐        │
    │                │                  │        │        │
    │                │                  │◀───────┘        │
    │                │                  │                 │
    │                │                  │ Upload Package  │
    │                │                  │────────────────▶│
    │                │                  │                 │
    │                │                  │   Success       │
    │                │                  │◀────────────────│
    │                │                  │                 │
    │                │  Workflow Success│                 │
    │                │◀─────────────────│                 │
    │                │                  │                 │
    │   Notification │                  │                 │
    │◀───────────────│                  │                 │
    │                │                  │                 │
```

## Data Flow

### What Gets Shared Between Systems

**From GitHub to GitHub Actions:**
- Repository code
- Release tag (e.g., `v1.0.0`)
- Release title
- Release notes/changelog
- Trigger event information

**From GitHub Actions to CurseForge:**
- Packaged addon file (ZIP/JAR)
- Project ID
- Version number (from tag)
- Changelog (from release notes)
- Game versions
- Release type (alpha/beta/release)

**From GitHub Secrets to GitHub Actions:**
- CurseForge API token (secure)

## Authentication Flow

```
1. Developer generates API token on CurseForge
           │
           ▼
2. Developer stores token as GitHub Secret
           │
           ▼
3. GitHub Actions retrieves secret at runtime
           │
           ▼
4. GitHub Actions authenticates API call to CurseForge
           │
           ▼
5. CurseForge validates token and processes upload
```

## Error Handling

```
┌──────────────┐
│ Start Upload │
└──────┬───────┘
       │
       ▼
┌──────────────┐      Failed      ┌──────────────┐
│  Validate    │─────────────────▶│   Log Error  │
│  Inputs      │                  │   & Exit     │
└──────┬───────┘                  └──────────────┘
       │ Valid
       ▼
┌──────────────┐      Failed      ┌──────────────┐
│ Authenticate │─────────────────▶│   Log Error  │
│  with API    │                  │   & Exit     │
└──────┬───────┘                  └──────────────┘
       │ Success
       ▼
┌──────────────┐      Failed      ┌──────────────┐
│   Upload     │─────────────────▶│   Log Error  │
│   Package    │                  │   & Exit     │
└──────┬───────┘                  └──────────────┘
       │ Success
       ▼
┌──────────────┐
│   Success    │
└──────────────┘
```

## Key Points

1. **Automation**: Everything happens automatically after creating a GitHub release
2. **Security**: API tokens are stored securely in GitHub Secrets
3. **Transparency**: All steps are logged in GitHub Actions
4. **Reliability**: Failed uploads are logged and can be retried
5. **Version Sync**: GitHub tags and CurseForge versions stay in sync
