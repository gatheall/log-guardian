<HTML>
<HEAD>
<TITLE>log-guardian</TITLE>
<LINK REV="made" HREF="mailto:theall@tifaware.com">
<LINK REL="shortcut icon" HREF="http://www.tifaware.com/favicon.ico">
</HEAD>

<!--#include virtual="/header.html"-->
<!--#if expr="$weblint" -->
<BODY BGCOLOR="#FFFFFF">
<!--#endif -->


<H4 ALIGN="center">
   <A HREF="/">TifaWARE</A> |
   <A HREF="../">Perl Programs</A> |
   log-guardian
</H4>
                                                                                
                                                                                
<HR>
<H2>log-guardian</H2>

<H3>Introduction</H3>

<P>This script lets you monitor one or more log files in an endless
loop, <I>a la</I> <CODE>tail -f</CODE>.  As lines are added to the
files, they are compared to one or more patterns specified as Perl
regular expressions.  And as matches are found, the script reacts by
running a block of Perl code.  Thus, for example, you could use
<B>log-guardian</B> to monitor web logs for problematic behaviour and
add troublesome hosts to a blocklist dynamically.  You could even use it
as a <A HREF="http://slashdot.org/articles/04/02/05/1834228.shtml">port
knocking</A> server!</P>

<P><B>log-guardian</B> is not a general-purpose log analyzer, though,
since it looks at lines only as they are added to logs, not at whole log
files themselves.  It's intended simply to allow you to react to issues
as they arise.</P>

<P>By virtue of its use of the Perl module <CODE>File::Tail</CODE>,
<B>log-guardian</B> offers a number of advantages.  For one, it's
unaffected by log file rotation -- if a file does not appear to have
input for a period of time, <CODE>File::Tail</CODE> will quietly re-open
the file and continue reading.  For another, it does not spend excessive
time checking log files with little or no traffic since
<CODE>File::Tail</CODE> adjusts how frequently it polls for input based
on past history.</P>

<P>You control what is monitored and how by modifying the scalar
<CODE>$monitors</CODE>, either in the script itself or in a separate
config file.  The scalar should be defined as a hash.  Each key is a log
file to monitor while the value is an array of hashes specifying what to
look for in the log file and how to react.  Keys in these secondary
hashes should be either <CODE>label</CODE>, <CODE>pattern</CODE>, or
<CODE>action</CODE>, corresponding to values representing respectively a
descriptive label of the monitor, a Perl regular expression to use as a
pattern, and an anonymous subroutine to be run if a match occurs. 
<CODE>pattern</CODE> is required while the other two key / value pairs
are optional.  If <CODE>action</CODE> is not present, the default action
in <CODE>$default_action</CODE> will be used instead.</P>

<P>When taking an action, <B>log-guardian</B> passes the subroutine
several arguments: name of log, contents of line, value of
<CODE>label</CODE>, value of <CODE>pattern</CODE>, and the results of
matching the line against the pattern in an array contents (eg,
<CODE>$1</CODE>, <CODE>$2</CODE>, etc).  These may be used however you
want.</P>

<P><B>log-guardian</B> is written in Perl.  It should work on any
unix-like system with Perl 5.003 or later.  It also requires the
following Perl modules:</P>

<UL>
   <LI><CODE>Carp</CODE></LI>
   <LI><CODE>Getopt::Long</CODE></LI>
   <LI><CODE>File::Tail</CODE></LI>
   <LI><CODE>Safe</CODE></LI>
</UL>

<P>If your system does not have these modules installed already, visit
<A HREF="http://search.cpan.org/">CPAN</A> for help.  Note that
<CODE>File::Tail</CODE> must be at least version 0.90 and
<CODE>Safe</CODE> at least version 2.0 (thus, it will not work with
versions of Perl older than 5.003).  Note also that
<CODE>File::Tail</CODE> is not included in the default Perl distribution
so you may need to install it yourself.</P>


<H3>Installation</H3>

<OL>
   <LI>Retrieve the script <A HREF="/code/log-guardian/log-guardian">log-guardian</A>
      and save it locally.  You may also wish to verify its
      <A HREF="/code/log-guardian/log-guardian.md5">MD5 checksum</A> or, better, its
      <A HREF="/code/log-guardian/log-guardian.asc">GPG signature</A> against
      <A HREF="/~theall/gpg.html">my current GPG key</A>.</LI>

   <LI>Verify ownership and permissions on the script - it will need to
      be invoked by root.</LI>

   <LI>Edit the script and set <CODE>$ENV{PATH}</CODE> and 
      <CODE>$monitors</CODE> according to your environment. You may also 
      wish to adjust the location of the perl interpreter in the first 
      line as well as <CODE>$default_action</CODE>, 
      <CODE>$max_interval</CODE>, and <CODE>$select_timeout</CODE> to 
      suit your tastes.</LI>
