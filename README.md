# Makefile Standard Targets

<a name="Standard-Targets"></a>

<div class="header">

Next: [Install Command Categories](Install-Command-Categories.html#Install-Command-Categories), Previous: [Directory Variables](Directory-Variables.html#Directory-Variables), Up: [Makefile Conventions](Makefile-Conventions.html#Makefile-Conventions)   [[Contents](index.html#SEC_Contents "Table of contents")][[Index](Index.html#Index "Index")]

</div>

* * *

<a name="Standard-Targets-for-Users"></a>

#### 7.2.6 Standard Targets for Users

All GNU programs should have the following targets in their Makefiles:

<dl compact="compact">

<dt>‘<samp>all</samp>’</dt>

<dd>

Compile the entire program. This should be the default target. This target need not rebuild any documentation files; Info files should normally be included in the distribution, and DVI (and other documentation format) files should be made only when explicitly asked for.

By default, the Make rules should compile and link with ‘<samp>-g</samp>’, so that executable programs have debugging symbols. Otherwise, you are essentially helpless in the face of a crash, and it is often far from easy to reproduce with a fresh build.

</dd>

<dt>‘<samp>install</samp>’</dt>

<dd>

Compile the program and copy the executables, libraries, and so on to the file names where they should reside for actual use. If there is a simple test to verify that a program is properly installed, this target should run that test.

Do not strip executables when installing them. This helps eventual debugging that may be needed later, and nowadays disk space is cheap and dynamic loaders typically ensure debug sections are not loaded during normal execution. Users that need stripped binaries may invoke the `install-strip` target to do that.

If possible, write the `install` target rule so that it does not modify anything in the directory where the program was built, provided ‘<samp>make all</samp>’ has just been done. This is convenient for building the program under one user name and installing it under another.

The commands should create all the directories in which files are to be installed, if they don’t already exist. This includes the directories specified as the values of the variables `prefix` and `exec_prefix`, as well as all subdirectories that are needed. One way to do this is by means of an `installdirs` target as described below.

Use ‘<samp>-</samp>’ before any command for installing a man page, so that `make` will ignore any errors. This is in case there are systems that don’t have the Unix man page documentation system installed.

The way to install Info files is to copy them into <samp>$(infodir)</samp> with `$(INSTALL_DATA)` (see [Command Variables](Command-Variables.html#Command-Variables)), and then run the `install-info` program if it is present. `install-info` is a program that edits the Info <samp>dir</samp> file to add or update the menu entry for the given Info file; it is part of the Texinfo package.

Here is a sample rule to install an Info file that also tries to handle some additional situations, such as `install-info` not being present.

<div class="smallexample">

<pre class="smallexample">do-install-info: foo.info installdirs
        $(NORMAL_INSTALL)
# Prefer an info file in . to one in srcdir.
        if test -f foo.info; then d=.; \
         else d="$(srcdir)"; fi; \
        $(INSTALL_DATA) $d/foo.info \
          "$(DESTDIR)$(infodir)/foo.info"
# Run install-info only if it exists.
# Use 'if' instead of just prepending '-' to the
# line so we notice real errors from install-info.
# Use '$(SHELL) -c' because some shells do not
# fail gracefully when there is an unknown command.
        $(POST_INSTALL)
        if $(SHELL) -c 'install-info --version' \
           >/dev/null 2>&1; then \
          install-info --dir-file="$(DESTDIR)$(infodir)/dir" \
                       "$(DESTDIR)$(infodir)/foo.info"; \
        else true; fi
</pre>

</div>

When writing the `install` target, you must classify all the commands into three categories: normal ones, _pre-installation_ commands and _post-installation_ commands. See [Install Command Categories](Install-Command-Categories.html#Install-Command-Categories).

</dd>

<dt>‘<samp>install-html</samp>’</dt>

<dt>‘<samp>install-dvi</samp>’</dt>

<dt>‘<samp>install-pdf</samp>’</dt>

<dt>‘<samp>install-ps</samp>’</dt>

<dd>

These targets install documentation in formats other than Info; they’re intended to be called explicitly by the person installing the package, if that format is desired. GNU prefers Info files, so these must be installed by the `install` target.

When you have many documentation files to install, we recommend that you avoid collisions and clutter by arranging for these targets to install in subdirectories of the appropriate installation directory, such as `htmldir`. As one example, if your package has multiple manuals, and you wish to install HTML documentation with many files (such as the “split” mode output by `makeinfo --html`), you’ll certainly want to use subdirectories, or two nodes with the same name in different manuals will overwrite each other.

Please make these `install-<var>format</var>` targets invoke the commands for the <var>format</var> target, for example, by making <var>format</var> a dependency.

</dd>

<dt>‘<samp>uninstall</samp>’</dt>

<dd>

Delete all the installed files—the copies that the ‘<samp>install</samp>’ and ‘<samp>install-*</samp>’ targets create.

This rule should not modify the directories where compilation is done, only the directories where files are installed.

The uninstallation commands are divided into three categories, just like the installation commands. See [Install Command Categories](Install-Command-Categories.html#Install-Command-Categories).

</dd>

<dt>‘<samp>install-strip</samp>’</dt>

<dd>

Like `install`, but strip the executable files while installing them. In simple cases, this target can use the `install` target in a simple way:

<div class="smallexample">

<pre class="smallexample">install-strip:
        $(MAKE) INSTALL_PROGRAM='$(INSTALL_PROGRAM) -s' \
                install
</pre>

</div>

But if the package installs scripts as well as real executables, the `install-strip` target can’t just refer to the `install` target; it has to strip the executables but not the scripts.

`install-strip` should not strip the executables in the build directory which are being copied for installation. It should only strip the copies that are installed.

Normally we do not recommend stripping an executable unless you are sure the program has no bugs. However, it can be reasonable to install a stripped executable for actual execution while saving the unstripped executable elsewhere in case there is a bug.

</dd>

<dt>‘<samp>clean</samp>’</dt>

<dd>

Delete all files in the current directory that are normally created by building the program. Also delete files in other directories if they are created by this makefile. However, don’t delete the files that record the configuration. Also preserve files that could be made by building, but normally aren’t because the distribution comes with them. There is no need to delete parent directories that were created with ‘<samp>mkdir -p</samp>’, since they could have existed anyway.

Delete <samp>.dvi</samp> files here if they are not part of the distribution.

</dd>

<dt>‘<samp>distclean</samp>’</dt>

<dd>

Delete all files in the current directory (or created by this makefile) that are created by configuring or building the program. If you have unpacked the source and built the program without creating any other files, ‘<samp>make distclean</samp>’ should leave only the files that were in the distribution. However, there is no need to delete parent directories that were created with ‘<samp>mkdir -p</samp>’, since they could have existed anyway.

</dd>

<dt>‘<samp>mostlyclean</samp>’</dt>

<dd>

Like ‘<samp>clean</samp>’, but may refrain from deleting a few files that people normally don’t want to recompile. For example, the ‘<samp>mostlyclean</samp>’ target for GCC does not delete <samp>libgcc.a</samp>, because recompiling it is rarely necessary and takes a lot of time.

</dd>

<dt>‘<samp>maintainer-clean</samp>’</dt>

<dd>

Delete almost everything that can be reconstructed with this Makefile. This typically includes everything deleted by `distclean`, plus more: C source files produced by Bison, tags tables, Info files, and so on.

The reason we say “almost everything” is that running the command ‘<samp>make maintainer-clean</samp>’ should not delete <samp>configure</samp> even if <samp>configure</samp> can be remade using a rule in the Makefile. More generally, ‘<samp>make maintainer-clean</samp>’ should not delete anything that needs to exist in order to run <samp>configure</samp> and then begin to build the program. Also, there is no need to delete parent directories that were created with ‘<samp>mkdir -p</samp>’, since they could have existed anyway. These are the only exceptions; `maintainer-clean` should delete everything else that can be rebuilt.

The ‘<samp>maintainer-clean</samp>’ target is intended to be used by a maintainer of the package, not by ordinary users. You may need special tools to reconstruct some of the files that ‘<samp>make maintainer-clean</samp>’ deletes. Since these files are normally included in the distribution, we don’t take care to make them easy to reconstruct. If you find you need to unpack the full distribution again, don’t blame us.

To help make users aware of this, the commands for the special `maintainer-clean` target should start with these two:

<div class="smallexample">

<pre class="smallexample">@echo 'This command is intended for maintainers to use; it'
@echo 'deletes files that may need special tools to rebuild.'
</pre>

</div>

</dd>

<dt>‘<samp>TAGS</samp>’</dt>

<dd>

Update a tags table for this program.

</dd>

<dt>‘<samp>info</samp>’</dt>

<dd>

Generate any Info files needed. The best way to write the rules is as follows:

<div class="smallexample">

<pre class="smallexample">info: foo.info

foo.info: foo.texi chap1.texi chap2.texi
        $(MAKEINFO) $(srcdir)/foo.texi
</pre>

</div>

You must define the variable `MAKEINFO` in the Makefile. It should run the `makeinfo` program, which is part of the Texinfo distribution.

Normally a GNU distribution comes with Info files, and that means the Info files are present in the source directory. Therefore, the Make rule for an info file should update it in the source directory. When users build the package, ordinarily Make will not update the Info files because they will already be up to date.

</dd>

<dt>‘<samp>dvi</samp>’</dt>

<dt>‘<samp>html</samp>’</dt>

<dt>‘<samp>pdf</samp>’</dt>

<dt>‘<samp>ps</samp>’</dt>

<dd>

Generate documentation files in the given format. These targets should always exist, but any or all can be a no-op if the given output format cannot be generated. These targets should not be dependencies of the `all` target; the user must manually invoke them.

Here’s an example rule for generating DVI files from Texinfo:

<div class="smallexample">

<pre class="smallexample">dvi: foo.dvi

foo.dvi: foo.texi chap1.texi chap2.texi
        $(TEXI2DVI) $(srcdir)/foo.texi
</pre>

</div>

You must define the variable `TEXI2DVI` in the Makefile. It should run the program `texi2dvi`, which is part of the Texinfo distribution. (`texi2dvi` uses TeX to do the real work of formatting. TeX is not distributed with Texinfo.) Alternatively, write only the dependencies, and allow GNU `make` to provide the command.

Here’s another example, this one for generating HTML from Texinfo:

<div class="smallexample">

<pre class="smallexample">html: foo.html

foo.html: foo.texi chap1.texi chap2.texi
        $(TEXI2HTML) $(srcdir)/foo.texi
</pre>

</div>

Again, you would define the variable `TEXI2HTML` in the Makefile; for example, it might run `makeinfo --no-split --html` (`makeinfo` is part of the Texinfo distribution).

</dd>

<dt>‘<samp>dist</samp>’</dt>

<dd>

Create a distribution tar file for this program. The tar file should be set up so that the file names in the tar file start with a subdirectory name which is the name of the package it is a distribution for. This name can include the version number.

For example, the distribution tar file of GCC version 1.40 unpacks into a subdirectory named <samp>gcc-1.40</samp>.

The easiest way to do this is to create a subdirectory appropriately named, use `ln` or `cp` to install the proper files in it, and then `tar` that subdirectory.

Compress the tar file with `gzip`. For example, the actual distribution file for GCC version 1.40 is called <samp>gcc-1.40.tar.gz</samp>. It is ok to support other free compression formats as well.

The `dist` target should explicitly depend on all non-source files that are in the distribution, to make sure they are up to date in the distribution. See [Making Releases](Releases.html#Releases).

</dd>

<dt>‘<samp>check</samp>’</dt>

<dd>

Perform self-tests (if any). The user must build the program before running the tests, but need not install the program; you should write the self-tests so that they work when the program is built but not installed.

</dd>

</dl>

The following targets are suggested as conventional names, for programs in which they are useful.

<dl compact="compact">

<dt>`installcheck`</dt>

<dd>

Perform installation tests (if any). The user must build and install the program before running the tests. You should not assume that <samp>$(bindir)</samp> is in the search path.

</dd>

<dt>`installdirs`</dt>

<dd>

It’s useful to add a target named ‘<samp>installdirs</samp>’ to create the directories where files are installed, and their parent directories. There is a script called <samp>mkinstalldirs</samp> which is convenient for this; you can find it in the Gnulib package. You can use a rule like this:

<div class="smallexample">

<pre class="smallexample"># Make sure all installation directories (e.g. $(bindir))
# actually exist by making them if necessary.
installdirs: mkinstalldirs
        $(srcdir)/mkinstalldirs $(bindir) $(datadir) \
                                $(libdir) $(infodir) \
                                $(mandir)
</pre>

</div>

or, if you wish to support `DESTDIR` (strongly encouraged),

<div class="smallexample">

<pre class="smallexample"># Make sure all installation directories (e.g. $(bindir))
# actually exist by making them if necessary.
installdirs: mkinstalldirs
        $(srcdir)/mkinstalldirs \
            $(DESTDIR)$(bindir) $(DESTDIR)$(datadir) \
            $(DESTDIR)$(libdir) $(DESTDIR)$(infodir) \
            $(DESTDIR)$(mandir)
</pre>

</div>

This rule should not modify the directories where compilation is done. It should do nothing but create installation directories.

</dd>

</dl>

* * *

<div class="header">

Next: [Install Command Categories](Install-Command-Categories.html#Install-Command-Categories), Previous: [Directory Variables](Directory-Variables.html#Directory-Variables), Up: [Makefile Conventions](Makefile-Conventions.html#Makefile-Conventions)   [[Contents](index.html#SEC_Contents "Table of contents")][[Index](Index.html#Index "Index")]

</div>
