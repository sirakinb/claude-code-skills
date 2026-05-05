---
name: visual-test
description: Visual component testing for web, mobile, and desktop apps. Discovers all components/pages, spins up a browser or simulator, tests each one visually and functionally, and reports bugs, issues, and suggested fixes. Use when asked to test the app, test components, run visual tests, check the UI, find bugs, or QA the product.
---

# Visual Component Test Runner

Automatically discover, render, and test every component/page in your app. Works for web apps (Playwright + browser), React Native / Expo (iOS/Android simulator), and Electron/Tauri desktop apps.

**CRITICAL WORKFLOW - Follow these steps in exact order:**

## Step 1: Detect Project Type

Scan the project root to determine what kind of app this is:

```bash
# Check for project type indicators
PROJECT_DIR="$(pwd)"

# Web frameworks
[ -f "$PROJECT_DIR/next.config.js" ] || [ -f "$PROJECT_DIR/next.config.ts" ] || [ -f "$PROJECT_DIR/next.config.mjs" ] && echo "TYPE: nextjs"
[ -f "$PROJECT_DIR/vite.config.ts" ] || [ -f "$PROJECT_DIR/vite.config.js" ] && echo "TYPE: vite"
[ -f "$PROJECT_DIR/nuxt.config.ts" ] && echo "TYPE: nuxt"
[ -f "$PROJECT_DIR/svelte.config.js" ] && echo "TYPE: sveltekit"
[ -f "$PROJECT_DIR/angular.json" ] && echo "TYPE: angular"
[ -f "$PROJECT_DIR/remix.config.js" ] || [ -f "$PROJECT_DIR/remix.config.ts" ] && echo "TYPE: remix"

# Mobile - Cross-platform
[ -f "$PROJECT_DIR/app.json" ] && grep -q "expo" "$PROJECT_DIR/app.json" 2>/dev/null && echo "TYPE: expo"
[ -f "$PROJECT_DIR/react-native.config.js" ] && echo "TYPE: react-native"
grep -q "\"flutter\"" "$PROJECT_DIR/pubspec.yaml" 2>/dev/null && echo "TYPE: flutter"

# Mobile - Native iOS (Swift/SwiftUI/Obj-C via Xcode)
find "$PROJECT_DIR" -maxdepth 2 -name "*.xcworkspace" -o -name "*.xcodeproj" 2>/dev/null | head -1 | grep -q . && echo "TYPE: ios-native"
[ -f "$PROJECT_DIR/ios/Podfile" ] && echo "TYPE: ios-native (with CocoaPods)"
find "$PROJECT_DIR" -maxdepth 3 -name "*.swift" 2>/dev/null | head -1 | grep -q . && echo "LANG: swift"
find "$PROJECT_DIR" -maxdepth 3 -name "*.storyboard" -o -name "*.xib" 2>/dev/null | head -1 | grep -q . && echo "UI: storyboard/xib"
find "$PROJECT_DIR" -maxdepth 3 -name "*.swift" -exec grep -l "SwiftUI" {} \; 2>/dev/null | head -1 | grep -q . && echo "UI: swiftui"

# Mobile - Native Android (Kotlin/Java via Android Studio)
[ -f "$PROJECT_DIR/build.gradle" ] || [ -f "$PROJECT_DIR/build.gradle.kts" ] && echo "TYPE: android-native"
[ -f "$PROJECT_DIR/android/build.gradle" ] || [ -f "$PROJECT_DIR/android/build.gradle.kts" ] && echo "TYPE: android-native (subdir)"
[ -f "$PROJECT_DIR/settings.gradle" ] || [ -f "$PROJECT_DIR/settings.gradle.kts" ] && echo "BUILD: gradle"
find "$PROJECT_DIR" -maxdepth 3 -name "*.kt" 2>/dev/null | head -1 | grep -q . && echo "LANG: kotlin"
find "$PROJECT_DIR" -maxdepth 4 -name "AndroidManifest.xml" 2>/dev/null | head -1 | grep -q . && echo "HAS: AndroidManifest"

# Desktop
grep -q "\"electron\"" "$PROJECT_DIR/package.json" 2>/dev/null && echo "TYPE: electron"
[ -f "$PROJECT_DIR/src-tauri/tauri.conf.json" ] && echo "TYPE: tauri"
```

## Step 2: Discover All Components & Pages

Based on project type, find every testable component and page.

### Web Apps (Next.js, Vite/React, Nuxt, SvelteKit, Remix, Angular)

```bash
# Next.js App Router pages
find "$PROJECT_DIR/app" -name "page.tsx" -o -name "page.jsx" -o -name "page.js" 2>/dev/null

# Next.js Pages Router
find "$PROJECT_DIR/pages" -name "*.tsx" -o -name "*.jsx" 2>/dev/null | grep -v "_app\|_document\|_error\|api/"

# Vite/React pages & components
find "$PROJECT_DIR/src" -name "*.tsx" -o -name "*.jsx" 2>/dev/null | grep -iE "(page|view|screen|component)" | head -50

# General: find all route/page files
find "$PROJECT_DIR/src" -path "*/pages/*" -o -path "*/views/*" -o -path "*/screens/*" -o -path "*/routes/*" 2>/dev/null | grep -E "\.(tsx|jsx|vue|svelte)$"

# Find component directories
find "$PROJECT_DIR/src/components" -name "*.tsx" -o -name "*.jsx" -o -name "*.vue" -o -name "*.svelte" 2>/dev/null | head -80
```

### Mobile Apps (React Native / Expo)

```bash
# Expo Router screens
find "$PROJECT_DIR/app" -name "*.tsx" -o -name "*.jsx" 2>/dev/null | grep -v "_layout\|+not-found"

# React Navigation screens
find "$PROJECT_DIR/src" -path "*/screens/*" -name "*.tsx" -o -path "*/screens/*" -name "*.jsx" 2>/dev/null

# Components
find "$PROJECT_DIR/src/components" -name "*.tsx" -o -name "*.jsx" 2>/dev/null | head -80
```

