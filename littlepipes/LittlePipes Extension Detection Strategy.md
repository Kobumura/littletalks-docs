# 🎯 LittlePipes Extension Detection Strategy

## 📋 **Problem Statement**
Generic iOS builds need to automatically detect and handle:
- Main app bundle ID
- Any number of extensions (0-10+)
- Proper provisioning profile mapping for exportOptions.plist
- Match certificate setup for all bundle IDs

## 🔍 **Detection Methods (Priority Order)**

### **Method 1: Project File Analysis (Most Reliable)**
```bash
# Parse .xcodeproj/project.pbxproj for all PRODUCT_BUNDLE_IDENTIFIER entries
find_all_bundle_ids() {
  grep -o 'PRODUCT_BUNDLE_IDENTIFIER = [^;]*' *.xcodeproj/project.pbxproj | \
  sed 's/PRODUCT_BUNDLE_IDENTIFIER = //;s/;//;s/"//g' | \
  sort -u
}
```

**Advantages:**
- ✅ 100% accurate - reflects actual Xcode configuration
- ✅ Catches custom bundle ID patterns
- ✅ Works with any extension type (OneSignal, Share, Widget, etc.)

### **Method 2: Info.plist Discovery**
```bash
# Find all Info.plist files and extract CFBundleIdentifier
find_bundle_ids_from_plists() {
  find . -name "Info.plist" -exec plutil -p {} \; | \
  grep CFBundleIdentifier | \
  sed 's/.*=> "//' | sed 's/".*//' | \
  sort -u
}
```

### **Method 3: Match Repository Analysis** 
```bash
# Query existing Match profiles to see what's already configured
query_existing_match_profiles() {
  fastlane match --git_url "$MATCH_GIT_URL" --readonly true --list
}
```

## 🏗️ **Implementation Architecture**

### **Phase 1: Repository Analysis Script**
```yaml
- name: Detect iOS Bundle IDs and Extensions
  id: bundle_detection
  working-directory: ./private-repo/ios
  run: |
    echo "🔍 Detecting iOS bundle IDs and extensions..."
    
    # Method 1: Parse project file (most reliable)
    BUNDLE_IDS=$(grep -o 'PRODUCT_BUNDLE_IDENTIFIER = [^;]*' *.xcodeproj/project.pbxproj | \
                sed 's/PRODUCT_BUNDLE_IDENTIFIER = //;s/;//;s/"//g' | \
                sort -u | tr '\n' ',' | sed 's/,$//')
    
    # Separate main app from extensions
    MAIN_BUNDLE_ID=$(echo "$BUNDLE_IDS" | tr ',' '\n' | grep -v '\.' | head -1)
    EXTENSION_IDS=$(echo "$BUNDLE_IDS" | tr ',' '\n' | grep '\.' | tr '\n' ',' | sed 's/,$//')
    
    echo "📱 Detected Bundle IDs:"
    echo "  Main App: $MAIN_BUNDLE_ID"
    echo "  Extensions: $EXTENSION_IDS"
    echo "  Total Count: $(echo "$BUNDLE_IDS" | tr ',' '\n' | wc -l)"
    
    # Output for later steps
    echo "main_bundle_id=$MAIN_BUNDLE_ID" >> $GITHUB_OUTPUT
    echo "extension_ids=$EXTENSION_IDS" >> $GITHUB_OUTPUT
    echo "all_bundle_ids=$BUNDLE_IDS" >> $GITHUB_OUTPUT
```

### **Phase 2: Dynamic Match Setup**
```yaml
- name: Setup Match Certificates (Dynamic)
  working-directory: ./private-repo/ios
  run: |
    echo "🔑 Setting up Match certificates for all detected bundle IDs..."
    
    # Convert comma-separated list to array
    IFS=',' read -ra BUNDLE_ARRAY <<< "${{ steps.bundle_detection.outputs.all_bundle_ids }}"
    
    # Setup Match for each bundle ID
    for BUNDLE_ID in "${BUNDLE_ARRAY[@]}"; do
      echo "🔐 Setting up Match for: $BUNDLE_ID"
      fastlane match appstore \
        --git_url "$MATCH_GIT_URL" \
        --app_identifier "$BUNDLE_ID" \
        --team_id "${{ needs.setup.outputs.apple_team_id }}" \
        --readonly true
    done
```

### **Phase 3: Dynamic Export Options**
```yaml
- name: Create Dynamic exportOptions.plist
  working-directory: ./private-repo
  run: |
    echo "📝 Creating exportOptions.plist with all detected bundle IDs..."
    
    # Start the plist file
    cat > ios/exportOptions.plist << 'PLIST_START'
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>method</key>
        <string>app-store</string>
        <key>teamID</key>
        <string>${{ needs.setup.outputs.apple_team_id }}</string>
        <key>uploadBitcode</key>
        <false/>
        <key>uploadSymbols</key>
        <true/>
        <key>compileBitcode</key>
        <false/>
        <key>manageAppVersionAndBuildNumber</key>
        <false/>
        <key>signingStyle</key>
        <string>manual</string>
        <key>provisioningProfiles</key>
        <dict>
    PLIST_START
    
    # Add each bundle ID dynamically
    IFS=',' read -ra BUNDLE_ARRAY <<< "${{ steps.bundle_detection.outputs.all_bundle_ids }}"
    for BUNDLE_ID in "${BUNDLE_ARRAY[@]}"; do
      echo "            <key>$BUNDLE_ID</key>" >> ios/exportOptions.plist
      echo "            <string>match AppStore $BUNDLE_ID</string>" >> ios/exportOptions.plist
    done
    
    # Close the plist file
    cat >> ios/exportOptions.plist << 'PLIST_END'
        </dict>
    </dict>
    </plist>
    PLIST_END
    
    echo "✅ Generated exportOptions.plist:"
    cat ios/exportOptions.plist
```

