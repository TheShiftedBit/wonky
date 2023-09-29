# Wonky Files
This is a collection of unusual files and directories, for use as test cases. You'll find everything from files named `*` to symlink cycles.

## What is this useful for?

Building more robust software.

A lot of characters have special meaning in certain programs, especially in file paths. For example, by default Bash interprets `user/ian/My Documents/foo.txt` as two arguments, one for `user/ian/My` and the other for `Documents/foo.txt`. This can wreak havoc on your shell scripts, but can be solved by putting arguments in quotes.

## What should I do about weird file paths? Can I just refuse to handle them?
You should try to handle weird file paths whenever you can. You might think nobody would name their file `*.cpp`, and you may be right; but what about `my awesome code.cpp`? Spaces are a lot more likely.

Failure to handle weird paths might even be a security vulnerability. For example, consider a pastebin-like service that stores user-controlled text files in `/data/<filename>.txt`. If an attacker calls their file `*.txt`, you'd store it at `/data/*.txt`. If your code for reading text files ran `cat /data/<filename>.txt`, reading the attacker's file would actually read _all_ pastes, showing the attacker everyone's secrets.

### Symlinks
Symlinks introduce an unusual situation. Generally, you should follow symlinks; but it's important to not get stuck in a symlink cycle. Any code for traversing a directory must check for cycles and refuse to follow a symlink to visit a place it's already visited.

In situations where users might be able to control a symlink - like in a tarball you're handling - you should only allow symlinks that point to things the user controls anyway. So symlinks in a tarball that point _within_ the tarball are fine, but a symlink that points to `/etc/shadow` is not.

### Weird filenames
Files like `*.txt`, `...`, etc. should be treated just like any other filename. In shell scripts, this means quoting them with the correct quote character. Most other languages handle them just fine unless you explicitly tell them otherwise.

### Sometimes, handling is impossible
Bazel uses `...` to signify "all directories recursively". There's no way to say "No, I mean the literal directory `...`". In situations like this, it's best to reject the weird thing outright - make bazel refuse to run if someone named their directory `...`. That avoids the risk of a security vulnerability, and lets the user know that something's wrong immediately rather than breaking things in weird ways later.

## Operating System Considerations
Different operating systems have different rules regarding what's a valid filename at all.

 - Linux supports arbitrary bytes. A filename can contain any byte except `/` or `\0`.
 - Windows uses UTF-16, but doesn't enforce surrogate pairs correctly.
 - Mac uses UTF-8.

This can mean archives created on one system might not be supported on others. If you make a filename on Linux that's invalid unicode, it simply can't be copied to Mac - whether in a zip file, git, or any other mechanism.

The `main` branch of this repo contains files that are valid on all three operating systems. There are separate branches `mac`, `windows`, and `linux` that also includes files valid on those machines. Finally, there's a `unix` branch for files valid on Mac and Linux but not Windows.