### Native iOS (Swift/SwiftUI)

```bash
# Find all SwiftUI views
find "$PROJECT_DIR" -name "*.swift" -exec grep -l "struct.*View.*:.*View\|NavigationView\|NavigationStack\|TabView" {} \; 2>/dev/null | head -50

# Find all UIKit view controllers
find "$PROJECT_DIR" -name "*.swift" -exec grep -l "UIViewController\|UITableViewController\|UICollectionViewController" {} \; 2>/dev/null | head -50

# Find storyboard scenes
find "$PROJECT_DIR" -name "*.storyboard" 2>/dev/null

# Find XIB files
find "$PROJECT_DIR" -name "*.xib" 2>/dev/null

# Find navigation/tab structure (SwiftUI)
find "$PROJECT_DIR" -name "*.swift" -exec grep -l "TabView\|NavigationLink\|NavigationDestination\|sheet\|fullScreenCover" {} \; 2>/dev/null | head -30

# Find the app entry point
find "$PROJECT_DIR" -name "*.swift" -exec grep -l "@main\|@UIApplicationMain" {} \; 2>/dev/null
```

### Native Android (Kotlin/Java)

```bash
# Find all Activities
find "$PROJECT_DIR" -name "*.kt" -o -name "*.java" 2>/dev/null | xargs grep -l "Activity()\|AppCompatActivity\|ComponentActivity" 2>/dev/null | head -30

# Find all Fragments
find "$PROJECT_DIR" -name "*.kt" -o -name "*.java" 2>/dev/null | xargs grep -l "Fragment()\|DialogFragment" 2>/dev/null | head -30

# Find Jetpack Compose screens
find "$PROJECT_DIR" -name "*.kt" 2>/dev/null | xargs grep -l "@Composable" 2>/dev/null | head -50

# Find XML layouts
find "$PROJECT_DIR" -path "*/res/layout*" -name "*.xml" 2>/dev/null | head -50

# Find navigation graph
find "$PROJECT_DIR" -path "*/res/navigation*" -name "*.xml" 2>/dev/null

# Find AndroidManifest for declared activities
find "$PROJECT_DIR" -name "AndroidManifest.xml" 2>/dev/null | head -3
```

### Flutter

```bash
find "$PROJECT_DIR/lib" -name "*.dart" 2>/dev/null | grep -iE "(screen|page|view|widget)" | head -80
```

### Desktop (Electron / Tauri)

```bash
# Electron renderer components (same as web)
find "$PROJECT_DIR/src" -name "*.tsx" -o -name "*.jsx" -o -name "*.vue" -o -name "*.svelte" 2>/dev/null | head -80
```

**After discovery, read each file to understand:**
- What the component renders (UI elements, text, images, forms)
- What props/params it expects
- What routes map to it
- Any loading, error, or empty states

## Step 3: Start the App & Test Environment

### Web Apps

```bash
# Detect running dev servers first
SKILL_DIR=~/.claude/skills/playwright-skill
cd $SKILL_DIR && node -e "require('./lib/helpers').detectDevServers().then(s => console.log(JSON.stringify(s)))"
```

If no server is running, start one:
```bash
# Try common dev commands in background
cd "$PROJECT_DIR" && npm run dev &
# or: yarn dev, pnpm dev, bun dev
# Wait for server to be ready
sleep 5
```

Then use Playwright (via the playwright-skill) to test:
```bash
cd ~/.claude/skills/playwright-skill && node run.js /tmp/visual-test-runner.js
```

### iOS Apps (Native Swift/SwiftUI, React Native, Expo, Flutter)

**CRITICAL: Always use Xcode Simulator for iOS testing.**

#### Step 3a: Check Xcode & Simulator Availability

```bash
# Verify Xcode is installed
xcode-select -p 2>/dev/null && echo "Xcode installed" || echo "ERROR: Xcode not installed"

# List all available simulator runtimes
xcrun simctl list runtimes

# List all available iOS simulator devices
xcrun simctl list devices available | grep -A 20 "iOS"
```

#### Step 3b: Boot the iOS Simulator

```bash
# Check if a simulator is already booted
BOOTED=$(xcrun simctl list devices | grep "Booted" | head -1)
if [ -n "$BOOTED" ]; then
  echo "Simulator already running: $BOOTED"
  DEVICE_ID=$(echo "$BOOTED" | grep -oE '[A-F0-9-]{36}')
else
  # Find the best available device (prefer latest iPhone)
  DEVICE_ID=$(xcrun simctl list devices available | grep "iPhone" | tail -1 | grep -oE '[A-F0-9-]{36}')

  if [ -z "$DEVICE_ID" ]; then
    # Create a device if none exist
    RUNTIME=$(xcrun simctl list runtimes | grep "iOS" | tail -1 | awk '{print $NF}')
    DEVICE_ID=$(xcrun simctl create "Test iPhone" "iPhone 16" "$RUNTIME")
    echo "Created simulator: $DEVICE_ID"
  fi

  # Boot it
  xcrun simctl boot "$DEVICE_ID"
  echo "Booted simulator: $DEVICE_ID"
fi

# Open the Simulator app so the window is visible
open -a Simulator

# Wait for simulator to fully boot
xcrun simctl bootstatus "$DEVICE_ID" -b
echo "Simulator ready"
```

#### Step 3c: Build & Install the App