## 🎛️ **User Configuration Options**

### **Option 1: Automatic Detection (Default)**
```yaml
# In LittlePipes UI/Config
ios_bundle_detection: "automatic"  # Default
ios_bundle_ids: []  # Empty = auto-detect
```

### **Option 2: Manual Override**
```yaml
# User-specified bundle IDs
ios_bundle_detection: "manual"
ios_bundle_ids:
  - "com.company.app"
  - "com.company.app.ShareExtension"
  - "com.company.app.WidgetExtension"
  - "com.company.app.NotificationServiceExtension"
```

### **Option 3: Hybrid Approach**
```yaml
# Auto-detect but allow user additions
ios_bundle_detection: "hybrid"
ios_additional_bundle_ids:
  - "com.company.app.CustomExtension"  # User adds extras
```

## 🛡️ **Error Handling & Validation**

### **Detection Validation**
```bash
validate_bundle_detection() {
  # Ensure we found at least one bundle ID
  if [ -z "$BUNDLE_IDS" ]; then
    echo "❌ No bundle IDs detected. Check project structure."
    exit 1
  fi
  
  # Validate bundle ID format
  for ID in $BUNDLE_IDS; do
    if ! echo "$ID" | grep -qE '^[a-zA-Z0-9.-]+\.[a-zA-Z0-9.-]+$'; then
      echo "⚠️ Invalid bundle ID format: $ID"
    fi
  done
  
  # Check for common patterns
  MAIN_COUNT=$(echo "$BUNDLE_IDS" | tr ',' '\n' | grep -v '\.' | wc -l)
  if [ "$MAIN_COUNT" -ne 1 ]; then
    echo "⚠️ Expected 1 main bundle ID, found $MAIN_COUNT"
  fi
}
```

### **Match Verification**
```bash
verify_match_profiles() {
  echo "🔍 Verifying Match profiles are available..."
  
  IFS=',' read -ra BUNDLE_ARRAY <<< "$ALL_BUNDLE_IDS"
  for BUNDLE_ID in "${BUNDLE_ARRAY[@]}"; do
    if ! fastlane match --list | grep -q "$BUNDLE_ID"; then
      echo "❌ Missing Match profile for: $BUNDLE_ID"
      echo "💡 Run: fastlane match appstore --app_identifier $BUNDLE_ID"
      exit 1
    fi
  done
}
```

## 📊 **LittlePipes Platform Integration**

### **Database Schema (LP-15 Future)**
```sql
-- Extension detection results stored per project
CREATE TABLE project_ios_bundles (
  project_id UUID,
  bundle_id VARCHAR(255),
  bundle_type ENUM('main', 'extension'),
  detection_method ENUM('automatic', 'manual'),
  last_detected_at TIMESTAMP,
  is_active BOOLEAN DEFAULT true
);
```

### **API Endpoints**
```javascript
// Auto-detect bundle IDs from repository
POST /api/projects/{id}/ios/detect-bundles
{
  "detection_method": "automatic",
  "validate_match": true
}

// Manual bundle ID configuration
PUT /api/projects/{id}/ios/bundle-config
{
  "bundle_ids": ["com.app.main", "com.app.extension"],
  "detection_method": "manual"
}
```

### **UI Integration**
```jsx
// LittlePipes Console - iOS Configuration
<IosConfigSection>
  <BundleDetection 
    mode={project.ios_bundle_detection || 'automatic'}
    detectedBundles={project.detected_bundles}
    manualBundles={project.manual_bundles}
    onDetectionModeChange={updateDetectionMode}
  />
</IosConfigSection>
```

## 🚀 **Implementation Phases**

### **Phase 1: Current (LP-21)**
- ✅ Fix OneSignal hardcoded extension
- 🔄 Implement automatic detection script
- ✅ Test with LittleTalks multi-extension setup

### **Phase 2: Platform Abstraction (LP-22)**
- 📝 Create reusable bundle detection shell script
- 🔧 Extract to `scripts/ios/detect-bundles.sh`
- 🧪 Test across multiple React Native projects

### **Phase 3: User Configuration (LP-23)**
- 🎛️ Add manual override options
- 💾 Implement bundle ID persistence
- 🔍 Add validation and error handling

### **Phase 4: SaaS Integration (LP-24)**
- 🌐 Database-driven bundle management
- 📊 UI for bundle configuration
- 📈 Analytics on extension usage patterns

## 🎯 **Success Metrics**

- **Accuracy**: 99%+ bundle ID detection rate
- **Coverage**: Support 0-20 extensions per project
- **User Experience**: Zero manual configuration required for 95% of projects
- **Performance**: Detection completes in <30 seconds
- **Reliability**: Handles edge cases (custom schemes, complex hierarchies)

---

*This strategy positions LittlePipes as the most intelligent iOS CI/CD platform, automatically handling complex extension configurations that would require hours of manual setup in traditional solutions.*