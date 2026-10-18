# Gnome Shell Extension Maximized Into Empty Workspace (Gnome 46+)

New and maximized windows will be moved to empty workspaces.

When a window is maximized (or fullscreen), it is moved to a free workspace and
the extension performs the **native GNOME Shell workspace-switch animation**
(the same smooth animation you get with trackpad gestures), so you clearly see
that you landed on a new workspace. When the window is unmaximized, restored or
closed, the same animation brings you back to the previous workspace.

Get this GNOME Extension at:  https://extensions.gnome.org/extension/8851/maximize-window-into-new-workspace/

This is a fork of [balintbarna's version](https://github.com/balintbarna/gnome-shell-extension-maximize-to-empty-workspace), updated to support Gnome 46+.

## Supported GNOME Shell versions

`46`, `47`, `48`, `49`, `50`, `51` (see `MaximizeWindowIntoNewWorkspace/metadata.json`).

## Installation

### From extensions.gnome.org (end users)

1. Visit the [extension page](https://extensions.gnome.org/extension/8851/maximize-window-into-new-workspace/).
2. Click the toggle to install it, or install the "Extension Manager" / "GNOME
   Shell Integration" browser add-on first if the toggle does not appear.
3. Enable it in **Extension Manager** or with:
   ```sh
   gnome-extensions enable MaximizeWindowIntoNewWorkspace@kyleross.com
   ```

### From source (for testing changes)

```sh
git clone https://github.com/Peheux/MaximizeWindowIntoNewWorkspace.git
cd MaximizeWindowIntoNewWorkspace
./deploy.sh
```

`deploy.sh` copies `MaximizeWindowIntoNewWorkspace/` to
`~/.local/share/gnome-shell/extensions/MaximizeWindowIntoNewWorkspace@kyleross.com/`.

Then enable it:

```sh
gnome-extensions enable MaximizeWindowIntoNewWorkspace@kyleross.com
```

### Reloading during development

GNOME Shell caches extension code, so after editing `extension.js` you must
reload it:

- **X11**: restart the Shell in place with `Alt+F2`, type `r`, press Enter,
  then re-enable the extension.
- **Wayland** (default on most distros): the Shell cannot be restarted in
  place. Log out and log back in, or use the convenient tool:
  ```sh
  # after ./deploy.sh, restart the extension without logging out (Wayland ok)
  busctl --user call org.gnome.Shell /org/gnome/Shell org.gnome.Shell Eval s 'Meta.restart("Reloading...")'
  ```
  If `Eval` is disabled on your system, a logout/login is the reliable option.

To disable/remove:

```sh
gnome-extensions disable MaximizeWindowIntoNewWorkspace@kyleross.com
rm -rf ~/.local/share/gnome-shell/extensions/MaximizeWindowIntoNewWorkspace@kyleross.com
```

## Testing the behaviour

Quick manual test plan (two normal windows A and B on the same workspace):

1. **Maximize → new workspace**: maximize window A. Expected: A moves to a new
   (empty) workspace, the native horizontal workspace-switch animation plays,
   and you are now on A's workspace.
2. **Unmaximize → back**: unmaximize A. Expected: A returns next to B and the
   same animation plays in the opposite direction.
3. **Fullscreen**: same as 1/2 with a fullscreen video (e.g. YouTube), including
   leaving fullscreen with `Esc`.
4. **Close window**: maximize A, then close it. Expected: animated switch back
   to the workspace holding B (A is destroyed, it is not moved).
5. **Minimize**: maximize A, then minimize it. Expected: animated switch back
   to B's workspace; unminimize A → animated switch to A's workspace again.
6. **No animation cases (expected)**: maximizing the only window on a
   workspace, or maximizing when there is no free workspace left, does nothing
   (there is no workspace change to animate).
7. **Multi-monitor**: with `workspaces-only-on-primary` enabled (GNOME
   default), only windows on the primary monitor are affected.

Notes on deliberate safeguards (verify they still hold):

- No reordering during the first ~10 s after login (startup grace period).
- Nothing happens while the Activities overview is open; pending actions run
  when it closes.
- Short-lived/transient windows are ignored.

## Development

Requirements: `node` ≥ 20 and `npm`.

```sh
npm install
npm run lint        # ESLint with the official 'eslint-config-gnome' rules
```

Create a distributable zip (what you would upload to extensions.gnome.org):

```sh
./makezip.sh        # produces MaximizeWindowIntoNewWorkspace.zip in the repo root
```

### Code layout

Everything lives in `MaximizeWindowIntoNewWorkspace/extension.js`:

- `placeOnWorkspace(win)` — moves a maximized window to the adjacent free
  workspace and activates it (native animated switch).
- `backto(win)` — moves the window back to the previously occupied workspace
  and activates it (native animated switch).
- `getFirstFreeMonitor` / `getLastOcupiedMonitor` — workspace lookup helpers.
- `window_manager_*` — signal handlers wired to `global.window_manager`
  (size-change, map, destroy, minimize, unminimize).

## CI/CD

A GitHub Actions workflow runs on every push and pull request (`.github/workflows/ci.yml`):

1. **Lint** — `npm run lint` with the official GNOME ESLint configuration, so
   style regressions are caught before review.
2. **Package** — builds the installable zip with `./makezip.sh` and uploads it
   as a workflow artifact, so anyone can download and test a branch without
   cloning.

Possible future extensions: a `tag`-triggered job creating a GitHub Release
with the zip, and an `eslint-formatter-junit` step publishing lint results in
the GitHub Actions summary.
