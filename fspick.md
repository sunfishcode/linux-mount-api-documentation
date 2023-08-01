<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>FSPICK</H1>
Section: Linux Programmer's Manual (2)<BR>Updated: 2020-08-24<BR><A HREF="#index">Index</A>
<HR>

<A NAME="lbAB">&nbsp;</A>
<H2>NAME</H2>

fspick - Select filesystem for reconfiguration
<A NAME="lbAC">&nbsp;</A>
<H2>SYNOPSIS</H2>

<PRE>
<B>#include &lt;sys/types.h&gt;</B>
<B>#include &lt;sys/mount.h&gt;</B>
<B>#include &lt;unistd.h&gt;</B>
<B>#include &lt;fcntl.h&gt; </B>/* Definition of AT_* constants */

<B>int fspick(int </B><I>dirfd</I><B>, const char *</B><I>pathname</I><B>, unsigned int </B><I>flags</I><B>);</B>
</PRE>

<P>

<I>Note</I>:

There is no glibc wrapper for this system call.
<A NAME="lbAD">&nbsp;</A>
<H2>DESCRIPTION</H2>

<P>

<B>fspick</B>()

creates a new filesystem configuration context within the kernel and attaches a
pre-existing superblock to it so that it can be reconfigured (similar to
<B><A HREF="https://man7.org/linux/man-pages/man8/mount.8.html">mount</A></B>(8)

with the &quot;-o remount&quot; option). The configuration context is marked as being in
reconfiguration mode and attached to a file descriptor, which is returned to
the caller. The file descriptor can be marked close-on-exec by setting
<B>FSPICK_CLOEXEC</B>

in
<I>flags</I>.

<P>

The target is whichever superblock backs the object determined by
<I>dfd</I>, <I>pathname</I> and <I>flags</I>.

The following can be set in
<I>flags</I>

to control the pathwalk to that object:
<DL COMPACT>
<DT><B>FSPICK_SYMLINK_NOFOLLOW</B>

<DD>
Don't follow symbolic links in the final component of the path.
<DT><B>FSPICK_NO_AUTOMOUNT</B>

<DD>
Don't follow automounts in the final component of the path.
<DT><B>FSPICK_EMPTY_PATH</B>

<DD>
Allow an empty string to be specified as the pathname. This allows
<I>dirfd</I>

to specify the target mount exactly.
</DL>
<P>

After calling fspick(), the file descriptor should be passed to the
<B><A HREF="fsconfig.md">fsconfig</A></B>(2)

system call, using that to specify the desired changes to filesystem and
security parameters.
<P>

When the parameters are all set, the
<B>fsconfig</B>()

system call should then be called again with
<B>FSCONFIG_CMD_RECONFIGURE</B>

as the command argument to effect the reconfiguration.
<P>

After the reconfiguration has taken place, the context is wiped clean (apart
from the superblock attachment, which remains) and can be reused to make
another reconfiguration.
<P>

The file descriptor also serves as a channel by which more comprehensive error,
warning and information messages may be retrieved from the kernel using
<B><A HREF="https://man7.org/linux/man-pages/man2/read.2.html">read</A></B>(2).

<A NAME="lbAE">&nbsp;</A>
<H3>Message Retrieval Interface</H3>

The context file descriptor may be queried for message strings at any time by
calling
<B><A HREF="https://man7.org/linux/man-pages/man2/read.2.html">read</A></B>(2)

on the file descriptor. This will return formatted messages that are prefixed
to indicate their class:
<DL COMPACT>
<DT><B>&quot;e &lt;message&gt;&quot;</B><DD>
An error message string was logged.
<DT><B>&quot;i &lt;message&gt;&quot;</B><DD>
An informational message string was logged.
<DT><B>&quot;w &lt;message&gt;&quot;</B><DD>
An warning message string was logged.
</DL>
<P>

Messages are removed from the queue as they're read and the queue has a limited
depth of 8 messages, so it's possible for some to get lost.
<A NAME="lbAF">&nbsp;</A>
<H2>RETURN VALUE</H2>

On success, the function returns a file descriptor. On error, -1 is returned,
and
<I>errno</I>

is set appropriately.
<A NAME="lbAG">&nbsp;</A>
<H2>ERRORS</H2>