**Native iOS (Swift/SwiftUI/Obj-C):**
```bash
# Find the Xcode project or workspace
WORKSPACE=$(find "$PROJECT_DIR" -maxdepth 2 -name "*.xcworkspace" | head -1)
XCODEPROJ=$(find "$PROJECT_DIR" -maxdepth 2 -name "*.xcodeproj" | head -1)

if [ -n "$WORKSPACE" ]; then
  # Get the scheme name
  SCHEME=$(xcodebuild -workspace "$WORKSPACE" -list 2>/dev/null | awk '/Schemes:/{found=1; next} found && NF{print $1; exit}')
  
  # Build for simulator
  xcodebuild -workspace "$WORKSPACE" \
    -scheme "$SCHEME" \
    -sdk iphonesimulator \
    -destination "id=$DEVICE_ID" \
    -derivedDataPath /tmp/visual-test-build \
    build 2>&1 | tail -5

  # Find and install the .app
  APP_PATH=$(find /tmp/visual-test-build -name "*.app" -path "*/Debug-iphonesimulator/*" | head -1)
  xcrun simctl install "$DEVICE_ID" "$APP_PATH"
  
  # Get bundle ID and launch
  BUNDLE_ID=$(/usr/libexec/PlistBuddy -c "Print CFBundleIdentifier" "$APP_PATH/Info.plist")
  xcrun simctl launch "$DEVICE_ID" "$BUNDLE_ID"
elif [ -n "$XCODEPROJ" ]; then
  SCHEME=$(xcodebuild -project "$XCODEPROJ" -list 2>/dev/null | awk '/Schemes:/{found=1; next} found && NF{print $1; exit}')
  
  xcodebuild -project "$XCODEPROJ" \
    -scheme "$SCHEME" \
    -sdk iphonesimulator \
    -destination "id=$DEVICE_ID" \
    -derivedDataPath /tmp/visual-test-build \
    build 2>&1 | tail -5

  APP_PATH=$(find /tmp/visual-test-build -name "*.app" -path "*/Debug-iphonesimulator/*" | head -1)
  xcrun simctl install "$DEVICE_ID" "$APP_PATH"
  BUNDLE_ID=$(/usr/libexec/PlistBuddy -c "Print CFBundleIdentifier" "$APP_PATH/Info.plist")
  xcrun simctl launch "$DEVICE_ID" "$BUNDLE_ID"
fi
```

**Expo:**
```bash
cd "$PROJECT_DIR"
# Install dependencies if needed
[ -d "node_modules" ] || npm install

# Prebuild iOS native project if needed
[ -d "ios" ] || npx expo prebuild --platform ios

# Run on simulator (this builds, installs, and launches)
npx expo run:ios --device "$DEVICE_ID" &
EXPO_PID=$!

# Wait for app to launch (watch for the build to complete)
sleep 30
echo "Expo app launched on simulator"
```

**React Native (bare workflow):**
```bash
cd "$PROJECT_DIR"
[ -d "node_modules" ] || npm install
cd ios && pod install && cd ..

# Build and run on simulator
npx react-native run-ios --simulator "$(xcrun simctl list devices booted | grep "iPhone" | head -1 | sed 's/ (.*//' | xargs)" &
RN_PID=$!
sleep 30
echo "React Native app launched on simulator"
```

**Flutter (iOS):**
```bash
cd "$PROJECT_DIR"
flutter pub get
flutter run -d "$DEVICE_ID" &
FLUTTER_PID=$!
sleep 30
echo "Flutter app launched on simulator"
```

#### Step 3d: iOS Simulator Interaction & Screenshots

```bash
RESULTS_DIR="/tmp/visual-test-results"
mkdir -p "$RESULTS_DIR"

# Take screenshot of current screen
xcrun simctl io booted screenshot "$RESULTS_DIR/ios-screen-current.png"

# Get device screen size for tap coordinates
# iPhone 16: 393x852 points, iPhone 16 Pro Max: 440x956 points
# Adjust coordinates based on actual device

# Tap at coordinates (x, y in points)
xcrun simctl io booted tap 200 400

# Swipe (fromX, fromY, toX, toY)
xcrun simctl io booted swipe 200 600 200 200  # Swipe up/scroll down

# Type text into focused field
xcrun simctl io booted type "test@example.com"

# Press hardware buttons
xcrun simctl io booted keypress home
xcrun simctl io booted keypress volumeUp

# Open a deep link / URL scheme
xcrun simctl openurl booted "myapp://settings"

# Send push notification (for testing notification UI)
xcrun simctl push booted "$BUNDLE_ID" /tmp/test-notification.json

# Record video of interaction
xcrun simctl io booted recordVideo "$RESULTS_DIR/ios-test-recording.mp4" &
RECORD_PID=$!
# ... perform interactions ...
kill $RECORD_PID 2>/dev/null

# Change device appearance (dark/light mode)
xcrun simctl ui booted appearance dark
sleep 1
xcrun simctl io booted screenshot "$RESULTS_DIR/ios-dark-mode.png"
xcrun simctl ui booted appearance light
sleep 1
xcrun simctl io booted screenshot "$RESULTS_DIR/ios-light-mode.png"

# Set status bar overrides (clean screenshots)
xcrun simctl status_bar booted override \
  --time "9:41" \
  --batteryState charged \
  --batteryLevel 100 \
  --wifiBars 3 \
  --cellularBars 4

# Test different content size categories (accessibility text sizes)
# This requires changing simulator settings via UI or defaults command

# Simulate location
xcrun simctl location booted set 37.7749 -122.4194  # San Francisco

# Clear app data and relaunch for fresh state testing
xcrun simctl terminate booted "$BUNDLE_ID"
xcrun simctl privacy booted reset all "$BUNDLE_ID"
xcrun simctl launch booted "$BUNDLE_ID"
```

---

### Android Apps (Native Kotlin/Java, React Native, Expo, Flutter)

**CRITICAL: Always use Android Studio Emulator for Android testing.**

#### Step 3a: Check Android SDK & Emulator Availability

```bash
# Verify Android SDK
[ -d "$ANDROID_HOME" ] || [ -d "$ANDROID_SDK_ROOT" ] || [ -d "$HOME/Library/Android/sdk" ] && echo "Android SDK found"

# Set ANDROID_HOME if not set
export ANDROID_HOME="${ANDROID_HOME:-${ANDROID_SDK_ROOT:-$HOME/Library/Android/sdk}}"
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"

# List available AVDs (Android Virtual Devices)
emulator -list-avds 2>/dev/null

# List available system images
sdkmanager --list 2>/dev/null | grep "system-images" | grep "google_apis" | tail -10
```

