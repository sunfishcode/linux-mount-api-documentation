<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>FSOPEN</H1>
Section: Linux Programmer's Manual (2)<BR>Updated: 2020-08-07<BR><A HREF="#index">Index</A>
<HR>

<A NAME="lbAB">&nbsp;</A>
<H2>NAME</H2>

fsopen, fsmount - Filesystem parameterisation and mount creation
<A NAME="lbAC">&nbsp;</A>
<H2>SYNOPSIS</H2>

<PRE>
<B>#include &lt;sys/types.h&gt;</B>
<B>#include &lt;sys/mount.h&gt;</B>
<B>#include &lt;unistd.h&gt;</B>
<B>#include &lt;fcntl.h&gt; </B>/* Definition of AT_* constants */

<B>int fsopen(const char *</B><I>fsname</I><B>, unsigned int </B><I>flags</I><B>);</B>

<B>int fsmount(int </B><I>fd</I><B>, unsigned int </B><I>flags</I><B>, unsigned int </B><I>mount_attrs</I><B>);</B>
</PRE>

<P>

<I>Note</I>:

There are no glibc wrappers for these system calls.
<A NAME="lbAD">&nbsp;</A>
<H2>DESCRIPTION</H2>

<P>

<B>fsopen</B>()

creates a blank filesystem configuration context within the kernel for the
filesystem named in the
<I>fsname</I>

parameter, puts it into creation mode and attaches it to a file descriptor,
which it then returns. The file descriptor can be marked close-on-exec by
setting
<B>FSOPEN_CLOEXEC</B>

in
<I>flags</I>.

<P>

After calling fsopen(), the file descriptor should be passed to the
<B><A HREF="fsconfig.md">fsconfig</A></B>(2)

system call, using that to specify the desired filesystem and security
parameters.
<P>

When the parameters are all set, the
<B>fsconfig</B>()

system call should then be called again with
<B>FSCONFIG_CMD_CREATE</B>

as the command argument to effect the creation.
<DL COMPACT><DT><DD>
<P>

<B>[!]&nbsp;NOTE</B>:

Depending on the filesystem type and parameters, this may rather share an
existing in-kernel filesystem representation instead of creating a new one.
In such a case, the parameters specified may be discarded or may overwrite the
parameters set by a previous mount - at the filesystem's discretion.
</DL>

<P>

The file descriptor also serves as a channel by which more comprehensive error,
warning and information messages may be retrieved from the kernel using
<B><A HREF="https://man7.org/linux/man-pages/man2/read.2.html">read</A></B>(2).

<P>

Once the creation command has been successfully run on a context, the context
will not accept further configuration. At
this point,
<B>fsmount</B>()

should be called to create a mount object.
<P>

<B>fsmount</B>()

takes the file descriptor returned by
<B>fsopen</B>()

and creates a mount object for the filesystem root specified there. The
attributes of the mount object are set from the
<I>mount_attrs</I>

parameter. The attributes specify the propagation and mount restrictions to
be applied to accesses through this mount.
<P>

The mount object is then attached to a new file descriptor that looks like one
created by
<B><A HREF="https://man7.org/linux/man-pages/man2/open.2.html">open</A></B>(2) with <B>O_PATH</B> or <B><A HREF="open_tree.md">open_tree</A></B>(2).

This can be passed to
<B><A HREF="move_mount.md">move_mount</A></B>(2)

to attach the mount object to a mountpoint, thereby completing the process.
<P>

The file descriptor returned by fsmount() is marked close-on-exec if
FSMOUNT_CLOEXEC is specified in
<I>flags</I>.

<P>

After fsmount() has completed, the context created by fsopen() is reset and
moved to reconfiguration state, allowing the new superblock to be
reconfigured. See
<B><A HREF="fspick.md">fspick</A></B>(2)

for details.
<P>

To use either of these calls, the caller requires the appropriate privilege
(Linux: the
<B>CAP_SYS_ADMIN</B>

capability).
<P>

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

Messages are removed from the queue as they're read.
<A NAME="lbAF">&nbsp;</A>
<H2>RETURN VALUE</H2>

On success, both functions return a file descriptor. On error, -1 is
returned, and
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
<DT><B>EBUSY</B>

<DD>
The context referred to by
<I>fd</I>

is not in the right state to be used by
<B>fsmount</B>().

<DT><B>EFAULT</B>

<DD>
One of the pointer arguments points outside the user address space.
<DT><B>EINVAL</B>

<DD>
<I>flags</I>

