# Phase 11: CI/CD & Deployment Strategy 🚀

Welcome to **Phase 11**. A Senior Engineer doesn't just "ship it"; they build a reliable, automated pipeline that ensures every release is high-quality and low-risk.

---

## 📋 Table of Contents
1. [The CI/CD Pipeline](#1-the-cicd-pipeline)
2. [Automated Testing in CI](#2-automated-testing-in-ci)
3. [Fastlane: Automation Swiss Army Knife](#3-fastlane-automation-swiss-army-knife)
4. [OTA Updates (Expo Updates/CodePush)](#4-ota-updates-expo-updatescodepush)
5. [App Store & Play Store Best Practices](#5-app-store--play-store-best-practices)
6. [Staging vs. Production Environments](#6-staging-vs-production-environments)

---

## 1. The CI/CD Pipeline
A typical React Native CI pipeline includes:
1. **Lint & Type Check:** Run ESLint and TypeScript compiler.
2. **Unit & Integration Tests:** Run Jest/RNTL.
3. **Build:** Generate `.ipa` (iOS) and `.apk`/`.aab` (Android).
4. **Deploy:** Upload to TestFlight, Firebase App Distribution, or Play Store.

---

## 2. Automated Testing in CI
Never merge code without passing tests.
- **GitHub Actions / CircleCI / Bitrise:** Popular choices for RN automation.
- **E2E in CI:** Running Detox in CI is powerful but expensive and slow. Most teams run it only on PRs to `main`.

---

## 3. Fastlane: Automation Swiss Army Knife
Fastlane automates the most tedious parts of deployment:
- **Match:** Sync certificates and provisioning profiles across the team.
- **Screengrab/Deliver:** Automate screenshots and metadata uploads.
- **Supply:** Upload binaries to the Google Play Store.

---

## 4. OTA Updates (Expo Updates/CodePush)
**Over-the-Air (OTA)** updates allow you to fix bugs in the JS layer without a full store review.

| Feature | CodePush (App Center) | Expo Updates |
| :--- | :--- | :--- |
| **Platform** | Vanilla/Expo Bare | Expo (Go/Bare) |
| **Ease of Use** | High | Very High |
| **Customization** | Flexible | Opinionated |

> [!CAUTION]
> **Caution:** OTA updates can only change JavaScript and assets. Any changes to native code (adding a new library) REQUIRE a new store submission.

---

## 5. App Store & Play Store Best Practices
- **Phased Releases:** Release to 1%, 5%, 20%, then 100% of users to catch crashes early.
- **Changelogs:** Keep them clear and localized.
- **App Review:** Be prepared for Apple's strict human interface guidelines.

---

## 6. Staging vs. Production Environments
Use different bundle IDs (e.g., `com.app.staging` vs `com.app.prod`) to allow both versions to live on the same device.

---

## 🏁 Next Steps
You've built it, secured it, and deployed it. Now, you need to defend it. In **Phase 12**, we prepare for the **Portfolio & Final Defense**, where you'll learn to articulate your technical choices to stakeholders.

[Back to Main README](../README.md) | [Go to Phase 12: Portfolio Defense](./phase12-portfolio-defense.md)