#### Step 3b: Create & Boot Android Emulator

```bash
export ANDROID_HOME="${ANDROID_HOME:-${ANDROID_SDK_ROOT:-$HOME/Library/Android/sdk}}"
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/cmdline-tools/latest/bin:$PATH"

# Check if an emulator is already running
RUNNING=$(adb devices 2>/dev/null | grep "emulator" | head -1 | awk '{print $1}')
if [ -n "$RUNNING" ]; then
  echo "Emulator already running: $RUNNING"
  EMULATOR_SERIAL="$RUNNING"
else
  # List existing AVDs
  AVDS=$(emulator -list-avds 2>/dev/null)
  
  if [ -z "$AVDS" ]; then
    # No AVDs exist - create one
    echo "No AVDs found. Creating one..."
    
    # Download system image if needed
    yes | sdkmanager "system-images;android-35;google_apis;arm64-v8a" 2>/dev/null
    
    # Create AVD
    echo "no" | avdmanager create avd \
      -n "visual_test_device" \
      -k "system-images;android-35;google_apis;arm64-v8a" \
      -d "pixel_8" \
      --force
    
    AVD_NAME="visual_test_device"
  else
    # Use first available AVD
    AVD_NAME=$(echo "$AVDS" | head -1)
  fi
  
  echo "Booting emulator: $AVD_NAME"
  
  # Launch emulator (not headless - we want to see it)
  emulator -avd "$AVD_NAME" -no-snapshot-load -gpu auto &
  EMULATOR_PID=$!
  
  # Wait for emulator to fully boot
  adb wait-for-device
  while [ "$(adb shell getprop sys.boot_completed 2>/dev/null)" != "1" ]; do
    sleep 2
  done
  echo "Emulator fully booted"
  
  EMULATOR_SERIAL=$(adb devices | grep "emulator" | head -1 | awk '{print $1}')
fi
```

#### Step 3c: Build & Install the App (Android)

**Native Android (Kotlin/Java):**
```bash
cd "$PROJECT_DIR"

# Build debug APK
./gradlew assembleDebug 2>&1 | tail -5

# Find the APK
APK_PATH=$(find "$PROJECT_DIR" -name "*.apk" -path "*/debug/*" | head -1)

if [ -z "$APK_PATH" ]; then
  APK_PATH=$(find "$PROJECT_DIR" -name "*-debug.apk" | head -1)
fi

# Install on emulator
adb -s "$EMULATOR_SERIAL" install -r "$APK_PATH"

# Get package name from manifest
PACKAGE_NAME=$(aapt dump badging "$APK_PATH" 2>/dev/null | grep "package:" | sed "s/.*name='//" | sed "s/'.*//")
LAUNCH_ACTIVITY=$(aapt dump badging "$APK_PATH" 2>/dev/null | grep "launchable-activity" | sed "s/.*name='//" | sed "s/'.*//")

# Launch the app
adb -s "$EMULATOR_SERIAL" shell am start -n "$PACKAGE_NAME/$LAUNCH_ACTIVITY"
echo "App launched: $PACKAGE_NAME"
```

**Expo (Android):**
```bash
cd "$PROJECT_DIR"
[ -d "node_modules" ] || npm install

# Prebuild Android native project if needed  
[ -d "android" ] || npx expo prebuild --platform android

# Run on emulator
npx expo run:android &
EXPO_PID=$!
sleep 40
echo "Expo app launched on Android emulator"
```

**React Native (bare workflow - Android):**
```bash
cd "$PROJECT_DIR"
[ -d "node_modules" ] || npm install

npx react-native run-android &
RN_PID=$!
sleep 40
echo "React Native app launched on Android emulator"
```

**Flutter (Android):**
```bash
cd "$PROJECT_DIR"
flutter pub get

# Get emulator device ID
FLUTTER_DEVICE=$(flutter devices | grep "emulator" | awk '{print $2}' | tr -d '(' | head -1)
flutter run -d "$FLUTTER_DEVICE" &
FLUTTER_PID=$!
sleep 40
echo "Flutter app launched on Android emulator"
```

#### Step 3d: Android Emulator Interaction & Screenshots