</OL>


<H3>Use</H3>

<P>If you have just a few logs to monitor, you may wish to run only one
instance of <B>log-guardian</B>, starting it, say, when the system boots
and putting it in the background.  Alternatively, if you have high
volume services, you may find associating one instance with each of
those services improves responsiveness and / or maintainability.  In
this case, you might find it best to invoke <B>log-guardian</B> as part
of each service's startup script.  Regardless, understand that the
script will run in an endless loop and that, if you put it in the
background, you will need to redirect output to a file somewhere in
order to see it.</P>

<P>Examples:</P>
<BLOCKQUOTE>
<DL>
   <DT><CODE>log-guardian</CODE></DT>
   <DD>monitors logfiles.</DD>

   <DT><CODE>log-guardian -d</CODE></DT>
   <DD>same as above but with copious debugging messages.</DD>

   <DT><CODE>log-guardian /etc/log-guardian/weblogs</CODE></DT>
   <DD>monitors logfiles using settings in 
      <CODE>/etc/log-guardian/weblogs</CODE>.</DD>
</DL>
</BLOCKQUOTE>


<H3>Known Bugs and Caveats</H3>

<P>Currently, I am not aware of any bugs in this script.</P>

<P>Understand that actions undertaken by <B>log-guardian</B> are
arbitrary Perl code.  Be careful to control on one hand access to that
code and on the other the content of that code.  And read <A
HREF="http://www.ossec.net/en/attacking-loganalysis.html">Daniel Cid's
discussion of Attacking Log analysis tools</A> to understand some of the
pitfalls that can arise.</P>

<P>If you encounter an error saying something like <CODE>Can't parse
'<I>file</I>' - '<I>function</I>' trapped by operation mask</CODE>, you
will need to adjust the list of operators permitted by
<CODE>Safe</CODE>.  Look for the line with
<CODE>$sandbox-&gt;permit_only</CODE> and refer to the manpage for
<CODE>Opcode</CODE> for possible operators.</P>

<P>You must include a pathname when specifying a separate configuration
file; otherwise, it will be silently ignored.</P>

<P>When making changes to <CODE>$monitors</CODE>, you would be wise to
redirect the script's output to a file for a period of time.  This will
help diagnose problems in the patterns and / or actions.</P>


<H3>Copyright and License</H3>

<P>Copyright (c) 2004-2007, George A. Theall.<BR>
All rights reserved.</P>

<P>This script is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.</P>


<H3>History</H3>

<CENTER>
<TABLE CELLPADDING="5" CELLSPACING="0" WIDTH="95%" BORDER="1">
   <TR>
      <TH ALIGN="left">Date </TH>
      <TH ALIGN="left">Version </TH>
      <TH ALIGN="left">Verification </TH>
      <TH ALIGN="left">Notes<BR></TH>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">17-May-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/log-guardian/log-guardian-1.01">1.01</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/log-guardian/log-guardian-1.01.md5">MD5</A> / 
         <A HREF="/code/log-guardian/log-guardian-1.01.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Added a configuration variable (<CODE>$max_interval</CODE>) to 
            control the maximum number of seconds to sleep between checks 
            of logs.</LI>
         <LI>Changed <CODE>$select_timeout</CODE> to have global scope 
            and lowered its default value.</LI>
      </UL></TD>
   </TR>
   <TR>
      <TH ALIGN="left" VALIGN="top">04-May-2004 </TH>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/log-guardian/log-guardian-1.00">1.00</A> </TD>
      <TD ALIGN="left" VALIGN="top"><A HREF="/code/log-guardian/log-guardian-1.00.md5">MD5</A> / 
         <A HREF="/code/log-guardian/log-guardian-1.00.asc">GPG</A> </TD>
      <TD ALIGN="left" VALIGN="top">
      <UL>
         <LI>Initial version.</LI>
      </UL></TD>
   </TR>
</TABLE>
</CENTER>


<HR>
<H4 ALIGN="center">
   <A HREF="/">TifaWARE</A> |
   <A HREF="../">Perl Programs</A> |
   log-guardian
</H4>

<!-- $Id$ -->

<!--#include virtual="/footer.html" -->
<!--#if expr="$weblint" -->
</BODY>
<!--#endif -->
</HTML>
