# Flutter Vibe: Modular Agents & Rules for Best Flutter Practices 🚀🧩

This repository provides a set of **agents** and **rules** to help you build scalable, maintainable, and beautiful Flutter apps with best practices enforced by design. Use these as a toolkit or reference for your own projects, or as a starting point for new teams. ✨

## What is this? 🤔

- **Agents**: Modular guides for each major concern in Flutter development (architecture, state, UI, navigation, animation, validation, data, etc). Each agent describes responsibilities, required packages, forbidden patterns, and code patterns. 🧑‍💻
- **Rules**: Concrete, actionable rules for using specific packages and patterns (async state, images, validation, hooks, localization, motion, project structure, state management, routing, responsive design, SDGA, skeleton loading, animations, notifications, etc). 📏

Together, these files form a living documentation and code quality system for Flutter projects. 📚

---

## Agents Overview 🕵️‍♂️

- **Animation Agent** 🎞️: Ensures all animations use `flutter_staggered_animations`, Material Motion, and Skeletonizer. No manual animation controllers for entrances. All loading states use skeletons, not progress indicators.
- **Data Layer Agent** 🔌: All server state is managed with `fquery` and Supabase. UI never calls APIs directly. Repository pattern is enforced. No manual HTTP or FutureBuilder for server data.
- **Flutter Architect Agent** 🏗️: The master orchestrator. Enforces project structure, package compliance, code review, and cross-agent coordination. All other agents report to this one.
- **Navigation Agent** 🧭: All navigation uses `routefly` with file-based routing. No direct Navigator calls. Type-safe navigation and route guards are required.
- **State Management Agent** 🧠: Local state uses `flutter_riverpod`. No setState or Provider. Encourages testable, reactive state logic.
- **UI Component Agent** 🎨: All UI is built with the SDGA design system and `flutter_screenutil` for responsive design. No Material widgets directly. Accessibility and performance are required.
- **Validation Schema Agent** ✅: All validation uses `acanthis` schemas. No manual if/else or RegExp. Validation is type-safe and integrated with forms and APIs.

---

## Rules Overview 📜

- **Async State Management**: Use `fquery` for all async/server state. No manual FutureBuilder or custom caching. ⏳
- **Cached Network Images**: Use `cached_network_image` for all remote images. No Image.network or NetworkImage directly. 🖼️
- **Validation**: Use `acanthis` for all validation. No manual logic or custom validators. 🛡️
- **Hooks**: Use `flutter_hooks` for reusable stateful logic. No StatefulWidget when hooks can be used. 🪝
- **Localization**: Use `flutter_locales` for multi-language support. No hardcoded strings or manual localization. 🌍
- **Material Motion Animations**: Use the `animations` package for all transitions and motion patterns. 🎬
- **Project Structure**: Enforce a clear core/features split, repository pattern, and proper folder usage. 🗂️
- **Riverpod Local State**: Use `flutter_riverpod` for all local state. No setState or Provider. 🧬
- **Routefly Navigation**: Use `routefly` for all navigation and routing. No Navigator or go_router. 🛣️
- **ScreenUtil Responsive**: Use `flutter_screenutil` for all sizing, spacing, and font scaling. 📱
- **SDGA Design System**: Use `sdga_design_system` for all UI. No Material widgets or custom design systems. 🏛️
- **Skeletonizer Loading States**: Use `skeletonizer` for all loading states. No progress indicators or custom shimmer. 💀
- **Staggered Animations**: Use `flutter_staggered_animations` for all list/grid entrance animations. 🌀
- **Toastification Notifications**: Use `toastification` for all user feedback and notifications. 🔔

---

## How to Use 📝

1. **Read each agent and rule** to understand the standards and patterns enforced.
2. **Follow the code patterns and forbidden practices** in your own codebase.
3. **Adopt the recommended packages** for each concern (state, UI, navigation, etc).
4. **Share with your team** to ensure everyone is aligned on best practices. 🤝

## Why use this? 💡

- **Consistency**: Every developer follows the same patterns and standards. 📏
- **Scalability**: The architecture supports large teams and complex apps. 📈
- **Maintainability**: Clear separation of concerns and modular code. 🧩
- **Professional UI/UX**: SDGA design, responsive layouts, smooth animations, and accessible components. 🏅
- **Type Safety & Robustness**: Strong validation, error handling, and state management. 🦾

## Credits 🙏

Created for the best Flutter experience. Inspired by real-world production needs and modern Flutter architecture.

---

**Feel free to fork, adapt, and contribute!** 🌟
