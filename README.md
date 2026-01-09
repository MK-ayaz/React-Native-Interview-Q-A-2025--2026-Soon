<style>
  .senior-handbook {
    background-color: #ffffff;
    color: #1e293b;
    line-height: 1.8;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    max-width: 1000px;
    margin: 0 auto;
    padding: 40px 20px;
  }

  .hero-section {
    text-align: center;
    padding: 60px 0;
    background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
    border-radius: 32px;
    color: white;
    margin-bottom: 60px;
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  }

  .hero-section h1 {
    color: #60a5fa;
    font-size: 3rem;
    margin-bottom: 16px;
    letter-spacing: -0.05em;
  }

  .pillar-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 24px;
    margin: 40px 0;
  }

  .pillar-card {
    background-color: #ffffff;
    border: 1px solid #e2e8f0;
    padding: 32px;
    border-radius: 24px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  }

  .pillar-card:hover {
    border-color: #3b82f6;
    transform: translateY(-4px);
    box-shadow: 0 20px 25px -5px rgba(59, 130, 246, 0.1);
  }

  .phase-table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0 12px;
  }

  .phase-row {
    background-color: #f8fafc;
    transition: all 0.2s ease;
  }

  .phase-row td {
    padding: 20px;
    border-top: 1px solid #e2e8f0;
    border-bottom: 1px solid #e2e8f0;
  }

  .phase-row td:first-child {
    border-left: 1px solid #e2e8f0;
    border-radius: 12px 0 0 12px;
    font-weight: 800;
    color: #3b82f6;
  }

  .phase-row td:last-child {
    border-right: 1px solid #e2e8f0;
    border-radius: 0 12px 12px 0;
  }

  .status-badge {
    padding: 4px 12px;
    border-radius: 9999px;
    font-size: 0.75rem;
    font-weight: 700;
    text-transform: uppercase;
  }

  .status-complete { background-color: #dcfce7; color: #166534; }
</style>

<div class="senior-handbook">

<div class="hero-section">
  <h1>React Native Architect</h1>
  <p style="font-size: 1.25rem; opacity: 0.9;">The Senior Interview Handbook for 2024-2025</p>
  <div style="margin-top: 24px;">
    <img src="https://img.shields.io/badge/Status-Production--Ready-34d399?style=for-the-badge" alt="Production Ready" />
    <img src="https://img.shields.io/badge/Engine-New--Architecture-60a5fa?style=for-the-badge" alt="New Architecture" />
  </div>
</div>

## 🏗️ Core Pillars

<div class="pillar-grid">
  <div class="pillar-card">
    <h3 style="color: #3b82f6; margin-top: 0;">Internals</h3>
    <p style="color: #64748b; font-size: 0.95rem;">Master JSI, TurboModules, and Fabric to understand the engine under the hood.</p>
  </div>
  <div class="pillar-card">
    <h3 style="color: #8b5cf6; margin-top: 0;">Performance</h3>
    <p style="color: #64748b; font-size: 0.95rem;">Achieve 60fps with Reanimated 3, Skia, and advanced memory profiling.</p>
  </div>
  <div class="pillar-card">
    <h3 style="color: #f59e0b; margin-top: 0;">Security</h3>
    <p style="color: #64748b; font-size: 0.95rem;">Defense-in-depth with SSL Pinning, Keychain, and Biometric security.</p>
  </div>
</div>

## 🗺️ Engineering Roadmap

<table class="phase-table">
  <tr class="phase-row">
    <td>01</td>
    <td><a href="phases/phase1-core-fundamentals.md">React Native Fundamentals</a></td>
    <td>What is RN?, Components, State/Props, Hooks</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>02</td>
    <td><a href="phases/phase2-react-fundamentals.md">React Fundamentals</a></td>
    <td>Hooks, Re-renders, Memoization</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>03</td>
    <td><a href="phases/phase3-navigation-architecture.md">Navigation & Arch</a></td>
    <td>Deep Linking, Persistence, Auth</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>04</td>
    <td><a href="phases/phase4-performance-memory.md">Performance & Memory</a></td>
    <td>Profiling, FlashList, JSI Bridge</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>05</td>
    <td><a href="phases/phase5-native-modules-apis.md">Native Modules & APIs</a></td>
    <td>TurboModules, C++, Hardware</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>06</td>
    <td><a href="phases/phase6-state-management.md">State Management</a></td>
    <td>Zustand, RTK, 4-Tier Model</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>07</td>
    <td><a href="phases/phase7-styling-design.md">Styling & Design</a></td>
    <td>Atomic Design, Tokens, Theme</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>08</td>
    <td><a href="phases/phase8-animations-gestures.md">Animations & Gestures</a></td>
    <td>Reanimated 3, RNGH, Physics</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>09</td>
    <td><a href="phases/phase9-testing-qa.md">Testing & QA</a></td>
    <td>Detox, MSW, Testing Trophy</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>10</td>
    <td><a href="phases/phase10-security-best-practices.md">Security Best Practices</a></td>
    <td>Keychain, SSL Pinning, Biometrics</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>11</td>
    <td><a href="phases/phase11-cicd-deployment.md">CI/CD & Deployment</a></td>
    <td>Fastlane, CodePush, OTA</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>12</td>
    <td><a href="phases/phase12-portfolio-defense.md">Portfolio Defense</a></td>
    <td>Architecture, Trade-offs, Leadership</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
  <tr class="phase-row">
    <td>Bonus</td>
    <td><a href="bonus/bonus-modules.md">Essential Modules & Use Cases</a></td>
    <td>React Navigation, Zustand, Reanimated, Axios, Sentry</td>
    <td><span class="status-badge status-complete">Completed</span></td>
  </tr>
</table>

---

<div align="center" style="margin-top: 60px; color: #64748b;">
  <p>Built for the elite 1% of React Native Engineers.</p>
</div>

</div>