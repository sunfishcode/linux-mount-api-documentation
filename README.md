Linux's `fspick`, `fsconfig`, `open_tree`, `move_mount`, `fsopen`, and
`fsmount` syscalls currently lack official documentation. It appears that
drafts of manual pages were [posted to the Linux mailing list], but not
finished.

[posted to the Linux mailing list]: https://lwn.net/ml/linux-kernel/159827191245.306468.4903071494263813779.stgit@warthog.procyon.org.uk/

This repository contains HTML renderings of those drafts, generated using
man2html, with adjustments to URLs so that the links work:

<ul>
<li><a href="fsopen.md"><code>fsopen</code></a></li>
<li><a href="fsmount.md"><code>fsmount</code></a></li>
<li><a href="fspick.md"><code>fspick</code></a></li>
<li><a href="fsconfig.md"><code>fsconfig</code></a></li>
<li><a href="open_tree.md"><code>open_tree</code></a></li>
<li><a href="move_mount.md"><code>move_mount</code></a></li>
</ul>

The sources contain the following notices:

Copyright (c) 2020 David Howells <dhowells@redhat.com>


Permission is granted to make and distribute verbatim copies of this
manual provided the copyright notice and this permission notice are
preserved on all copies.

Permission is granted to copy and distribute modified versions of this
manual under the conditions for verbatim copying, provided that the
entire resulting derived work is distributed under the terms of a
permission notice identical to this one.

Since the Linux kernel and libraries are constantly changing, this
manual page may be incorrect or out-of-date.  The author(s) assume no
responsibility for errors or omissions, or for damages resulting from
the use of the information contained herein.  The author(s) may not
have taken the same level of care in the production of this manual,
which is licensed free of charge, as they might when working
professionally.

Formatted or processed versions of this manual, if unaccompanied by
the source, must acknowledge the copyright and authors of this work.
