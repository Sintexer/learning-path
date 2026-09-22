
### Step 0: The 1-Minute Prerequisite Setup
By default, macOS does not let you `Tab` through buttons in alert windows. Fix this immediately:
1. Open **System Settings → Keyboard**.
2. Turn on **Keyboard navigation** (or "Full Keyboard Access").
3. While there, set **Key repeat rate** to *Fast* and **Delay until repeat** to *Short*.

---

### 1. App & Window Management ("Folding" & Hiding)

| Shortcut | Action | Developer Use Case |
| :--- | :--- | :--- |
| **`Cmd + H`** | **Hide active app** | "Quick fold" for an app. Instantly removes it from sight without closing it. |
| **`Cmd + Option + H`** | **Hide all *other* apps** | Instantly clear clutter and focus on just your editor/terminal. |
| **`Cmd + M`** | **Minimize window** | Folds the current window into the Dock. |
| **`Cmd + Option + M`** | **Minimize all windows** | Folds all windows of the active app to the Dock. |
| **`Cmd + W`** | **Close window/tab** | Closes current window or tab. |
| **`Cmd + Option + W`** | **Close all windows** | Closes all open windows of the current app. |
| **`Cmd + Q`** | **Quit app** | Completely terminates the process. |
| **`Cmd + ` `~`** *(tilde)* | **Cycle windows of the *same* app** | Switch between two open VS Code or Chrome windows. |
| **`Cmd + Tab`** | **App switcher** | While holding `Cmd`: press `Q` to quit an app, or `H` to hide it. |

---

### 2. Native Window Tiling & Snapping (macOS Sequoia+)
Apple introduced native window snapping shortcuts:

* **`Fn + Ctrl + F`**: Fill / Maximize window.
* **`Fn + Ctrl + Left Arrow` / `Right Arrow`**: Snap window to the left / right half.
* **`Fn + Ctrl + Up Arrow` / `Down Arrow`**: Snap to top / bottom half.
* **`Fn + Ctrl + C`**: Center window on screen.
* **`Fn + Ctrl + R`**: Restore to previous window size.

---

### 3. Desktop Spaces & Mission Control

* **`Ctrl + Left / Right Arrow`**: Slide between virtual desktops (Spaces) or full-screen apps.
* **`Ctrl + Up Arrow`**: **Mission Control** (birds-eye view of all open windows).
* **`Ctrl + Down Arrow`**: **App Exposé** (shows all open windows belonging *only* to the current app).
* **`Cmd + F3`** (or `Fn + F11`): **Show Desktop** (pushes all windows aside).

---

### 4. Navigating Dialogs & System Menus Without a Mouse

* **`Enter`**: Confirms the primary/blue button (e.g., "OK", "Save").
* **`Esc`** or **`Cmd + .`**: Cancels or dismisses prompts and modal dialogs.
* **`Spacebar`**: Triggers whichever secondary button has a focus ring around it (after pressing `Tab`).
* **`Cmd + Delete`**: In "Save Changes?" dialogs, this immediately triggers **"Don't Save" / "Discard"**.
* **`Cmd + Shift + /` (`Cmd + ?`)**: **Search Menu Bar**. Jumps directly into the "Help" menu search field. Type any command (e.g., "Reopen Closed Tab", "Export") and hit `Enter` to run it.
* **`Ctrl + F2`** (or `Fn + Ctrl + F2`): Puts keyboard focus directly onto the Apple icon in the menu bar; use arrow keys to navigate.
* **`Cmd + Ctrl + Q`**: Lock screen immediately.
* **`Cmd + Option + Esc`**: Force Quit dialog.

---

### 5. Universal Developer Text Navigation (Works Across All macOS Apps)
macOS has standard Unix/Emacs keybindings built directly into its text engine (works in the Terminal, VS Code, Slack, Safari, Notes, etc.):

#### Word & Line Jumps
* **`Option + Left / Right`**: Move cursor one word backward/forward.
* **`Cmd + Left / Right`**: Move cursor to start/end of current line.
* **`Cmd + Up / Down`**: Jump to the very beginning/end of a file or page.
* **`Shift + [any movement]`**: Selects text while moving (e.g., `Cmd + Shift + Right` highlights to the end of the line).

#### Deletion
* **`Option + Delete`**: Delete previous word.
* **`Cmd + Delete`**: Delete entire line to the left of the cursor.
* **`Fn + Delete`** (or **`Ctrl + D`**): Forward delete (deletes the character to the right).

#### Unix / Terminal Native Jumpers
* **`Ctrl + A`**: Jump to start of line.
* **`Ctrl + E`**: Jump to end of line.
* **`Ctrl + K`**: Kill (cut) from cursor to end of line.
* **`Ctrl + Y`**: Yank (paste) previously killed text.
* **`Ctrl + U`**: Clear/delete entire line before cursor (in Terminal).

---

### 6. Finder Power Shortcuts

* **`Cmd + Shift + .`**: **Toggle hidden dotfiles** (reveals `.env`, `.git/`, `.zshrc` in Finder and open/save dialogs).
* **`Spacebar`**: **Quick Look** (previews images, PDFs, code files, and markdown without launching an app).
* **`Cmd + Down Arrow`** (or `Cmd + O`): Open selected folder/file.
* **`Cmd + Up Arrow`**: Go up one directory level.
* **`Cmd + Option + C`**: **Copy pathname as text** (copies absolute path `/Users/...` to clipboard for the terminal).
* **`Cmd + Shift + G`**: Open "Go to Folder" path prompt.

---

### Recommended Tooling for a 100% Mouse-Free Setup
If you want to eliminate the trackpad completely, consider these developer staples:

1. **[Raycast](https://www.raycast.com)**: Replaces Spotlight (`Cmd + Space`). Provides built-in window management, clipboard history, quick script execution, and GitHub/Jira integrations.
2. **[AeroSpace](https://github.com/nikitabobko/AeroSpace)** or **[Rectangle](https://rectangleapp.com/)**: AeroSpace brings an i3/Sway-like tiling window manager to macOS; Rectangle provides standard keyboard-based window resizing.
3. **[Vimium](https://vimium.github.io/)** (Chrome / Brave / Firefox extension): Allows you to browse the web, click links, and navigate pages using only Vim-style keys (`f` to hint links, `j`/`k` to scroll).
4. **[Homerow](https://www.homerow.app/)**: Generates keyboard shortcuts over any clickable UI button or icon on macOS, allowing clicks anywhere without reaching for the mouse.