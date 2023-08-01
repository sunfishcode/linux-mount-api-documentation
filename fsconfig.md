<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>FSCONFIG</H1>
Section: Linux Programmer's Manual (2)<BR>Updated: 2020-08-24<BR><A HREF="#index">Index</A>
<HR>

<A NAME="lbAB">&nbsp;</A>
<H2>NAME</H2>

fsconfig - Filesystem parameterisation
<A NAME="lbAC">&nbsp;</A>
<H2>SYNOPSIS</H2>

<PRE>
<B>#include &lt;sys/types.h&gt;</B>
<B>#include &lt;sys/mount.h&gt;</B>
<B>#include &lt;unistd.h&gt;</B>
<B>#include &lt;sys/mount.h&gt;</B>

<B>int fsconfig(int *</B><I>fd</I><B>, unsigned int </B><I>cmd</I><B>, const char *</B><I>key</I><B>,</B>
<BR>
<B> const void __user *</B><I>value</I><B>, int </B><I>aux</I><B>);</B>
<BR>
</PRE>

<P>

<I>Note</I>:

There is no glibc wrapper for this system call.
<A NAME="lbAD">&nbsp;</A>
<H2>DESCRIPTION</H2>

<P>

<B>fsconfig</B>()

is used to supply parameters to and issue commands against a filesystem
configuration context as set up by
<B><A HREF="fsopen.md">fsopen</A></B>(2)

or
<B><A HREF="fspick.md">fspick</A></B>(2).

The context is supplied attached to the file descriptor specified by
<I>fd</I>

argument.
<P>

The
<I>cmd</I>

argument indicates the command to be issued, where some of the commands simply
supply parameters to the context. The meaning of
<I>key</I>, <I>value</I> and <I>aux</I>

are command-dependent; unless required for the command, these should be set to
NULL or 0.
<P>

The available commands are:
<DL COMPACT>
<DT><B>FSCONFIG_SET_FLAG</B>

<DD>
Set the parameter named by
<I>key</I>

to true. This may fail with error
<B>EINVAL</B>

if the parameter requires an argument.
<DT><B>FSCONFIG_SET_STRING</B>

<DD>
Set the parameter named by
<I>key</I>

to a string. This may fail with error
<B>EINVAL</B>

if the parser doesn't want a parameter here, wants a non-string or the string
cannot be interpreted appropriately.
<I>value</I>

points to a NUL-terminated string.
<DT><B>FSCONFIG_SET_BINARY</B>

<DD>
Set the parameter named by
<I>key</I>

to be a binary blob argument. This may cause
<B>EINVAL</B>

to be returned if the filesystem parser isn't expecting a binary blob and it
can't be converted to something usable.
<I>value</I>

points to the data and
<I>aux</I>

indicates the size of the data.
<DT><B>FSCONFIG_SET_PATH</B>

<DD>
Set the parameter named by
<I>key</I>

to the object at the provided path.
<I>value</I>

should point to a NUL-terminated pathname string and aux may indicate
<B>AT_FDCWD</B>

or a file descriptor indicating a directory from which to begin a relative
path resolution. This may fail with error
<B>EINVAL</B>

if the parameter isn't expecting a path; it may also fail if the path cannot
be resolved with the typcal errors for that
(<B>ENOENT</B>, <B>ENOTDIR</B>, <B>EPERM</B>, <B>EACCES</B>, etc.).

<DT><DD>
Note that FSCONFIG_SET_STRING can be used instead, implying AT_FDCWD.
<DT><B>FSCONFIG_SET_PATH_EMPTY</B>

<DD>
As FSCONFIG_SET_PATH, but with
<B>AT_EMPTY_PATH</B>

applied to the pathwalk.
<DT><B>FSCONFIG_SET_FD</B>

<DD>
Set the parameter named by
<I>key</I>

to the file descriptor specified by
<I>aux</I>.

This will fail with
<B>EINVAL</B>

if the parameter doesn't expect a file descriptor or
<B>EBADF</B>

if the file descriptor is invalid.
<DT><DD>
Note that FSCONFIG_SET_STRING can be used instead with the file descriptor
passed as a decimal string.
<DT><B>FSCONFIG_CMD_CREATE</B>

<DD>
This command triggers the filesystem to take the parameters set in the context
and to try to create filesystem representation in the kernel. If an existing
representation can be shared, the filesystem may do that instead if the
parameters permit. This is intended for use with
<B><A HREF="fsopen.md">fsopen</A></B>(2).

<DT><B>FSCONFIG_CMD_RECONFIGURE</B>

<DD>
This command causes the driver to alter the parameters of an already live
filesystem instance according to the parameters stored in the context. This
is intended for use with
<B><A HREF="fspick.md">fspick</A></B>(2),

but may also by used against the context created by
<B>fsopen()</B>

after
<B><A HREF="fsmount.md">fsmount</A></B>(2)

has been called on it.
<P>

</DL>
<A NAME="lbAE">&nbsp;</A>
<H2>EXAMPLES</H2>

<P>


<PRE>
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;ro&quot;, NULL, 0);

fsconfig(sfd, FSCONFIG_SET_STRING, &quot;user_xattr&quot;, &quot;false&quot;, 0);

fsconfig(sfd, FSCONFIG_SET_BINARY, &quot;ms_pac&quot;, pac_buffer, pac_size);

fsconfig(sfd, FSCONFIG_SET_PATH, &quot;journal&quot;, &quot;/dev/sdd4&quot;, AT_FDCWD);