```bash
RESULTS_DIR="/tmp/visual-test-results"
mkdir -p "$RESULTS_DIR"

# Take screenshot
adb -s "$EMULATOR_SERIAL" exec-out screencap -p > "$RESULTS_DIR/android-screen-current.png"

# Get screen resolution for tap coordinates
SCREEN_SIZE=$(adb -s "$EMULATOR_SERIAL" shell wm size | awk '{print $NF}')
echo "Screen size: $SCREEN_SIZE"  # e.g., 1080x2400

# Tap at coordinates (in pixels)
adb -s "$EMULATOR_SERIAL" shell input tap 540 1200

# Swipe (fromX, fromY, toX, toY, durationMs)
adb -s "$EMULATOR_SERIAL" shell input swipe 540 1800 540 600 300  # Scroll down

# Type text
adb -s "$EMULATOR_SERIAL" shell input text "test@example.com"

# Press keys
adb -s "$EMULATOR_SERIAL" shell input keyevent KEYCODE_ENTER
adb -s "$EMULATOR_SERIAL" shell input keyevent KEYCODE_BACK
adb -s "$EMULATOR_SERIAL" shell input keyevent KEYCODE_HOME

# Open deep link
adb -s "$EMULATOR_SERIAL" shell am start -a android.intent.action.VIEW -d "myapp://settings"

# Record screen
adb -s "$EMULATOR_SERIAL" shell screenrecord /sdcard/test-recording.mp4 &
RECORD_PID=$!
# ... perform interactions ...
kill $RECORD_PID 2>/dev/null
adb -s "$EMULATOR_SERIAL" pull /sdcard/test-recording.mp4 "$RESULTS_DIR/android-test-recording.mp4"

# Toggle dark/light mode
adb -s "$EMULATOR_SERIAL" shell cmd uimode night yes
sleep 1
adb -s "$EMULATOR_SERIAL" exec-out screencap -p > "$RESULTS_DIR/android-dark-mode.png"
adb -s "$EMULATOR_SERIAL" shell cmd uimode night no
sleep 1
adb -s "$EMULATOR_SERIAL" exec-out screencap -p > "$RESULTS_DIR/android-light-mode.png"

# Change font scale (accessibility)
adb -s "$EMULATOR_SERIAL" shell settings put system font_scale 1.5
sleep 2
adb -s "$EMULATOR_SERIAL" exec-out screencap -p > "$RESULTS_DIR/android-large-text.png"
adb -s "$EMULATOR_SERIAL" shell settings put system font_scale 1.0

# Simulate location
adb -s "$EMULATOR_SERIAL" emu geo fix -122.4194 37.7749

# Clear app data and relaunch
adb -s "$EMULATOR_SERIAL" shell pm clear "$PACKAGE_NAME"
adb -s "$EMULATOR_SERIAL" shell am start -n "$PACKAGE_NAME/$LAUNCH_ACTIVITY"

# Dump UI hierarchy for inspection (accessibility IDs, element tree)
adb -s "$EMULATOR_SERIAL" shell uiautomator dump /sdcard/ui-dump.xml
adb -s "$EMULATOR_SERIAL" pull /sdcard/ui-dump.xml "$RESULTS_DIR/android-ui-hierarchy.xml"

# Get logcat errors (app crashes, exceptions)
adb -s "$EMULATOR_SERIAL" logcat -d -s "AndroidRuntime:E" "*:F" > "$RESULTS_DIR/android-crash-log.txt"
adb -s "$EMULATOR_SERIAL" logcat -d | grep -i "exception\|error\|fatal\|crash" | tail -30 > "$RESULTS_DIR/android-errors.txt"
```

---

### Cross-Platform Mobile Testing (iOS + Android Together)

When the project supports both platforms, test both:

```bash
# Boot both simulators in parallel
xcrun simctl boot "$DEVICE_ID" &
emulator -avd "$AVD_NAME" -no-snapshot-load -gpu auto &

# Wait for both to be ready
xcrun simctl bootstatus "$DEVICE_ID" -b &
adb wait-for-device && while [ "$(adb shell getprop sys.boot_completed 2>/dev/null)" != "1" ]; do sleep 2; done &
wait

echo "Both simulators ready"

# Build and install on both
# ... (use appropriate commands per platform)

# Screenshot both
xcrun simctl io booted screenshot "$RESULTS_DIR/ios-home.png"
adb exec-out screencap -p > "$RESULTS_DIR/android-home.png"
```

---

### Flutter (Both Platforms)

```bash
cd "$PROJECT_DIR"
flutter pub get

# Run on iOS
flutter run -d "$DEVICE_ID" &
sleep 30
xcrun simctl io booted screenshot "$RESULTS_DIR/flutter-ios-home.png"

# Run on Android (in parallel or after)
FLUTTER_ANDROID=$(flutter devices | grep "emulator" | awk '{print $2}' | tr -d '(' | head -1)
flutter run -d "$FLUTTER_ANDROID" &
sleep 30
adb exec-out screencap -p > "$RESULTS_DIR/flutter-android-home.png"
```

### Desktop (Electron)

```bash
cd "$PROJECT_DIR" && npm run dev &
# Electron apps render in a browser window - use Playwright to connect
```

## Step 4: Write & Execute Test Scripts

### Web App Test Script Template

Write this to `/tmp/visual-test-runner.js`:

