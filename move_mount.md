<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>MOVE_MOUNT</H1>
Section: Linux Programmer's Manual (2)<BR>Updated: 2020-08-24<BR><A HREF="#index">Index</A>
<HR>

<A NAME="lbAB">&nbsp;</A>
<H2>NAME</H2>

move_mount - Move mount objects around the filesystem topology
<A NAME="lbAC">&nbsp;</A>
<H2>SYNOPSIS</H2>

<PRE>
<B>#include &lt;sys/types.h&gt;</B>
<B>#include &lt;sys/mount.h&gt;</B>
<B>#include &lt;unistd.h&gt;</B>
<B>#include &lt;fcntl.h&gt; </B>/* Definition of AT_* constants */

<B>int move_mount(int </B><I>from_dirfd</I><B>, const char *</B><I>from_pathname</I><B>,</B>
<B> int </B><I>to_dirfd</I><B>, const char *</B><I>to_pathname</I><B>,</B>
<B> unsigned int </B><I>flags</I><B>);</B>
</PRE>

<P>

<I>Note</I>:

There is no glibc wrapper for this system call.
<A NAME="lbAD">&nbsp;</A>
<H2>DESCRIPTION</H2>

The
<B>move_mount</B>()

call moves a mount from one place to another; it can also be used to attach an
unattached mount that was created by
<B>fsmount</B>() or <B>open_tree</B>() with <B>OPEN_TREE_CLONE</B>.

<P>

If
<B>move_mount</B>()

is called repeatedly with a file descriptor that refers to a mount object,
then the object will be attached/moved the first time and then moved
repeatedly, detaching it from the previous mountpoint each time.
<P>

To access the source mount object or the destination mountpoint, no
permissions are required on the object itself, but if either pathname is
supplied, execute (search) permission is required on all of the directories
specified in
<I>from_pathname</I> or <I>to_pathname</I>.

<P>

The caller does, however, require the appropriate privilege (Linux: the
<B>CAP_SYS_ADMIN</B>

capability) to move or attach mounts.
<P>

<B>move_mount</B>()

uses
<I>from_pathname</I>, <I>from_dirfd</I> and part of <I>flags</I>

to locate the mount object to be moved and
<I>to_pathname</I>, <I>to_dirfd</I> and another part of <I>flags</I>

to locate the destination mountpoint. Each lookup can be done in one of a
variety of ways:
<DL COMPACT>
<DT>[*] By absolute path.<DD>
The pathname points to an absolute path and the dirfd is ignored. The file is
looked up by name, starting from the root of the filesystem as seen by the
calling process.
<DT>[*] By cwd-relative path.<DD>
The pathname points to a relative path and the dirfd is
<I>AT_FDCWD</I>.

The file is looked up by name, starting from the current working directory.
<DT>[*] By dir-relative path.<DD>
The pathname points to relative path and the dirfd indicates a file descriptor
pointing to a directory. The file is looked up by name, starting from the
directory specified by
<I>dirfd</I>.

<DT>[*] By file descriptor. The pathname is an empty string (&quot;&quot;), the dirfd<DD>
points directly to the mount object to move or the destination mount point and
the appropriate
<B>*_EMPTY_PATH</B>

flag is set.
</DL>
<P>

<I>flags</I>

can be used to influence a path-based lookup. The value for
<I>flags</I>

is constructed by OR'ing together zero or more of the following constants:
<DL COMPACT>
<DT><B>MOVE_MOUNT_F_EMPTY_PATH</B>

<DD>

If
<I>from_pathname</I>

is an empty string, operate on the file referred to by
<I>from_dirfd</I>

(which may have been obtained using the
<B><A HREF="https://man7.org/linux/man-pages/man2/open.2.html">open</A></B>(2)

<B>O_PATH</B>

flag or
<B>open_tree</B>())

If
<I>from_dirfd</I>

is
<B>AT_FDCWD</B>,

the call operates on the current working directory.
In this case,
<I>from_dirfd</I>

can refer to any type of file, not just a directory.
This flag is Linux-specific; define
<B>_GNU_SOURCE</B>


to obtain its definition.
<DT><B>MOVE_MOUNT_T_EMPTY_PATH</B>

<DD>
As above, but operating on
<I>to_pathname</I> and <I>to_dirfd</I>.

<DT><B>MOVE_MOUNT_F_AUTOMOUNTS</B>

<DD>
Don't automount the terminal (&quot;basename&quot;) component of
<I>from_pathname</I>

if it is a directory that is an automount point. This allows a mount object
that has an automount point at its root to be moved and prevents unintended
triggering of an automount point.
The
<B>MOVE_MOUNT_F_AUTOMOUNTS</B>

flag has no effect if the automount point has already been mounted over. This
flag is Linux-specific; define
<B>_GNU_SOURCE</B>


to obtain its definition.
<DT><B>MOVE_MOUNT_T_AUTOMOUNTS</B>

<DD>
As above, but operating on
<I>to_pathname</I> and <I>to_dirfd</I>.

This allows an automount point to be manually mounted over.
<DT><B>MOVE_MOUNT_F_SYMLINKS</B>

<DD>
If
<I>from_pathname</I>

is a symbolic link, then dereference it. The default for
<B>move_mount</B>()

is to not follow symlinks.
<DT><B>MOVE_MOUNT_T_SYMLINKS</B>

