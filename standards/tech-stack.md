# Tech Stack

## Context

Tech stack for Palma Android Application - Advanced Bionics hearing device companion app.

## Core Platform
- Platform: Android (Native)
- Language: Kotlin
- Architecture Pattern: MVVM
- UI Framework: Jetpack Compose
- Dependency Injection: Hilt
- Build System: Gradle 8.5.0
- JVM Target: Java 17

## Key Dependencies
- Sonova Mobile SDK: 19.0.0-2120 (hearing device communication)
- Compose BOM: 2025.06.01
- Hilt: 2.56.2
- Kotlin Coroutines: For async operations
- Jetpack Navigation Compose: App navigation
- Firebase: Analytics and crash reporting

## Android Configuration
- Minimum SDK: 26 (Android 8.0)
- Target SDK: 34 (Android 14)
- Compile SDK: 35 (Android 15)

## Module Structure
- app: Main application module (UI, ViewModels, Activities)
- common: Shared utilities and configurations
- controls: Reusable UI components
- data: Data persistence layer with DataStore
- service_layer: Business logic, DTOs, service interfaces

## Development Tools
- IDE: Android Studio Koala | 2024.1.1 Patch 1+
- Testing Framework: JUnit 4, MockK, Compose UI Testing
- Code Quality: Lint, Ktlint
- Deployment: Fastlane (AppCenter, Google Play Store)

## External Services
- Sonova Artifactory: https://sonova.jfrog.io/artifactory/sonova-android-gradle-release
- Firebase: Analytics and crash reporting
- Microsoft AppCenter: Beta distribution