had an invalid flag set.
<DT><B>EINVAL</B>

<DD>
<I>mount_attrs,</I>

includes invalid
<B>MOUNT_ATTR_*</B>

flags.
<DT><B>EMFILE</B>

<DD>
The system has too many open files to create more.
<DT><B>ENFILE</B>

<DD>
The process has too many open files to create more.
<DT><B>ENODEV</B>

<DD>
The filesystem
<I>fsname</I>

is not available in the kernel.
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

<B>fsopen</B>(), and <B>fsmount</B>()

were added to Linux in kernel 5.2.
<A NAME="lbAJ">&nbsp;</A>
<H2>NOTES</H2>

Glibc does not (yet) provide a wrapper for the
<B>fsopen</B>() or <B>fsmount</B>()

system calls; call them using
<B><A HREF="https://man7.org/linux/man-pages/man2/syscall.2.html">syscall</A></B>(2).

<A NAME="lbAK">&nbsp;</A>
<H2>EXAMPLES</H2>

To illustrate the process, here's an example whereby this can be used to mount
an ext4 filesystem on /dev/sdb1 onto /mnt.
<P>


<PRE>
sfd = fsopen(&quot;ext4&quot;, FSOPEN_CLOEXEC);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;ro&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;source&quot;, &quot;/dev/sdb1&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;noatime&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;acl&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;user_attr&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;iversion&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_CMD_CREATE, NULL, NULL, 0);
mfd = fsmount(sfd, FSMOUNT_CLOEXEC, MS_RELATIME);
move_mount(mfd, &quot;&quot;, AT_FDCWD, &quot;/mnt&quot;, MOVE_MOUNT_F_EMPTY_PATH);
</PRE>


<P>

Here, an ext4 context is created first and attached to sfd. The context is
then told where its source will be, given a bunch of options and a superblock
record object is then created. Then fsmount() is called to create a mount
object and
<B><A HREF="move_mount.md">move_mount</A></B>(2)

is called to attach it to its intended mountpoint.
<P>

And here's an example of mounting from an NFS server and setting a Smack
security module label on it too:
<P>


<PRE>
sfd = fsopen(&quot;nfs&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;source&quot;, &quot;example.com:/pub&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;nfsvers&quot;, &quot;3&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;rsize&quot;, &quot;65536&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;wsize&quot;, &quot;65536&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_STRING, &quot;smackfsdef&quot;, &quot;foolabel&quot;, 0);
fsconfig(sfd, FSCONFIG_SET_FLAG, &quot;rdma&quot;, NULL, 0);
fsconfig(sfd, FSCONFIG_CMD_CREATE, NULL, NULL, 0);
mfd = fsmount(sfd, 0, MS_NODEV);
move_mount(mfd, &quot;&quot;, AT_FDCWD, &quot;/mnt&quot;, MOVE_MOUNT_F_EMPTY_PATH);
</PRE>


<P>

<A NAME="lbAL">&nbsp;</A>
<H2>SEE ALSO</H2>

<B><A HREF="https://man7.org/linux/man-pages/man1/mountpoint.1.html">mountpoint</A></B>(1),

<B><A HREF="fsconfig.md">fsconfig</A></B>(2),

<B><A HREF="fspick.md">fspick</A></B>(2),

<B><A HREF="move_mount.md">move_mount</A></B>(2),

<B><A HREF="open_tree.md">open_tree</A></B>(2),

<B><A HREF="https://man7.org/linux/man-pages/man2/umount.2.html">umount</A></B>(2),

<B><A HREF="https://man7.org/linux/man-pages/man7/mount_namespaces.7.html">mount_namespaces</A></B>(7),

<B><A HREF="https://man7.org/linux/man-pages/man7/path_resolution.7.html">path_resolution</A></B>(7),

<B><A HREF="https://man7.org/linux/man-pages/man8/mount.8.html">mount</A></B>(8),

<B><A HREF="https://man7.org/linux/man-pages/man8/umount.8.html">umount</A></B>(8)


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
<DT><A HREF="#lbAJ">NOTES</A><DD>
<DT><A HREF="#lbAK">EXAMPLES</A><DD>
<DT><A HREF="#lbAL">SEE ALSO</A><DD>
</DL>
<HR>
This document was created by
<A HREF="http://primates.ximian.com/~flucifredi/man/">man2html</A>,
using the manual page posted
<A HREF="https://lwn.net/ml/linux-kernel/159827190508.306468.12755090833140558156.stgit@warthog.procyon.org.uk/">here</A>.
</BODY>
</HTML>
