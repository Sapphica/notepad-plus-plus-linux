# Notepad++ Linux: Find dialog navigation fix

**Contributor:** Shawna Flannigan (`Sapphica`)  
**Upstream project:** [mixelpixx/notepad-plus-plus-linux](https://github.com/mixelpixx/notepad-plus-plus-linux)  
**Pull request:** [#3, Fix Find dialog navigation in the current document](https://github.com/mixelpixx/notepad-plus-plus-linux/pull/3)  
**Changed file:** `core/MainWindow.cpp` (30 added lines)

## Bug found

The Find & Replace dialog opens with **Ctrl+F**, but the **Find Next** and **Find Previous** buttons do not move to matching text in the current document. The problem occurred even when the text was known to exist in the file.

I reproduced the failure with Notepad++ Linux v1.2.0 on Ubuntu 24.04 using a Python file. Searching from the Find dialog did not navigate to the expected matches; `grep` confirmed the search text was present in the file.

## Root cause

`FindReplaceDialog` already declares the `findNext` and `findPrevious` signals. The missing piece was in `MainWindow`: neither signal was connected to a search operation in the active editor. The dialog accepted the request, but no editor search was invoked.

## Fix

Added two signal connections and a shared search function in `MainWindow::setupUi()`:

- Connect `FindReplaceDialog::findNext` and `FindReplaceDialog::findPrevious` to QScintilla search on `currentEditor()`.
- Use the requested search direction, with wraparound at the document boundaries.
- Respect the match-case, whole-word and regular-expression checkboxes.
- Start at the appropriate selection boundary when text is selected, including reversed selections.
- Resolve `currentEditor()` on each search action so switching tabs changes the document being searched.
- Do nothing safely when there is no active editor or the search text is empty.

The change is confined to a 30-line insertion in `core/MainWindow.cpp`. No existing code was removed and the original source line endings were preserved.

## Verification

**Build environment:** Ubuntu 24.04, CMake 3.28.3, GCC 13.3, Qt 6.4.2, QScintilla 2.14.1.

The focused test used an out-of-tree C++ smoke harness with the real `MainWindow`, Find & Replace dialog buttons and options, and QScintilla editor:

- **Before:** unmodified upstream fails on the first Find Next action.
- **After:** the patched application passes the same search test.
- **Additional checks:** forward and backward search, document wraparound, normal and reversed selections, case-sensitive and whole-word matching, regex matches that consume text, UTF-8 text, switching tabs, no matching text, empty queries and documents, and no open editor.

The change was also built and manually checked against the original failing file, including launch from the installed desktop entry.

These are the validation results reported with the submitted PR, not a full application regression suite.

## Scope and known limitations

This patch fixes **Find Next / Find Previous navigation** in the current document. It does not alter Replace or Replace All.

Two separate cases remain outside this change:

- Zero-width regex searches (for example, `^`) can repeat at the same position.
- Split-pane focus tracking still uses the existing `currentEditor()` implementation, which resolves the primary tab widget.

## Submission

The source change is submitted in [upstream PR #3](https://github.com/mixelpixx/notepad-plus-plus-linux/pull/3) from the [`fix/find-dialog-search` branch](https://github.com/Sapphica/notepad-plus-plus-linux/tree/fix/find-dialog-search). The PR is subject to upstream maintainer review and merge.
