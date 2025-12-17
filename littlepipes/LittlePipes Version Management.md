# 🚀 LittlePipes Version Management

## Overview

LittlePipes now supports flexible version management strategies, allowing you to control how version numbers and build numbers are incremented. The system maintains the intelligent store-first approach by default while providing manual override options when needed.

## Version Strategy Options

### 1. **Auto (Default)**
- **Behavior**: Increments patch version automatically (e.g., 2.1.3 → 2.1.4)
- **Build Number**: Uses store-first approach (queries App Store/Google Play)
- **Use Case**: Regular releases, bug fixes, minor updates

### 2. **Major**
- **Behavior**: Increments major version (e.g., 2.1.3 → 3.0.0)
- **Build Number**: Auto-increments from store
- **Use Case**: Major feature releases, breaking changes, significant updates

### 3. **Minor**
- **Behavior**: Increments minor version (e.g., 2.1.3 → 2.2.0)
- **Build Number**: Auto-increments from store
- **Use Case**: New features, moderate changes, backwards-compatible updates

### 4. **Patch**
- **Behavior**: Increments patch version (e.g., 2.1.3 → 2.1.4)
- **Build Number**: Auto-increments from store
- **Use Case**: Bug fixes, small improvements (same as auto)

### 5. **Custom**
- **Behavior**: Uses the exact version you specify
- **Build Number**: Uses custom build number if provided, otherwise auto-increments
- **Use Case**: Special releases, version corrections, specific requirements

### 6. **Build Only**
- **Behavior**: Keeps current version, only increments build number
- **Build Number**: Auto-increments from store
- **Use Case**: Re-releases, build fixes, same version with new build

## Usage Examples

### Example 1: Major Version Release
```yaml
# Triggering a major version update (2.x.x → 3.0.0)
workflow_dispatch:
  inputs:
    platform: both
    version_strategy: major
```

### Example 2: Custom Version with Specific Build
```yaml
# Setting specific version 3.1.0 with build number 200
workflow_dispatch:
  inputs:
    platform: both
    version_strategy: custom
    custom_version: "3.1.0"
    custom_build_number: "200"
```

### Example 3: Build-Only Update
```yaml
# Keep version, just increment build number
workflow_dispatch:
  inputs:
    platform: ios
    version_strategy: build_only
```

## How It Works

### Version Number Management
1. **Current Version Detection**: Reads from project files (pbxproj for iOS, build.gradle for Android)
2. **Strategy Application**: Applies the selected increment strategy
3. **Custom Override**: Uses custom version if provided
4. **Consistency**: Ensures both platforms use the same version strategy

### Build Number Management
1. **Store Query**: Queries App Store Connect and Google Play Console
2. **Highest Build Detection**: Finds the highest build across all tracks
3. **Safety Buffer**: Adds buffer to prevent conflicts (Android)
4. **Custom Override**: Uses custom build number if provided
5. **Fallback**: Uses project file + increment if store query fails

## Platform-Specific Behavior

### iOS
- Version stored in: `MARKETING_VERSION`
- Build stored in: `CURRENT_PROJECT_VERSION`
- Store query: TestFlight via Fastlane
- Fallback: Project file or 100 (whichever is higher)

### Android
- Version stored in: `versionName`
- Build stored in: `versionCode`
- Store query: All tracks (internal, alpha, beta, production)
- Safety buffer: +10 to highest found build
- Fallback: Project file + 1

## Best Practices

### When to Use Each Strategy

1. **Use `auto` for**:
   - Regular development builds
   - Bug fixes
   - Minor improvements
   - Daily releases

2. **Use `major` for**:
   - Breaking API changes
   - Complete redesigns
   - Major feature sets
   - Marketing milestones

3. **Use `minor` for**:
   - New features
   - Significant improvements
   - Quarterly releases
   - Feature bundles

4. **Use `custom` for**:
   - Fixing version mistakes
   - Aligning with marketing requirements
   - Special version numbers
   - Platform synchronization

5. **Use `build_only` for**:
   - Re-submitting rejected builds
   - Build configuration changes
   - Certificate updates
   - Same version, new build

### Version Number Guidelines

- **Format**: Always use semantic versioning (X.Y.Z)
- **Consistency**: Keep iOS and Android versions in sync
- **Build Numbers**: Let the system manage these unless absolutely necessary
- **Custom Builds**: Only override build numbers for specific requirements

## Troubleshooting

### Common Issues

1. **"Version already exists" error**
   - Solution: Use `build_only` strategy to increment just the build number
   - Alternative: Use `custom` with a higher build number

2. **Version mismatch between platforms**
   - Solution: Use `custom` strategy to manually align versions
   - Prevention: Always build both platforms together

3. **Build number conflicts**
   - Solution: Let the store-first approach handle it
   - Alternative: Use custom build number with sufficient buffer

### Validation

The system validates:
- Custom version format (must be X.Y.Z)
- Custom build number (must be numeric)
- Store query results
- Version increment logic

## Integration with CI/CD

### GitHub Actions UI
When triggering manually through GitHub Actions:
1. Go to Actions tab
2. Select "Trigger LittlePipes Build"
3. Click "Run workflow"
4. Choose your version strategy from the dropdown
5. Fill in custom values if using "custom" strategy

### API Trigger
```bash
# Example: Trigger major version release via API
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/YOUR_ORG/YOUR_REPO/actions/workflows/trigger-build.yaml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "platform": "both",
      "version_strategy": "major"
    }
  }'
```

## Future Enhancements

- **Version Templates**: Pre-defined version patterns
- **Auto-Detection**: Detect version strategy from commit messages
- **Version History**: Track version decisions in database
- **Platform-Specific Versions**: Allow different versions per platform
- **Preview Mode**: Show what version will be before building

---

**Remember**: The store-first approach ensures you never have version conflicts. Manual overrides should be used sparingly and only when you have specific versioning requirements.