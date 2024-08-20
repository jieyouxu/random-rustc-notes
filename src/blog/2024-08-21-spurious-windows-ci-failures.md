# Investigating spurious Windows CI failures

| Metadata       |            |
|----------------|------------|
| Date published | 2024-08-21 |

// TODO(jieyouxu): keep updating this page

## Background

> **Tracking issue**
>
> [High failure rate on Windows MSVC CI with filesystem errors #127883][tracking-issue].

There's some Windows CI spurious failures that's been plaguing our CI for a good while now (a few
month as of 2024-08-21). They look like one of:

```
error: couldn't create a temp dir: Access is denied. (os error 5) at path "C:\\a\\_temp\\msys64\\tmp\\rustcFKhJYE"
```

```
failed to copy `C:\a\rust\rust\build\x86_64-pc-windows-msvc\stage1-tools\x86_64-pc-windows-msvc\release\miri.exe`
to `C:\a\rust\rust\build\x86_64-pc-windows-msvc\stage1-tools-bin\miri.exe`: The process cannot
access the file because it is being used by another process. (os error 32)
```

```
LINK : fatal error LNK1104: cannot open file 'C:\a\rust\rust\build\x86_64-pc-windows-msvc\stage1-tools\x86_64-pc-windows-msvc\release\deps\miri-bdcbed7500be44cb.exe'
```

```
error: failed to remove file `C:\a\rust\rust\build\x86_64-pc-windows-msvc\stage1-tools\x86_64-pc-windows-msvc\release\miri.exe`, Access is denied. (os error 5)
```

```
std\src\os\windows\process.rs - os::windows::process::CommandExt::raw_attribute (line 330)
called `Result::unwrap()` on an `Err` value: Os { code: 5, kind: PermissionDenied, message: "Access is denied."
```

## Initial suspicions

I have a couple of suspicions (but none confirmed yet):

1. Some kind of malware scanner is opening the executable files for scanning while bootstrap is
  trying to remove it.
2. Bootstrap is somehow mishandling symbolic links on Windows (see [Symlinks are
  hard](../notes/symlinks-are-hard.md) for some background on Windows symlinks).
    - There is precedent for bootstrap and compiletest mishandling (Windows) symlinks, see:
        - [bootstrap: fix clean's remove_dir_all implementation #129187][bootstrap-clean-fix] and
        - [compiletest: use std::fs::remove_dir_all now that it is available
          #129302][compiletest-rm-rf].
3. Bootstrap is somehow trying to remove an executable file which the process spawned from the file
  is still running (seems less likely).

My current working theory and primary suspect is (2) with some combination of (1).

// TODO

## Malware scanner?

[@ehuss][ehuss] [tried many methods of trying to inspect which process is holding file handles of
the executable files when bootstrap is trying to remove them][windows-ci-try-pr], but unfortunately
did not obtain much information or discover conclusive evidence.

I don't think I would have much luck in this avenue, either.

## Bootstrap investigations

... so I'm going to look at bootstrap's filesystem interactions instead. Namely:

- How symbolic links (soft links / hard links on linux, and symlinks vs junction points vs hard
  links on Windows) are handled in bootstrap. And in conjunction with,
- How directories are recursively created and removed. In bootstrap's clean.rs, previously there was
  a `rm_rf` that tried to account for read-only files and symlinks on Windows [but was
  incorrect][bootstrap-clean-fix].
- `compiletest` had some weird path canonicalization dances. I wonder if bootstrap also has them.

// TODO

[tracking-issue]: https://github.com/rust-lang/rust/issues/127883
[bootstrap-clean-fix]: https://github.com/rust-lang/rust/pull/129187
[compiletest-rm-rf]: https://github.com/rust-lang/rust/pull/129302
[remove_dir_all]: https://doc.rust-lang.org/std/fs/fn.remove_dir_all.html
[ehuss]: https://github.com/ehuss
[windows-ci-try-pr]: https://github.com/rust-lang/rust/pull/127373
