<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>OPEN_TREE</H1>
Section: Linux Programmer's Manual (2)<BR>Updated: 2020-08-24<BR><A HREF="#index">Index</A>
<HR>

<A NAME="lbAB">&nbsp;</A>
<H2>NAME</H2>

open_tree - Pick or clone mount object and attach to fd
<A NAME="lbAC">&nbsp;</A>
<H2>SYNOPSIS</H2>

<PRE>
<B>#include &lt;sys/types.h&gt;</B>
<B>#include &lt;sys/mount.h&gt;</B>
<B>#include &lt;unistd.h&gt;</B>
<B>#include &lt;fcntl.h&gt; </B>/* Definition of AT_* constants */

<B>int open_tree(int </B><I>dirfd</I><B>, const char *</B><I>pathname</I><B>, unsigned int </B><I>flags</I><B>);</B>
</PRE>

<P>

<I>Note</I>:

There are no glibc wrappers for these system calls.
<A NAME="lbAD">&nbsp;</A>
<H2>DESCRIPTION</H2>

<B>open_tree</B>()

picks the mount object specified by the pathname and attaches it to a new file
descriptor or clones it and attaches the clone to the file descriptor. The
resultant file descriptor is indistinguishable from one produced by
<B><A HREF="https://man7.org/linux/man-pages/man2/open.2.html">open</A></B>(2) with <B>O_PATH</B>.

<P>

In the case that the mount object is cloned, the clone will be &quot;unmounted&quot; and
destroyed when the file descriptor is closed if it is not otherwise mounted
somewhere by calling
<B><A HREF="move_mount.md">move_mount</A></B>(2).

<P>

To select a mount object, no permissions are required on the object referred
to by the path, but execute (search) permission is required on all of the
directories in
<I>pathname</I>

that lead to the object.
<P>

Appropriate privilege (Linux: the
<B>CAP_SYS_ADMIN</B>

capability) is required to clone mount objects.
<P>

<B>open_tree</B>()

uses
<I>pathname</I>, <I>dirfd</I> and <I>flags</I>

to locate the target object in one of a variety of ways:
<DL COMPACT>
<DT>[*] By absolute path.<DD>
<I>pathname</I>

points to an absolute path and
<I>dirfd</I>

is ignored. The object is looked up by name, starting from the root of the
filesystem as seen by the calling process.
<DT>[*] By cwd-relative path.<DD>
<I>pathname</I>

points to a relative path and
<I>dirfd</I> is <I>AT_FDCWD</I>.

The object is looked up by name, starting from the current working directory.
<DT>[*] By dir-relative path.<DD>
<I>pathname</I>

points to relative path and
<I>dirfd</I>

indicates a file descriptor pointing to a directory. The object is looked up
by name, starting from the directory specified by
<I>dirfd</I>.

<DT>[*] By file descriptor.<DD>
<I>pathname</I>

is &quot;&quot;,
<I>dirfd</I>

indicates a file descriptor and
<B>AT_EMPTY_PATH</B>

is set in
<I>flags</I>.

The mount attached to the file descriptor is queried directly. The file
descriptor may point to any type of file, not just a directory.
</DL>
<P>

<I>flags</I>

can be used to control the operation of the function and to influence a
path-based lookup. A value for
<I>flags</I>

is constructed by OR'ing together zero or more of the following constants:
<DL COMPACT>
<DT><B>AT_EMPTY_PATH</B>

<DD>

If
<I>pathname</I>

is an empty string, operate on the file referred to by
<I>dirfd</I>

(which may have been obtained from
<B><A HREF="https://man7.org/linux/man-pages/man2/open.2.html">open</A></B>(2) with

<B>O_PATH</B>, from <B><A HREF="fsmount.md">fsmount</A></B>(2)

or from another
<B>open_tree</B>()).

If
<I>dirfd</I>

is
<B>AT_FDCWD</B>,

the call operates on the current working directory.
In this case,
<I>dirfd</I>

can refer to any type of file, not just a directory.
This flag is Linux-specific; define
<B>_GNU_SOURCE</B>


to obtain its definition.
<DT><B>AT_NO_AUTOMOUNT</B>

<DD>
Don't automount the final (&quot;basename&quot;) component of
<I>pathname</I>

if it is a directory that is an automount point. This flag allows the
automount point itself to be picked up or a mount cloned that is rooted on the
automount point. The
<B>AT_NO_AUTOMOUNT</B>

flag has no effect if the mount point has already been mounted over.
This flag is Linux-specific; define
<B>_GNU_SOURCE</B>


to obtain its definition.
<DT><B>AT_SYMLINK_NOFOLLOW</B>

<DD>
If
<I>pathname</I>

is a symbolic link, do not dereference it: instead pick up or clone a mount
rooted on the link itself.
<DT><B>OPEN_TREE_CLOEXEC</B>

<DD>
Set the close-on-exec flag for the new file descriptor. This will cause the
file descriptor to be closed automatically when a process exec's.
<DT><B>OPEN_TREE_CLONE</B>