The error values given below result from filesystem type independent errors.
Additionally, each filesystem type may have its own special errors and its own
special behavior. See the Linux kernel source code for details.
<DL COMPACT>
<DT><B>EACCES</B>

<DD>
A component of a path was not searchable.
(See also
<B><A HREF="https://man7.org/linux/man-pages/man7/path_resolution.7.html">path_resolution</A></B>(7).)

<DT><B>EFAULT</B>

<DD>
<I>pathname</I>

points outside the user address space.
<DT><B>EINVAL</B>

<DD>
<I>flags</I>

includes an undefined value.
<DT><B>ELOOP</B>

<DD>
Too many links encountered during pathname resolution.
<DT><B>EMFILE</B>

<DD>
The system has too many open files to create more.
<DT><B>ENFILE</B>

<DD>
The process has too many open files to create more.
<DT><B>ENAMETOOLONG</B>

<DD>
A pathname was longer than
<B>MAXPATHLEN</B>.

<DT><B>ENOENT</B>

<DD>
A pathname was empty or had a nonexistent component.
<DT><B>ENOMEM</B>

<DD>
The kernel could not allocate sufficient memory to complete the call.
<DT><B>EPERM</B>

<DD>
The caller does not have the required privileges.
</DL>
<A NAME="lbAH">&nbsp;</A>
<H2>CONFORMING TO</H2>

These functions are Linux-specific and should not be used in programs intended
to be portable.
<A NAME="lbAI">&nbsp;</A>
<H2>VERSIONS</H2>

<B>fsopen</B>(), <B>fsmount</B>() and <B>fspick</B>()

were added to Linux in kernel 5.2.
<A NAME="lbAJ">&nbsp;</A>
<H2>EXAMPLES</H2>

To illustrate the process, here's an example whereby this can be used to
reconfigure a filesystem:
<P>


<PRE>
sfd = fspick(AT_FDCWD, &quot;/mnt&quot;, FSPICK_NO_AUTOMOUNT | FSPICK_CLOEXEC);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;ro&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;user_xattr&quot;, &quot;false&quot;, 0);
fsconfig(sfd, FSCONFIG_CMD_RECONFIGURE, NULL, NULL, 0);
</PRE>


<P>

<A NAME="lbAK">&nbsp;</A>
<H2>NOTES</H2>

Glibc does not (yet) provide a wrapper for the
<B>fspick</B>()

system call; call it using
<B><A HREF="https://man7.org/linux/man-pages/man2/syscall.2.html">syscall</A></B>(2).

<A NAME="lbAL">&nbsp;</A>
<H2>SEE ALSO</H2>

<B><A HREF="https://man7.org/linux/man-pages/man1/mountpoint.1.html">mountpoint</A></B>(1),

<B><A HREF="fsconfig.md">fsconfig</A></B>(2),

<B><A HREF="fsopen.md">fsopen</A></B>(2),

<B><A HREF="https://man7.org/linux/man-pages/man7/path_resolution.7.html">path_resolution</A></B>(7),

<B><A HREF="https://man7.org/linux/man-pages/man8/mount.8.html">mount</A></B>(8)

<P>

<HR>
<A NAME="index">&nbsp;</A><H2>Index</H2>
<DL>
<DT><A HREF="#lbAB">NAME</A><DD>
<DT><A HREF="#lbAC">SYNOPSIS</A><DD>
<DT><A HREF="#lbAD">DESCRIPTION</A><DD>
<DL>
<DT><A HREF="#lbAE">Message Retrieval Interface</A><DD>
</DL>
<DT><A HREF="#lbAF">RETURN VALUE</A><DD>
<DT><A HREF="#lbAG">ERRORS</A><DD>
<DT><A HREF="#lbAH">CONFORMING TO</A><DD>
<DT><A HREF="#lbAI">VERSIONS</A><DD>
<DT><A HREF="#lbAJ">EXAMPLES</A><DD>
<DT><A HREF="#lbAK">NOTES</A><DD>
<DT><A HREF="#lbAL">SEE ALSO</A><DD>
</DL>
<HR>
This document was created by
<A HREF="http://primates.ximian.com/~flucifredi/man/">man2html</A>,
using the manual page posted
<A HREF="https://lwn.net/ml/linux-kernel/159827189767.306468.1803062787718957199.stgit@warthog.procyon.org.uk/">here</A>.
</BODY>
</HTML>
