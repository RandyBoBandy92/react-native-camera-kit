# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Testing
- `yarn test` - Run Jest tests
- `yarn lint` - Run ESLint with project configuration
- `yarn bootstrap` - Set up example project (runs `yarn example && yarn && yarn pods`)
- `yarn example ios` - Run example app on iOS
- `yarn example android` - Run example app on Android

### Example Project Development
- `yarn pods` - Install iOS CocoaPods dependencies for example app
- Example app is located in `/example/` directory with its own package.json

## Architecture Overview

This is a React Native camera library that provides native camera functionality through a bridge architecture. The codebase is structured as:

### Platform-Specific Implementation Pattern
The library uses a consistent pattern where each major component has platform-specific implementations:
- `Component.ios.js` - iOS-specific React Native code
- `Component.android.js` - Android-specific React Native code
- Native iOS code in `ios/ReactNativeCameraKit/` (28 Objective-C files)
- Native Android code in `android/src/main/java/com/rncamerakit/` (27 Java files)

### Core Components
1. **CameraKitCamera** - Main camera component with capture, flash, torch, and focus controls
2. **CameraKitGalleryView** - Native gallery grid view (UICollectionView on iOS, RecyclerView on Android)
3. **CameraKitGallery** - Gallery API for accessing device photos and albums
4. **CameraKitCameraScreen** - Full-screen camera with QR/barcode scanning

### Native Bridge Architecture
- **iOS**: Uses PhotoKit framework for gallery access, AVFoundation for camera
  - Gallery photos sorted by `creationDate` ascending (oldest first) - see `CKGalleryManager.m:35`
  - Implements favorite photo marking functionality
- **Android**: Uses MediaStore for gallery access, Camera2 API for camera functionality
  - Gallery queries use MediaStore.Images.Media projections
- Bridge modules export methods to React Native layer via `NativeModules`

### Key Technical Details
- Gallery images are fetched with creation date sorting: `sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:YES]]`
- iOS gallery data includes favorite status via `asset.isFavorite` property
- Cross-platform image selection/deselection handled through native bridge
- Performance optimizations for high photo capture rates
- iCloud photo downloading support with progress indicators (iOS)

### File Organization
```
src/
├── index.js (main exports)
├── CameraKitCamera.{ios,android}.js
├── CameraKitGallery.{ios,android}.js  
├── CameraKitGalleryView.{ios,android}.js
└── CameraScreen/
    ├── CameraKitCameraScreen.{ios,android}.js
    └── CameraKitCameraScreenBase.js
```

### Development Notes
- When modifying gallery sorting behavior, update both iOS (`CKGalleryManager.m`) and Android (`NativeGalleryModule.java`) implementations
- Gallery component supports favorite photo indicators on iOS - new features should consider this existing capability
- Image capture performance is critical - changes should be tested on real devices
- The library maintains backward compatibility - breaking changes require version bumps

### Gallery Sorting Feature
The gallery component now supports dynamic sorting through `sortBy` and `sortOrder` props:

```jsx
<CameraKitGalleryView
  sortBy="creationDate"     // or "modificationDate" 
  sortOrder="ascending"     // or "descending"
  // ... other props
/>
```

**Implementation details:**
- **iOS**: Uses PHFetchOptions with dynamic NSSortDescriptor based on props
- **Android**: Uses MediaStore ORDER BY clause with DATE_ADDED or DATE_MODIFIED
- **Defaults**: `sortBy="creationDate"`, `sortOrder="ascending"` (maintains backward compatibility)
- **Behavior**: Changes trigger gallery refresh automatically via setter methods