<DD>
As above, but operating on
<I>to_pathname</I> and <I>to_dirfd</I>.

</DL>
<A NAME="lbAE">&nbsp;</A>
<H2>RETURN VALUE</H2>

On success, 0 is returned. On error, -1 is returned, and
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
<I>from_dirfd</I> or <I>to_dirfd</I>

is not a valid open file descriptor.
<DT><B>EFAULT</B>

<DD>
<I>from_pathname</I> or <I>to_pathname</I>

is NULL or either one point to a location outside the process's accessible
address space.
<DT><B>EINVAL</B>

<DD>
Reserved flag specified in
<I>flags</I>.

<DT><B>ELOOP</B>

<DD>
Too many symbolic links encountered while traversing the pathname.
<DT><B>ENAMETOOLONG</B>

<DD>
<I>from_pathname</I> or <I>to_pathname</I>

is too long.
<DT><B>ENOENT</B>

<DD>
A component of
<I>from_pathname</I> or <I>to_pathname</I>

does not exist, or one is an empty string and the appropriate
<B>*_EMPTY_PATH</B>

was not specified in
<I>flags</I>.

<DT><B>ENOMEM</B>

<DD>
Out of memory (i.e., kernel memory).
<DT><B>ENOTDIR</B>

<DD>
A component of the path prefix of
<I>from_pathname</I> or <I>to_pathname</I>

is not a directory or one or the other is relative and the appropriate
<I>*_dirfd</I>

is a file descriptor referring to a file other than a directory.
</DL>
<A NAME="lbAG">&nbsp;</A>
<H2>VERSIONS</H2>

<B>move_mount</B>()

was added to Linux in kernel 5.2.
<A NAME="lbAH">&nbsp;</A>
<H2>CONFORMING TO</H2>

<B>move_mount</B>()

is Linux-specific.
<A NAME="lbAI">&nbsp;</A>
<H2>NOTES</H2>

Glibc does not (yet) provide a wrapper for the
<B>move_mount</B>()

system call; call it using
<B><A HREF="https://man7.org/linux/man-pages/man2/syscall.2.html">syscall</A></B>(2).

<A NAME="lbAJ">&nbsp;</A>
<H2>EXAMPLES</H2>

The
<B>move_mount</B>()

function can be used like the following:
<P>

<DL COMPACT><DT><DD>
<PRE>
move_mount(AT_FDCWD, &quot;/a&quot;, AT_FDCWD, &quot;/b&quot;, 0);
</PRE>

</DL>

<P>

This would move the object mounted on &quot;/a&quot; to &quot;/b&quot;. It can also be used in
conjunction with
<B><A HREF="https://man7.org/linux/man-pages/man2/open_tree.2.html">open_tree</A></B>(2) or <B><A HREF="https://man7.org/linux/man-pages/man2/open.2.html">open</A></B>(2) with <B>O_PATH</B>:

<P>

<DL COMPACT><DT><DD>
<PRE>
fd = open_tree(AT_FDCWD, &quot;/mnt&quot;, 0);
move_mount(fd, &quot;&quot;, AT_FDCWD, &quot;/mnt2&quot;, MOVE_MOUNT_F_EMPTY_PATH);
move_mount(fd, &quot;&quot;, AT_FDCWD, &quot;/mnt3&quot;, MOVE_MOUNT_F_EMPTY_PATH);
move_mount(fd, &quot;&quot;, AT_FDCWD, &quot;/mnt4&quot;, MOVE_MOUNT_F_EMPTY_PATH);
</PRE>

</DL>

<P>

This would attach the path point for &quot;/mnt&quot; to fd, then it would move the
mount to &quot;/mnt2&quot;, then move it to &quot;/mnt3&quot; and finally to &quot;/mnt4&quot;.
<P>

It can also be used to attach new mounts:
<P>

<DL COMPACT><DT><DD>
<PRE>
sfd = fsopen(&quot;ext4&quot;, FSOPEN_CLOEXEC);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;source&quot;, &quot;/dev/sda1&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;user_xattr&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_CMD_CREATE, NULL, NULL, 0);
mfd = fsmount(sfd, FSMOUNT_CLOEXEC, MOUNT_ATTR_NODEV);
move_mount(mfd, &quot;&quot;, AT_FDCWD, &quot;/home&quot;, MOVE_MOUNT_F_EMPTY_PATH);
</PRE>

</DL>

<P>

Which would open the Ext4 filesystem mounted on &quot;/dev/sda1&quot;, turn on user
extended attribute support and create a mount object for it. Finally, the new
mount object would be attached with
<B>move_mount</B>()

to &quot;/home&quot;.
<A NAME="lbAK">&nbsp;</A>
<H2>SEE ALSO</H2>

<B><A HREF="fsmount.md">fsmount</A></B>(2),

<B><A HREF="fsopen.md">fsopen</A></B>(2),

<B><A HREF="open_tree.md">open_tree</A></B>(2)

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
<DT><A HREF="#lbAJ">EXAMPLES</A><DD>
<DT><A HREF="#lbAK">SEE ALSO</A><DD>
</DL>
<HR>
This document was created by
<A HREF="http://primates.ximian.com/~flucifredi/man/">man2html</A>,
using the manual page posted
<A HREF="https://lwn.net/ml/linux-kernel/159827189025.306468.4916341547843731338.stgit@warthog.procyon.org.uk/">here</A>.
</BODY>
</HTML>
