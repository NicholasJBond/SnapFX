# ADR 0009: Framework-Managed Floating Window Maximize

- **Status**: Accepted
- **Date**: 2026-10-24

## Context

Floating windows are `StageStyle.UNDECORATED` stages with a custom title bar that provides maximize/restore,
double-click toggle, and drag-from-maximized restore.

The original implementation delegated to native `Stage.setMaximized(...)`. On macOS, JavaFX maximizes undecorated
(borderless) windows through `NSWindow` zoom, which caused:

- unreliable `Stage.isMaximized()` reports (for example `true` directly after a normal show)
- animated native restore frames that move the window toward a stale frame at the screen's top-left corner
- drag-from-maximized fighting that animation, so the window repeatedly jumped back to the top-left while dragging

## Decision Drivers

- Identical maximize/restore behavior on Windows, Linux, and macOS
- Deterministic state for title-bar drag and resize logic
- No dependency on platform-specific native window-manager behavior for undecorated stages

## Considered Options

### Option A: Keep native maximize and add macOS-specific workarounds

- Pros: uses platform semantics where they work
- Cons: depends on timing of asynchronous native callbacks; hard to test and fragile across macOS/JavaFX versions

### Option B (accepted): Framework-managed maximize state

- Pros: deterministic, platform-independent, fully unit-testable
- Cons: native window-manager maximize gestures are not reflected in SnapFX maximize state

## Decision

We use **Option B**.

- `DockFloatingWindow` tracks maximize state in its own field and never calls `Stage.setMaximized(...)`.
- Maximize remembers current bounds and applies the visual bounds of the screen containing the window.
- Restore (button, double-click, or drag-from-maximized) re-applies the remembered bounds.
- Bounds capture/hide, resize guards, and the maximize-button icon use the managed state, not `Stage.isMaximized()`.

## Consequences

Positive:

- Drag-from-maximized restores exactly once per gesture and follows the pointer on all platforms
- Maximize state is consistent with the custom title-bar controls

Trade-offs:

- `Stage.isMaximized()` of floating stages is no longer meaningful for SnapFX behavior
- Maximized bounds do not automatically follow screen visual-bounds changes (for example dock/taskbar resize)