<DD>
Rather than directly attaching the selected object to the file descriptor,
clone the object, set the root of the new mount object to that point and
attach the clone to the file descriptor.
<DT><B>AT_RECURSIVE</B>

<DD>
This is only permitted in conjunction with OPEN_TREE_CLONE. It causes the
entire mount subtree rooted at the selected spot to be cloned rather than just
that one mount object.
</DL>
<A NAME="lbAE">&nbsp;</A>
<H2>RETURN VALUE</H2>

On success, the new file descriptor is returned. On error, -1 is returned,
and
<I>errno</I>

is set appropriately.
<A NAME="lbAF">&nbsp;</A>
<H2>ERRORS</H2>

<DL COMPACT>
<DT><B>EACCES</B>

<DD>
Search permission is denied for one of the directories
in the path prefix of
<I>pathname</I>.

(See also
<B><A HREF="https://man7.org/linux/man-pages/man7/path_resolution.7.html">path_resolution</A></B>(7).)

<DT><B>EBADF</B>

<DD>
<I>dirfd</I>

is not a valid open file descriptor.
<DT><B>EFAULT</B>

<DD>
<I>pathname</I>

is NULL or
<I>pathname</I>

point to a location outside the process's accessible address space.
<DT><B>EINVAL</B>

<DD>
Reserved flag specified in
<I>flags</I>.

<DT><B>ELOOP</B>

<DD>
Too many symbolic links encountered while traversing the pathname.
<DT><B>ENAMETOOLONG</B>

<DD>
<I>pathname</I>

is too long.
<DT><B>ENOENT</B>

<DD>
A component of
<I>pathname</I>

does not exist, or
<I>pathname</I>

is an empty string and
<B>AT_EMPTY_PATH</B>

was not specified in
<I>flags</I>.

<DT><B>ENOMEM</B>

<DD>
Out of memory (i.e., kernel memory).
<DT><B>ENOTDIR</B>

<DD>
A component of the path prefix of
<I>pathname</I>

is not a directory or
<I>pathname</I>

is relative and
<I>dirfd</I>

is a file descriptor referring to a file other than a directory.
</DL>
<A NAME="lbAG">&nbsp;</A>
<H2>VERSIONS</H2>

<B>open_tree</B>()

was added to Linux in kernel 5.2.
<A NAME="lbAH">&nbsp;</A>
<H2>CONFORMING TO</H2>

<B>open_tree</B>()

is Linux-specific.
<A NAME="lbAI">&nbsp;</A>
<H2>NOTES</H2>

Glibc does not (yet) provide a wrapper for the
<B>open_tree</B>()

system call; call it using
<B><A HREF="https://man7.org/linux/man-pages/man2/syscall.2.html">syscall</A></B>(2).

<A NAME="lbAJ">&nbsp;</A>
<H2>EXAMPLE</H2>

The
<B>open_tree</B>()

function can be used like the following:
<P>

<DL COMPACT><DT><DD>
<PRE>
fd1 = open_tree(AT_FDCWD, &quot;/mnt&quot;, 0);
fd2 = open_tree(fd1, &quot;&quot;,
 AT_EMPTY_PATH | OPEN_TREE_CLONE | AT_RECURSIVE);
move_mount(fd2, &quot;&quot;, AT_FDCWD, &quot;/mnt2&quot;, MOVE_MOUNT_F_EMPTY_PATH);
</PRE>

</DL>

<P>

This would attach the path point for &quot;/mnt&quot; to fd1, then it would copy the
entire subtree at the point referred to by fd1 and attach that to fd2; lastly,
it would attach the clone to &quot;/mnt2&quot;.
<A NAME="lbAK">&nbsp;</A>
<H2>SEE ALSO</H2>

<B><A HREF="fsmount.md">fsmount</A></B>(2),

<B><A HREF="move_mount.md">move_mount</A></B>(2),

<B><A HREF="https://man7.org/linux/man-pages/man2/open.2.html">open</A></B>(2)

<P>

<HR>
<A NAME="index">&nbsp;</A><H2>Index</H2>
<DL>
<DT><A HREF="#lbAB">NAME</A><DD>
<DT><A HREF="#lbAC">SYNOPSIS</A><DD>
<DT><A HREF="#lbAD">DESCRIPTION</A><DD>
<DT><A HREF="#lbAE">RETURN VALUE</A><DD>
<DT><A HREF="#lbAF">ERRORS</A><DD>
<DT><A HREF="#lbAG">VERSIONS</A><DD>
<DT><A HREF="#lbAH">CONFORMING TO</A><DD>
<DT><A HREF="#lbAI">NOTES</A><DD>
<DT><A HREF="#lbAJ">EXAMPLE</A><DD>
<DT><A HREF="#lbAK">SEE ALSO</A><DD>
</DL>
<HR>
This document was created by
<A HREF="http://primates.ximian.com/~flucifredi/man/">man2html</A>,
using the manual page posted
<A HREF="https://lwn.net/ml/linux-kernel/159827188271.306468.16962617119460123110.stgit@warthog.procyon.org.uk/">here</A>.
</BODY>
</HTML>
