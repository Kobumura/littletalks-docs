# 🔍 iOS Build Number Issue - Complete Context Summary

**Date**: August 6, 2025  
**Issue**: iOS builds failing with "build 100 already exists" despite store-first version management  
**Status**: Root cause identified, multiple solutions available

## 📋 The Problem

1. **What's happening**: 
   - iOS builds are trying to upload build 100, which already exists in TestFlight
   - The App Store Connect query IS working (returns 100 correctly)
   - But the version logic is using "fallback" mode instead of store-first
   - Android versioning works perfectly (currently at build 1880+)

2. **Current TestFlight Status**:
   - Version 2.0.0 - Build 100 ✅
   - Version 1.5.17 - Build 100 ✅
   - Version 1.5.16 - Build 100, Build 1 ✅
   - Version 1.5.15 - Build 1 ✅
   - (All iOS builds oddly use 100 or 1, while Android is at 1880+)

## 🔬 What We've Discovered

### 1. **JSON Parsing Issue - SOLVED**
- **Original Problem**: Multiline private key in JSON causing parse errors
- **Error**: `ERROR:unexpected token at '{'`
- **Solution Found**: Escape newlines in the private key
- **Test Results** (from private repo test):
  ```
  Test 1: JSON with escaped newlines - ✅ SUCCESS: Build number is 100
  Test 2: Separate .p8 file - ❌ ERROR: missing field(s): key
  Test 3: Direct API key in Fastfile - ✅ SUCCESS: Build number is 100
  ```

### 2. **Lane Name Conflict - IDENTIFIED**
- **Problem**: Lane named `get_build_number` conflicts with Fastlane action
- **Solution**: Rename to `query_build_number` or similar

### 3. **Version Logic Bug - IDENTIFIED**
- **Current behavior**:
  ```
  📋 iOS Version Management:
  Current Version: 1.5.15 (build 7)
  Version Strategy: patch
  Patch increment: 1.5.15 → 1.5.16
  Build number: 100 (fallback)  <-- PROBLEM: Using fallback instead of store value
  ```
- **Root Cause**: Even though App Store query succeeds, the condition check fails:
  ```bash
  if [ "$APP_STORE_SUCCESS" = "true" ] && [ "$LATEST_APP_STORE_BUILD" != "0" ]; then
    NEXT_BUILD=$((LATEST_APP_STORE_BUILD + 1))  # Should use this
  else
    FALLBACK_BUILD=100  # But uses this instead
  fi
  ```

### 4. **Security Concern - NOTED**
- Secrets are being printed to logs in the public repo
- Need to mask sensitive data in workflow outputs

## 🛠️ Solutions to Implement

### Fix 1: App Store Query Job (Escaped JSON)
```yaml
# In Setup App Store Connect API Key step:
# Escape the key for JSON - replace newlines with \n
ESCAPED_KEY=$(echo "$KEY" | awk '{printf "%s\\n", $0}' | sed 's/\\n$//')

# Create API key JSON with properly escaped key
cat > ios/fastlane/api_key.json << EOF
{
  "key_id": "$ASC_API_KEY_ID",
  "issuer_id": "$ASC_ISSUER_ID",
  "key": "$ESCAPED_KEY",
  "duration": 1200,
  "in_house": false
}
EOF
```

### Fix 2: Lane Name Change
```ruby
# Change from:
lane :get_build_number do
# To:
lane :query_build_number do
```

### Fix 3: Debug Version Logic
Need to check why `APP_STORE_SUCCESS` or `LATEST_APP_STORE_BUILD` variables aren't being passed correctly from the `app-store-query` job to the `ios-build` job.

## 📊 Current Architecture

- **Version Management**: Store-first approach queries App Store/Google Play
- **iOS Query**: `app-store-query` job runs on macOS, queries TestFlight
- **Android Query**: `google-play-query` job queries all tracks successfully
- **Problem**: iOS query succeeds but ios-build job doesn't use the result

## 🎯 Next Steps

1. **Immediate Fix**: 
   - Apply JSON escaping fix
   - Fix lane name conflict
   - Add debugging to see why store query result isn't being used

2. **Debug the job outputs**:
   - Check if `needs.app-store-query.outputs.latest_build_number` is properly set
   - Check if `needs.app-store-query.outputs.query_success` is "true"
   - Add echo statements to debug variable values

3. **Consider Manual Override**:
   - Temporarily set iOS build number to match Android (1900+) for consistency
   - Or increment from 100 → 101 manually

## 💡 Key Insights

1. **The App Store Connect API IS working** - we can query successfully
2. **The JSON parsing IS fixed** with the escape approach
3. **The issue is in the condition logic** that determines whether to use store value
4. **iOS and Android build numbers are completely out of sync** (100 vs 1880)

## 🔧 Test Workflow Created

Successfully created and ran test workflow in private repo:
- Location: `.github/workflows/test-app-store-query.yaml`
- Results: 2 of 3 methods work (escaped JSON ✅, direct API ✅)
- Confirms our API credentials are valid and working

## 📝 For Next Session

1. Apply the JSON escaping fix to `app-store-query` job
2. Debug why the store query success isn't propagating to build job
3. Consider syncing iOS build numbers with Android
4. Add proper secret masking to prevent leaks in public logs
5. Document the working solution in LP-21

## 🏷️ Related Tickets

- **LP-21**: iOS Build Version Conflicts (main ticket)
- **LP-23**: Manual Version Control (completed - major version feature)
- **LP-24**: Store-First Version Management (future - full version from store)