# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mute is an iOS library that detects if the silent/mute switch is enabled/disabled on a device. It works by playing a short sound and measuring the playback duration to determine the mute state.

## Architecture

**Core Library**: Single Swift file (`Mute/Classes/Mute.swift`) implementing the mute detection logic using AudioToolbox framework. The library uses a singleton pattern (`Mute.shared`) and plays a silent audio file (`mute.aiff`) to detect mute state.

**Resource Management**: Supports both CocoaPods and Swift Package Manager with proper bundle resource loading for the audio file.

**Example App**: Located in `Example/` directory, demonstrates library usage with a simple UI showing mute state.

## Development Commands

### Library Development (Swift Package Manager)
```bash
swift build                    # Build library
swift build -c release         # Build release version
swift package clean            # Clean build artifacts
swift package generate-xcodeproj  # Generate Xcode project
```

### Example App Development (CocoaPods)
```bash
cd Example
pod install                    # Install dependencies
open Mute.xcworkspace         # Open workspace in Xcode

# Build from command line
xcodebuild -workspace Mute.xcworkspace -scheme Mute-Example build

# Build for simulator
xcodebuild -workspace Mute.xcworkspace -scheme Mute-Example -destination 'platform=iOS Simulator,name=iPhone 14' build
```

### Testing
```bash
# Run tests (Note: currently no test files exist)
xcodebuild -workspace Mute.xcworkspace -scheme Mute-Example test
```

## Key Implementation Details

**Mute Detection Logic**: The library measures the time difference between starting and finishing playback of a silent sound. If elapsed time < 0.1 seconds, device is considered muted.

**Background Handling**: Automatically pauses detection when app enters background and resumes when returning to foreground.

**Configuration Options**:
- `checkInterval`: Detection frequency (minimum 0.5 seconds)
- `alwaysNotify`: Whether to notify every check or only on state change
- `isPaused`: Control detection state

**Dependencies**: AudioToolbox framework for system sound playback, Foundation for basic functionality.