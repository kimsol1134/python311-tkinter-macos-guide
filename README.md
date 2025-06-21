# python311-tkinter-macos-guide
tkinter-macos-pyenv-fix

# tkinter‑macos‑pyenv‑fix

> **Goal**  Make `import tkinter` work on **Python 3.11 (pyenv)** running on **macOS Sequoia 15.5 / Apple Silicon (M4)**.

---

## Why it breaks

Homebrew now ships **Tcl/Tk 9.x** by default, but **Python 3.11** is only compatible with **Tcl/Tk 8.6**. During `pyenv install` the build script sees version 9 and quietly skips the `_tkinter` extension, so the final Python is GUI‑less.

---

## One‑liner fix

```bash
brew uninstall --ignore-dependencies tcl-tk && \
brew install tcl-tk@8 && brew link --force --overwrite tcl-tk@8 && \
TK8=$(brew --prefix tcl-tk@8) && \
export PATH="$TK8/bin:$PATH" LDFLAGS="-L$TK8/lib" \
       CPPFLAGS="-I$TK8/include" PKG_CONFIG_PATH="$TK8/lib/pkgconfig" && \
export SDKROOT=$(xcrun --show-sdk-path) ARCHFLAGS="-arch arm64" && \
export PYTHON_CONFIGURE_OPTS="--with-tcltk-includes='-I$TK8/include' \
  --with-tcltk-libs='-L$TK8/lib -ltcl8.6 -ltk8.6' --enable-shared" && \
pyenv uninstall -f 3.11.9 && pyenv install -v 3.11.9 && \
pyenv global 3.11.9 && pyenv rehash && \
python - <<'PY'  # should print Tk 8.6
import tkinter as tk, sys; print("Python", sys.version); print("Tk", tk.TkVersion)
PY
```

Skip the one‑liner? Follow the detailed steps below.

---

## Step‑by‑step

### 1  Remove Tcl/Tk 9

```bash
brew uninstall --ignore-dependencies tcl-tk
```

### 2  Install & link Tcl/Tk 8.6

```bash
brew install tcl-tk@8   # Homebrew labels it @8.6
brew link --force --overwrite tcl-tk@8
```

### 3  Set build flags

```bash
TK8=$(brew --prefix tcl-tk@8)
export PATH="$TK8/bin:$PATH"
export LDFLAGS="-L$TK8/lib"
export CPPFLAGS="-I$TK8/include"
export PKG_CONFIG_PATH="$TK8/lib/pkgconfig"
export SDKROOT=$(xcrun --show-sdk-path)
export ARCHFLAGS="-arch arm64"
export PYTHON_CONFIGURE_OPTS="\
  --with-tcltk-includes='-I$TK8/include' \
  --with-tcltk-libs='-L$TK8/lib -ltcl8.6 -ltk8.6' \
  --enable-shared"
```

### 4  Re‑install Python 3.11

```bash
pyenv uninstall -f 3.11.9
pyenv install -v 3.11.9   # keep the log if it fails
```

### 5  Verify

```bash
pyenv global 3.11.9 && pyenv rehash
python - <<'PY'
import tkinter as tk, sys
print("Python", sys.version)
print("Tk", tk.TkVersion, "Tcl", tk.TclVersion)
tk.Tk().destroy()
PY
```

You should see `Tk 8.6` and the script should exit without errors.

---

## Post‑install checklist

| Action                                                                              | Why                                    |
| ----------------------------------------------------------------------------------- | -------------------------------------- |
| `brew pin tcl-tk@8`                                                                 | Prevent accidental upgrade back to 9.x |
| Add the flag exports to your shell profile                                          | Persist across sessions                |
| Quick health check: `python -m tkinter -c 'import tkinter; tkinter.Tk().destroy()'` | Catch problems in new virtualenvs      |

---

## References

* [pyenv Issue #2790](https://github.com/pyenv/pyenv/issues/2790) – *Unable to install Python 3.11 with tkinter on macOS*
* [pyenv PR #2791](https://github.com/pyenv/pyenv/pull/2791) – *Fix CPPFLAGS order so python‑build finds Tcl/Tk 8.6 first*

---

## License

Released under **CC BY‑SA 4.0** – feel free to share or adapt with attribution.