```javascript
const { chromium } = require('playwright');
const fs = require('fs');
const path = require('path');

const TARGET_URL = 'http://localhost:3000'; // Auto-detected
const RESULTS_DIR = '/tmp/visual-test-results';

// Components/pages discovered in Step 2
const PAGES = [
  // { path: '/', name: 'Home' },
  // { path: '/about', name: 'About' },
  // Populated from discovery
];

const VIEWPORTS = [
  { name: 'desktop', width: 1440, height: 900 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'mobile', width: 375, height: 812 },
];

(async () => {
  // Create results directory
  if (!fs.existsSync(RESULTS_DIR)) fs.mkdirSync(RESULTS_DIR, { recursive: true });

  const browser = await chromium.launch({ headless: false, slowMo: 50 });
  const results = [];

  for (const pageInfo of PAGES) {
    console.log(`\n--- Testing: ${pageInfo.name} (${pageInfo.path}) ---`);
    const pageResults = { name: pageInfo.name, path: pageInfo.path, issues: [], screenshots: [] };

    for (const viewport of VIEWPORTS) {
      const context = await browser.newContext({
        viewport: { width: viewport.width, height: viewport.height },
      });
      const page = await context.newPage();

      // Collect console errors
      const consoleErrors = [];
      page.on('console', msg => {
        if (msg.type() === 'error') consoleErrors.push(msg.text());
      });

      // Collect failed network requests
      const failedRequests = [];
      page.on('requestfailed', req => {
        failedRequests.push({ url: req.url(), error: req.failure()?.errorText });
      });

      try {
        const response = await page.goto(`${TARGET_URL}${pageInfo.path}`, {
          waitUntil: 'networkidle',
          timeout: 15000,
        });

        // Check HTTP status
        if (!response || response.status() >= 400) {
          pageResults.issues.push({
            type: 'HTTP_ERROR',
            viewport: viewport.name,
            detail: `Status ${response?.status() || 'no response'}`,
          });
        }

        // Wait for content to render
        await page.waitForTimeout(1000);

        // Screenshot
        const screenshotPath = `${RESULTS_DIR}/${pageInfo.name.replace(/\s+/g, '-').toLowerCase()}-${viewport.name}.png`;
        await page.screenshot({ path: screenshotPath, fullPage: true });
        pageResults.screenshots.push(screenshotPath);

        // --- Visual & Functional Checks ---

        // 1. Check for visible error boundaries / crash screens
        const errorBoundary = await page.locator('text=/something went wrong|error|crash|unexpected/i').count();
        if (errorBoundary > 0) {
          pageResults.issues.push({
            type: 'ERROR_BOUNDARY',
            viewport: viewport.name,
            detail: 'Error boundary or crash message visible on page',
          });
        }

        // 2. Check for overlapping or clipped text (overflow detection)
        const overflowIssues = await page.evaluate(() => {
          const issues = [];
          document.querySelectorAll('*').forEach(el => {
            const style = window.getComputedStyle(el);
            if (el.scrollWidth > el.clientWidth && style.overflow === 'visible' && el.textContent?.trim()) {
              const rect = el.getBoundingClientRect();
              if (rect.width > 0 && rect.height > 0 && rect.width < 2000) {
                issues.push({
                  tag: el.tagName,
                  class: el.className?.toString().slice(0, 60),
                  text: el.textContent.trim().slice(0, 40),
                  scrollW: el.scrollWidth,
                  clientW: el.clientWidth,
                });
              }
            }
          });
          return issues.slice(0, 10);
        });
        if (overflowIssues.length > 0) {
          pageResults.issues.push({
            type: 'TEXT_OVERFLOW',
            viewport: viewport.name,
            detail: overflowIssues,
          });
        }

        // 3. Check for broken images
        const brokenImages = await page.evaluate(() => {
          return Array.from(document.querySelectorAll('img'))
            .filter(img => !img.complete || img.naturalWidth === 0)
            .map(img => ({ src: img.src, alt: img.alt }));
        });
        if (brokenImages.length > 0) {
          pageResults.issues.push({
            type: 'BROKEN_IMAGES',
            viewport: viewport.name,
            detail: brokenImages,
          });
        }

        // 4. Check for empty containers that should have content
        const emptyContainers = await page.evaluate(() => {
          return Array.from(document.querySelectorAll('main, section, article, [role="main"], .content, .container'))
            .filter(el => el.textContent?.trim().length === 0 && el.children.length === 0)
            .map(el => ({ tag: el.tagName, id: el.id, class: el.className?.toString().slice(0, 60) }));
        });
        if (emptyContainers.length > 0) {
          pageResults.issues.push({
            type: 'EMPTY_CONTAINER',
            viewport: viewport.name,
            detail: emptyContainers,
          });
        }

        // 5. Check for accessibility issues
        const a11yIssues = await page.evaluate(() => {
          const issues = [];
          // Images without alt
          document.querySelectorAll('img:not([alt])').forEach(img => {
            issues.push({ type: 'img-no-alt', src: img.src?.slice(0, 80) });
          });
          // Buttons without accessible name
          document.querySelectorAll('button').forEach(btn => {
            if (!btn.textContent?.trim() && !btn.getAttribute('aria-label') && !btn.getAttribute('title')) {
              issues.push({ type: 'button-no-label', html: btn.outerHTML.slice(0, 80) });
            }
          });
          // Links without text
          document.querySelectorAll('a').forEach(a => {
            if (!a.textContent?.trim() && !a.getAttribute('aria-label')) {
              issues.push({ type: 'link-no-text', href: a.href });
            }
          });
          // Missing form labels
          document.querySelectorAll('input:not([type="hidden"]):not([type="submit"]):not([type="button"]), textarea, select').forEach(input => {
            const id = input.id;
            const hasLabel = id && document.querySelector(`label[for="${id}"]`);
            const hasAriaLabel = input.getAttribute('aria-label') || input.getAttribute('aria-labelledby');
            const wrappedInLabel = input.closest('label');
            if (!hasLabel && !hasAriaLabel && !wrappedInLabel) {
              issues.push({ type: 'input-no-label', name: input.name, id: input.id });
            }
          });
          return issues.slice(0, 20);
        });
        if (a11yIssues.length > 0) {
          pageResults.issues.push({
            type: 'ACCESSIBILITY',
            viewport: viewport.name,
            detail: a11yIssues,
          });
        }

        // 6. Check color contrast (basic check for very light text on white)
        const contrastIssues = await page.evaluate(() => {
          const issues = [];
          document.querySelectorAll('p, span, h1, h2, h3, h4, h5, h6, a, button, label, li, td, th').forEach(el => {
            const style = window.getComputedStyle(el);
            const color = style.color;
            const bg = style.backgroundColor;
            // Flag near-white text on near-white background
            const parseRGB = c => c.match(/\d+/g)?.map(Number) || [];
            const fg = parseRGB(color);
            const bgc = parseRGB(bg);
            if (fg.length >= 3 && bgc.length >= 3) {
              const fgLight = fg[0] > 220 && fg[1] > 220 && fg[2] > 220;
              const bgLight = bgc[0] > 220 && bgc[1] > 220 && bgc[2] > 220;
              if (fgLight && bgLight && el.textContent?.trim()) {
                issues.push({
                  text: el.textContent.trim().slice(0, 30),
                  fg: color,
                  bg: bg,
                });
              }
            }
          });
          return issues.slice(0, 10);
        });
        if (contrastIssues.length > 0) {
          pageResults.issues.push({
            type: 'LOW_CONTRAST',
            viewport: viewport.name,
            detail: contrastIssues,
          });
        }

        // 7. Check interactive elements are clickable (not covered by other elements)
        const clickabilityIssues = await page.evaluate(() => {
          const issues = [];
          document.querySelectorAll('button, a, input, select, textarea, [role="button"]').forEach(el => {
            const rect = el.getBoundingClientRect();
            if (rect.width === 0 || rect.height === 0) return;
            const cx = rect.left + rect.width / 2;
            const cy = rect.top + rect.height / 2;
            const topEl = document.elementFromPoint(cx, cy);
            if (topEl && topEl !== el && !el.contains(topEl) && !topEl.closest('a, button, [role="button"]')) {
              issues.push({
                tag: el.tagName,
                text: el.textContent?.trim().slice(0, 30),
                coveredBy: topEl.tagName + (topEl.className ? '.' + topEl.className.toString().split(' ')[0] : ''),
              });
            }
          });
          return issues.slice(0, 10);
        });
        if (clickabilityIssues.length > 0) {
          pageResults.issues.push({
            type: 'BLOCKED_INTERACTIVE',
            viewport: viewport.name,
            detail: clickabilityIssues,
          });
        }

        // 8. Check forms submit correctly (dry run - just verify structure)
        const formIssues = await page.evaluate(() => {
          const issues = [];
          document.querySelectorAll('form').forEach((form, i) => {
            const hasSubmit = form.querySelector('button[type="submit"], input[type="submit"]');
            const hasAction = form.action && form.action !== window.location.href;
            if (!hasSubmit) {
              issues.push({ form: i, issue: 'No submit button found' });
            }
          });
          return issues;
        });
        if (formIssues.length > 0) {
          pageResults.issues.push({
            type: 'FORM_STRUCTURE',
            viewport: viewport.name,
            detail: formIssues,
          });
        }

        // 9. Console errors
        if (consoleErrors.length > 0) {
          pageResults.issues.push({
            type: 'CONSOLE_ERRORS',
            viewport: viewport.name,
            detail: consoleErrors.slice(0, 10),
          });
        }

        // 10. Failed network requests
        if (failedRequests.length > 0) {
          pageResults.issues.push({
            type: 'FAILED_REQUESTS',
            viewport: viewport.name,
            detail: failedRequests.slice(0, 10),
          });
        }

      } catch (err) {
        pageResults.issues.push({
          type: 'PAGE_LOAD_FAILURE',
          viewport: viewport.name,
          detail: err.message,
        });
      }

      await context.close();
    }

    results.push(pageResults);
  }

  // Write results summary
  const summary = JSON.stringify(results, null, 2);
  fs.writeFileSync(`${RESULTS_DIR}/test-results.json`, summary);
  console.log(`\n\n========== TEST RESULTS ==========`);
  console.log(summary);
  console.log(`\nScreenshots and results saved to ${RESULTS_DIR}`);

  await browser.close();
})();
```