dirfd = open(&quot;/dev/&quot;, O_PATH);
fsconfig(sfd, FSCONFIG_SET_PATH, &quot;journal&quot;, &quot;sdd4&quot;, dirfd);

fd = open(&quot;/overlays/mine/&quot;, O_PATH);
fsconfig(sfd, FSCONFIG_SET_PATH_EMPTY, &quot;lower_dir&quot;, &quot;&quot;, fd);

pipe(pipefds);
fsconfig(sfd, FSCONFIG_SET_FD, &quot;fd&quot;, NULL, pipefds[1]);
</PRE>


<P>

<A NAME="lbAF">&nbsp;</A>
<H2>RETURN VALUE</H2>

On success, the function returns 0. On error, -1 is returned, and
<I>errno</I>

is set appropriately.
<A NAME="lbAG">&nbsp;</A>
<H2>ERRORS</H2>

The error values given below result from filesystem type independent
errors.
Each filesystem type may have its own special errors and its
own special behavior.
See the Linux kernel source code for details.
<DL COMPACT>
<DT><B>EACCES</B>

<DD>
A component of a path was not searchable.
(See also
<B><A HREF="https://man7.org/linux/man-pages/man7/path_resolution.7.html">path_resolution</A></B>(7).)

<DT><B>EACCES</B>

<DD>
Mounting a read-only filesystem was attempted without specifying the
'<B>ro</B>'

parameter.
<DT><B>EACCES</B>

<DD>
A specified block device is located on a filesystem mounted with the
<B>MS_NODEV</B>

option.


<DT><B>EBADF</B>

<DD>
The file descriptor given by
<I>fd</I>

or possibly by
<I>aux</I>

(depending on the command) is invalid.
<DT><B>EBUSY</B>

<DD>
The context attached to
<I>fd</I>

is in the wrong state for the given command.
<DT><B>EBUSY</B>

<DD>
The filesystem representation cannot be reconfigured read-only because it still
holds files open for writing.
<DT><B>EFAULT</B>

<DD>
One of the pointer arguments points outside the accessible address space.
<DT><B>EINVAL</B>

<DD>
<I>fd</I>

does not refer to a filesystem configuration context.
<DT><B>EINVAL</B>

<DD>
One of the source parameters referred to an invalid superblock.
<DT><B>ELOOP</B>

<DD>
Too many links encountered during pathname resolution.
<DT><B>ENAMETOOLONG</B>

<DD>
A path name was longer than
<B>MAXPATHLEN</B>.

<DT><B>ENOENT</B>

<DD>
A pathname was empty or had a nonexistent component.
<DT><B>ENOMEM</B>

<DD>
The kernel could not allocate sufficient memory to complete the call.
<DT><B>ENOTBLK</B>

<DD>
Once of the parameters does not refer to a block device (and a device was
required).
<DT><B>ENOTDIR</B>

<DD>
<I>pathname</I>,

or a prefix of
<I>source</I>,

is not a directory.
<DT><B>EOPNOTSUPP</B>

<DD>
The command given by
<I>cmd</I>

was not valid.
<DT><B>ENXIO</B>

<DD>
The major number of a block device parameter is out of range.
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

<B>fsconfig</B>()

was added to Linux in kernel 5.2.
<A NAME="lbAJ">&nbsp;</A>
<H2>NOTES</H2>

Glibc does not (yet) provide a wrapper for the
<B>fsconfig</B>()

system call; call it using
<B><A HREF="https://man7.org/linux/man-pages/man2/syscall.2.html">syscall</A></B>(2).

<A NAME="lbAK">&nbsp;</A>
<H2>SEE ALSO</H2>

<B><A HREF="https://man7.org/linux/man-pages/man1/mountpoint.1.html">mountpoint</A></B>(1),

<B><A HREF="fsmount.md">fsmount</A></B>(2),

<B><A HREF="fsopen.md">fsopen</A></B>(2),

<B><A HREF="fspick.md">fspick</A></B>(2),

<B><A HREF="https://man7.org/linux/man-pages/man7/mount_namespaces.7.html">mount_namespaces</A></B>(7),

<B><A HREF="https://man7.org/linux/man-pages/man7/path_resolution.7.html">path_resolution</A></B>(7)

<P>

<HR>
<A NAME="index">&nbsp;</A><H2>Index</H2>
<DL>
<DT><A HREF="#lbAB">NAME</A><DD>
<DT><A HREF="#lbAC">SYNOPSIS</A><DD>
<DT><A HREF="#lbAD">DESCRIPTION</A><DD>
<DT><A HREF="#lbAE">EXAMPLES</A><DD>
<DT><A HREF="#lbAF">RETURN VALUE</A><DD>
<DT><A HREF="#lbAG">ERRORS</A><DD>
<DT><A HREF="#lbAH">CONFORMING TO</A><DD>
<DT><A HREF="#lbAI">VERSIONS</A><DD>
<DT><A HREF="#lbAJ">NOTES</A><DD>
<DT><A HREF="#lbAK">SEE ALSO</A><DD>
</DL>
<HR>
This document was created by
<A HREF="http://primates.ximian.com/~flucifredi/man/">man2html</A>,
using the manual page posted
<A HREF="https://lwn.net/ml/linux-kernel/159827191245.306468.4903071494263813779.stgit@warthog.procyon.org.uk/">here</A>.
</BODY>
</HTML>
