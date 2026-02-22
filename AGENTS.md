# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ruqola is a Rocket.Chat desktop client for the KDE desktop environment. It is a C++/Qt6 application using KDE Frameworks 6, built with CMake and Ninja.

## Build Commands

Configure and build (debug):
```bash
cmake --preset dev
cmake --build --preset dev
```

Run all tests:
```bash
ctest --preset dev
```

Run a single test by name:
```bash
ctest --preset dev -R <test_name>
# Example: ctest --preset dev -R accountmanagertest
```

Build output goes to `build-<preset>/` (e.g., `build-dev/`).

Other useful presets: `asan` (sanitizers), `clazy` (Qt static analysis), `unity` (faster compilation), `coverage`.

## Architecture

The codebase follows a layered architecture with three main libraries under `src/`:

- **`core/` → `libruqolacore`**: Business logic, data models, and Rocket.Chat protocol handling. Contains `Connection` (WebSocket/DDP communication), `AccountManager` (multi-account support), message/room/user models, and feature modules organized in subdirectories (channels/, chat/, e2e/, emoji/, teams/, etc.).

- **`rocketchatrestapi-qt/` → `librocketchatrestapi-qt`**: REST API client layer. Each Rocket.Chat REST endpoint has a corresponding job class (e.g., `ChannelListJob`, `SendMessageJob`). Jobs are organized in subdirectories matching API categories.

- **`widgets/` → `libruqolawidgets`**: Qt Widgets UI layer. Contains dialogs, channel list, message views, room header, login widget, admin dialogs, and all visual components. Organized by feature area in subdirectories.

Data flows: **widgets → core → rocketchatrestapi-qt** (for REST calls) and **core ↔ WebSocket** (for real-time DDP protocol).

Additional components:
- `src/apps/`: Application entry point (`main.cpp`)
- `src/plugins/`: Authentication and text processing plugins
- `tests/`: Standalone GUI test applications (not unit tests)

## Testing

Unit tests (using QTest) live in `autotests/` subdirectories within each library:
- `src/core/autotests/`
- `src/rocketchatrestapi-qt/autotests/`
- `src/widgets/autotests/`

Tests typically use data-driven patterns with `_data()` methods and compare against JSON fixtures stored alongside the test files.

## Code Style

- Enforced via clang-format (WebKit base style, 160 column limit, Linux brace style)
- Pre-commit hooks configured: clang-format, codespell, gersemi (CMake formatting), shellcheck, markdownlint
- Clang-tidy configuration in `.clang-tidy`
- Install hooks: `pre-commit install -f`
- Run manually: `pre-commit run --all`

## Key Build Options

- `OPTION_USE_E2E_SUPPORT` (OFF by default): End-to-end encryption (experimental, requires OpenSSL)
- `OPTION_ADD_OFFLINE_SUPPORT` (ON by default): Local database caching
- `OPTION_USE_PLASMA_ACTIVITIES` (ON by default, Linux/FreeBSD only)

## Dependencies

- Qt 6.9.0+, KDE Frameworks 6.19.0+
- Required Qt modules: Core, Gui, Widgets, WebSockets, Network, NetworkAuth, MultimediaWidgets, Sql
- Required KF modules: CoreAddons, I18n, Crash, Notifications, IconThemes, SyntaxHighlighting, and many others (see CMakeLists.txt)

## License

GPL-2.0-or-later. Uses SPDX headers for REUSE compliance.
