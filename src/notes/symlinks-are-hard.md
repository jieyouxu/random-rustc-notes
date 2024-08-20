# Symlinks are hard

... and it's harder in relation to the Rust stdlib, for better or worse.

// TODO(jieyouxu): learn this and write it up tmrw

## Relevant links

// TODO(jieyouxu): tidy this up, currently just dumping all relevant links since I still have to read
and understand them.

- <https://en.wikipedia.org/wiki/Symbolic_link>
- symlink: <https://man7.org/linux/man-pages/man2/symlink.2.html>
- link syscall: <https://man7.org/linux/man-pages/man2/link.2.html>
- <https://en.wikipedia.org/wiki/Hard_link>
- Symlinks in Windows 10!: <https://blogs.windows.com/windowsdeveloper/2016/12/02/symlinks-windows-10/>
- Hard links and junctions: <https://learn.microsoft.com/en-us/windows/win32/fileio/hard-links-and-junctions>
- compiletest `aggressive_rm_rf` [issue](https://github.com/rust-lang/rust/issues/129155) and
  [PR](https://github.com/rust-lang/rust/pull/129302).
- bootstrap clean `rm_rf` [issue](https://github.com/rust-lang/rust/issues/112544) and
  [PR](https://github.com/rust-lang/rust/pull/129187)
- my review comment for `incr-add-rust-src-component` rmake.rs PR:
  <https://github.com/rust-lang/rust/pull/128562#discussion_r1719902008>