### Mobile Test Script Templates

For mobile apps, use a combination of direct simulator commands and test frameworks.

#### Option A: Direct Simulator Commands (No Extra Dependencies)

Write a shell script to `/tmp/visual-test-mobile.sh`:

```bash
#!/bin/bash
# /tmp/visual-test-mobile.sh
# Automated mobile UI test via simulator commands

RESULTS_DIR="/tmp/visual-test-results"
PLATFORM="${1:-ios}"  # "ios" or "android"
mkdir -p "$RESULTS_DIR"

screenshot() {
  local name="$1"
  if [ "$PLATFORM" = "ios" ]; then
    xcrun simctl io booted screenshot "$RESULTS_DIR/ios-$name.png"
  else
    adb exec-out screencap -p > "$RESULTS_DIR/android-$name.png"
  fi
  echo "Screenshot: $name"
  sleep 1
}

tap() {
  local x="$1" y="$2"
  if [ "$PLATFORM" = "ios" ]; then
    xcrun simctl io booted tap "$x" "$y"
  else
    adb shell input tap "$x" "$y"
  fi
  sleep 2
}

swipe_up() {
  if [ "$PLATFORM" = "ios" ]; then
    xcrun simctl io booted swipe 200 600 200 200
  else
    adb shell input swipe 540 1800 540 600 300
  fi
  sleep 1
}

type_text() {
  local text="$1"
  if [ "$PLATFORM" = "ios" ]; then
    xcrun simctl io booted type "$text"
  else
    adb shell input text "$text"
  fi
  sleep 1
}

go_back() {
  if [ "$PLATFORM" = "ios" ]; then
    # Swipe from left edge to go back (iOS gesture)
    xcrun simctl io booted swipe 10 400 300 400
  else
    adb shell input keyevent KEYCODE_BACK
  fi
  sleep 1
}

echo "=== Starting Mobile Visual Tests ($PLATFORM) ==="

# Test 1: Home screen
screenshot "01-home"

# Test 2: Scroll content
swipe_up
screenshot "02-home-scrolled"

# Test 3: Dark mode
if [ "$PLATFORM" = "ios" ]; then
  xcrun simctl ui booted appearance dark
else
  adb shell cmd uimode night yes
fi
sleep 1
screenshot "03-dark-mode"

# Restore light mode
if [ "$PLATFORM" = "ios" ]; then
  xcrun simctl ui booted appearance light
else
  adb shell cmd uimode night no
fi
sleep 1

# Test 4: Large text / accessibility
if [ "$PLATFORM" = "android" ]; then
  adb shell settings put system font_scale 1.5
  sleep 2
  screenshot "04-large-text"
  adb shell settings put system font_scale 1.0
fi

# Test 5: Navigate through tabs/screens
# (Customize tap coordinates based on discovered UI layout)
# tap 100 800   # Tab 1
# screenshot "05-tab1"
# tap 200 800   # Tab 2
# screenshot "06-tab2"

echo "=== Tests complete. Results in $RESULTS_DIR ==="
```

Run it:
```bash
chmod +x /tmp/visual-test-mobile.sh
/tmp/visual-test-mobile.sh ios     # or: /tmp/visual-test-mobile.sh android
```

#### Option B: Maestro (Recommended for Thorough Testing)

Maestro provides accessibility-ID-based testing and works on both iOS and Android.

**Install Maestro (if not installed):**
```bash
# macOS
curl -Ls "https://get.maestro.mobile.dev" | bash
export PATH="$PATH:$HOME/.maestro/bin"
maestro --version
```

**Generate a Maestro flow from discovered screens.** Write to `/tmp/visual-test-flow.yaml`:

```yaml
# /tmp/visual-test-flow.yaml
appId: com.yourapp.bundleid  # From app.json, build.gradle, or Info.plist
---
# Screen 1: Home / Landing
- launchApp
- assertVisible: ".*"  # App renders something
- takeScreenshot: /tmp/visual-test-results/maestro-01-home

# Scroll through home content
- scroll:
    direction: DOWN
- takeScreenshot: /tmp/visual-test-results/maestro-02-home-scrolled

# Screen 2: Navigate to second tab/screen
# (adjust selector based on discovered navigation)
- tapOn: "Settings"
- takeScreenshot: /tmp/visual-test-results/maestro-03-settings

# Screen 3: Test a form
- tapOn: "Profile"
- takeScreenshot: /tmp/visual-test-results/maestro-04-profile
- tapOn:
    id: "email-input"  # Use accessibility ID
- inputText: "test@example.com"
- takeScreenshot: /tmp/visual-test-results/maestro-05-form-filled

# Test error state
- tapOn:
    id: "submit-button"
- takeScreenshot: /tmp/visual-test-results/maestro-06-after-submit

# Go back and test other screens
- pressKey: back
- takeScreenshot: /tmp/visual-test-results/maestro-07-back
```

**Run the flow:**
```bash
mkdir -p /tmp/visual-test-results
maestro test /tmp/visual-test-flow.yaml 2>&1 | tee /tmp/visual-test-results/maestro-output.txt
```

**Maestro tips for building flows from discovered components:**
- Read each screen/component file to find `testID`, `accessibilityLabel`, or `nativeID` props
- Use `tapOn: { id: "testID-value" }` for reliable element targeting
- Use `assertVisible: "Text on screen"` to verify correct screen loaded
- Use `assertNotVisible: "Error"` to verify no error states
- Add `- scroll: { direction: DOWN }` between sections to test full page content

#### Option C: Detox (React Native Only - Full Integration Tests)

```bash
# Install Detox CLI if needed
npm install -g detox-cli

# In project directory
cd "$PROJECT_DIR"
npx detox build --configuration ios.sim.debug
npx detox test --configuration ios.sim.debug --take-screenshots all \
  --artifacts-location /tmp/visual-test-results/detox/
```

#### Analyzing Mobile Screenshots

After capturing screenshots, **always use the Read tool to visually inspect each screenshot**:

```
Read /tmp/visual-test-results/ios-01-home.png
Read /tmp/visual-test-results/android-01-home.png
```

Check for:
- Elements cut off or overlapping
- Text too small or unreadable
- Buttons/touch targets too small (minimum 44x44pt iOS, 48x48dp Android)
- Content hidden behind notch, status bar, or home indicator
- Dark mode colors: sufficient contrast, no invisible text
- Large text mode: layout doesn't break, text isn't clipped
- Empty states: screens without data show appropriate placeholders
- Loading states: spinners or skeletons render correctly

For Android, also parse the UI hierarchy dump:
```bash
# Read the XML dump for accessibility analysis
cat /tmp/visual-test-results/android-ui-hierarchy.xml
# Look for: missing content-desc, tiny clickable areas, deep nesting
```

For iOS crash/error detection:
```bash
# Check simulator system log for crashes
xcrun simctl spawn booted log show --predicate 'eventMessage contains "crash" OR eventMessage contains "fatal"' --last 5m 2>/dev/null | tail -20
```

## Step 5: Analyze Results & Report

After running tests, analyze all screenshots and the results JSON. Report findings organized as:

### Report Format

```
## Visual Test Report

### Summary
- Pages/screens tested: X
- Total issues found: X
- Critical: X | Warning: X | Info: X

### Critical Issues (must fix)
- [Page Name] HTTP 500 error on /api/data endpoint
- [Page Name] Error boundary triggered on mobile viewport
- [Page Name] Form submit button blocked by overlay

### Warnings (should fix)
- [Page Name] 3 images missing alt text
- [Page Name] Text overflow on mobile (375px) in header
- [Page Name] Low contrast text in footer

### Suggestions (nice to have)
- [Page Name] Consider adding loading states for async content
- [Page Name] Empty container visible when no data present

### Screenshots
All screenshots saved to /tmp/visual-test-results/
- home-desktop.png, home-tablet.png, home-mobile.png
- about-desktop.png, about-tablet.png, about-mobile.png
...

### Recommended Code Changes
1. File: src/components/Header.tsx, Line 42
   Issue: Text overflows container on mobile
   Fix: Add `overflow-hidden text-ellipsis` or reduce font size

2. File: src/app/page.tsx, Line 15
   Issue: Missing error handling for data fetch
   Fix: Add error boundary or try-catch with fallback UI
```

## Step 6: Fix Issues (if requested)

After presenting the report, if the user asks to fix the issues:
1. Fix each issue one at a time
2. Re-run the test for that specific page/component to verify
3. Move to the next issue

## Tips

- **Always discover components first** - Don't assume what pages exist; scan the project
- **Test all viewports** - Desktop, tablet, and mobile are the minimum for web apps
- **Read screenshots** - Use the Read tool on saved PNG screenshots to visually inspect them
- **Check console/logcat errors** - These often reveal issues not visible in the UI
- **Test interactive elements** - Forms, buttons, links, modals, dropdowns
- **Test loading states** - Throttle network to see loading/skeleton states
- **Test error states** - Try invalid inputs, missing data scenarios
- **Test dark mode** - Use `xcrun simctl ui booted appearance dark` (iOS) or `adb shell cmd uimode night yes` (Android)
- **Test accessibility text sizes** - Android: `adb shell settings put system font_scale 1.5`
- **For iOS native** - Use `xcodebuild` to build, `xcrun simctl` to install/launch/screenshot/interact
- **For Android native** - Use `./gradlew assembleDebug` to build, `adb` to install/launch/screenshot/interact
- **Prefer Maestro for mobile** - It works on both iOS and Android with accessibility-ID-based selectors
- **Check both platforms** - If the app supports iOS and Android, test both simulators
- **Keep scripts in /tmp** - Never write test files to the project directory
- **Report actionable findings** - Every issue should include the file, line, and suggested fix
- **Clean up simulators** - After testing, offer to shut down simulators with `xcrun simctl shutdown all` or closing the emulator
