Onboarding
==========

.. highlight:: sh

.. see Documentation/howto-doc.rst about |nbsp| versus |~|
.. |nbsp| unicode:: 0xa0
   :trim:

This page describes the pre-requisite knowledge needed to efficiently contribute
high-quality PRs to the Gramine project. This page also describes typical flows
that Gramine developers should follow to make the process of PR review pleasant
to everyone involved.

This page is intended to be a Getting Started guide for new Gramine developers,
*not* a reference manual on exact rules for contributing to Gramine. For the
latter, please refer to :doc:`contributing`.

Pre-requisite knowledge
-----------------------

Gramine is a Library OS that emulates the Linux kernel. Think of Gramine as a
tiny Linux re-implementation. As one example, Gramine has its own
implementations of Linux system calls. As another example, Gramine implements
its own ``/proc/`` file system.

Moreover, Gramine with the Intel SGX backend also emulates/validates some CPU
features. For example, Gramine-SGX verifies the return values of the CPUID
instruction. As another example, Gramine-SGX verifies the sanity of the CPU/NUMA
topology.

Finally, Gramine is a standalone, "nostdlib" software. This means that Gramine
itself doesn't link against any standard libraries and cannot rely on common
functionality like ``malloc``, ``memcmp``, etc. (in fact, Gramine re-implements
these common functions from scratch). For example, upon startup Gramine must
relocate and resolve its own symbols because there is no external dynamic
linker/loader (``ld``) to which this work could be offloaded.

Therefore, a new Gramine developer should be well-versed in low-level
programming. We recommend the following books and resources:

- Operating systems concepts

  - "Operating Systems: Three Easy Pieces" by R. Arpaci-Dusseau and A.
    Arpaci-Dusseau, available online at https://pages.cs.wisc.edu/~remzi/OSTEP/
  - "Operating System Concepts" by Silberschatz, Galvin and Gagne, see
    https://www.os-book.com/OS10/index.html
  - "Modern Operating Systems" by A. Tanenbaum

- Programming for the Linux kernel

  - "Linux Kernel Development" by R. Love
  - "Linux System Programming" by R. Love
  - The Linux Kernel documentation itself, see
    https://www.kernel.org/doc/html/latest/ and especially
    https://www.kernel.org/doc/html/latest/process/howto.html
  - The Linux source code itself, you can use https://elixir.bootlin.com/ to
    browse it
  - Man pages, type ``man man`` in your Linux terminal for reference or visit
    https://man7.org/linux/man-pages/

- Hardware architecture (especially for Gramine-SGX development)

  - "Computer Architecture: A Quantitative Approach" by D. Patterson and J.
    Hennessy
  - "Structured Computer Organization" by A. Tanenbaum
  - The latest revision of "Intel 64 and IA-32 Architectures Software
    Developer’s Manual, combined volumes"

A Gramine developer is also expected to be proficient in the following OS kernel
subsystems in particular:

- Virtual memory management (in particular, Virtual Memory Areas, or VMAs)

- File systems internals

  - Virtual File System concept
  - Dentries vs inodes
  - Mount points and chroot
  - File locking (mandatory locks, advisory locks)
  - Pseudo file systems (``/proc``, ``/sys``, ``/dev``)

- Thread management

  - Multi-threading
  - Concurrent programming, synchronization and data races, locks and lockless
    algorithms
  - Futexes

- Process management

  - Multi-processing
  - Fork and copy-on-write semantics
  - Execve

- Signal handling

  - Synchronous signals (traps, exceptions)
  - Asynchronous signals (interrupts)

- Communication primitives

  - Inter-Process Communication (IPC)
  - eventfd
  - Pipes
  - Named pipes (FIFOs)
  - Sockets (network sockets, UNIX domain sockets, raw NETLINK sockets)
  - IO multiplexing (select, poll, epoll)

- ELF binary loading and symbol relocation & resolution

  - ELF file format, see https://refspecs.linuxfoundation.org/elf/elf.pdf
  - ELF details, see
    https://www.intezer.com/blog/research/executable-linkable-format-101-part1-sections-segments/
    and follow-up posts
  - ELF linking and loading, see
    https://eli.thegreenplace.net/tag/linkers-and-loaders

Finally, a Gramine developer is also expected to be proficient in the software
development tools and flows adopted by the Gramine community:

- Git -- all Gramine development is done using Git distributed VCS

  - Official Git documentation, see https://git-scm.com/doc
  - Atlassian documentation and tutorials, see https://www.atlassian.com/git
  - GitLab documentation, see https://docs.gitlab.com/ee/topics/git/

- Git rebase flow -- all Gramine development is done using this flow, **not**
  merge flow

  - https://medium.com/singlestone/a-git-workflow-using-rebase-1b1210de83e5
  - https://www.atlassian.com/git/tutorials/merging-vs-rebasing
  - https://www.atlassian.com/git/articles/git-team-workflows-merge-or-rebase
    (Gramine uses what is called "rebase team policy")

- Developer Certificate of Origin (DCO) -- all Git commits in all Gramine
  repositories must contain the author's signoff

  - https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---signoff
  - https://developercertificate.org/
  - https://stackoverflow.com/questions/1962094/what-is-the-sign-off-feature-in-git-for

- GitHub -- all Gramine repositories are hosted on the GitHub web-site

  - Using GitHub is simple, but here is the official documentation:
    https://docs.github.com/en

- Reviewable -- all Gramine repositories use the Reviewable plugin to GitHub
  instead of the default GitHub "Review changes" flow

  - https://docs.reviewable.io/

- GDB debugger -- Gramine has full GDB support, see the next section

The above list with pre-requisite knowledge may be overwhelming at first.
Indeed, Gramine developers are expected to have solid systems programming
background and deep UNIX systems knowledge. We highly encourage to read several
books on Linux kernel development and OS concepts, as well as get comfortable
with git and GDB tools.

Debugging with GDB
------------------

We highly recommend to debug your applications and Gramine with classic GDB
debugger. Gramine has comprehensive support for GDB, both in direct mode and in
SGX mode. Please see :doc:`debugging` for information on how to build Gramine
with GDB and start Gramine under GDB.

GDB is a powerful tool, but it comes with its own quirks. Mastering GDB may take
time, but it pays off handsomely. Below we give some general advices for using
GDB to debug Gramine:

- For better debugging experience, we recommend to use GDB's TUI::

      $ (gdb) layout split
      ... now there are three panes: source code, asm and cmds

  You can return to normal command line by hitting ``Ctrl-x + a``. You can also
  switch between different layouts: ``layout src``, ``layout asm``, ``layout
  regs``. At any time, you can change focus to any of the panes: ``focus cmd``,
  ``focus src``, ``focus asm``, ``focus regs`` -- this is useful when you want
  to scroll up/down or look at the history of GDB commands. Also, whenever GDB's
  TUI output gets "mangled", type ``refresh``.

- For better debugging experience of multi-threaded applications, use ``set
  scheduler-locking on`` whenever you want to step through a single thread (GDB
  will not execute other threads in the background). To check the state of other
  threads, switch between threads via ``thread 1``, ``thread 2`` and so on.

- For better debugging experience of multi-process applications, use ``set
  follow-fork-mode parent`` or ``set follow-fork-mode child`` to instruct GDB to
  stay in the parent process or switch to the child process during fork.
  Another useful GDB command is ``set detach-on-fork on`` to instruct GDB to
  "forget about" (detach from) the parent/child process. To check the state of
  other processes (inferiors in GDB parlance), switch between them via
  ``inferior 1`` or (between processes + threads) ``thread 1.1``.

- We highly recommend to get acquainted with different ways of stepping through
  code in GDB. Apart from the classic ``continue``, ``step`` and ``next``, we
  recommend using ``finish`` and ``until``, as well as stepping through assembly
  code: ``stepi`` and ``nexti``. Stepping through assembly is especially useful
  when debugging tricky bugs in SGX.

Also, pay attention to Gramine-SGX quirks when debugged under GDB:

- Periodically, when stepping through the code, GDB may unexpectedly jump to
  ``sgx_entry.S: async_exit_pointer``. This is the "landing pad" of the AEX flow
  of Intel SGX, and can happen at any moment in SGX enclave execution. Simply
  step through this ``async_exit_pointer`` function until the ``enclu``
  instruction (which performs ERESUME), and GDB will continue at the correct
  in-enclave code.

- Currently, when executing in-enclave code, GDB correctly shows values only in
  General Purposes Registers (GPRs) like ``eax``, ``rip``, etc. Special-purpose
  registers like ``xmm0``, ``fs``/``gs``, ``bnd1`` will show *incorrect*
  values.

Typical Gramine development flows
---------------------------------

Below we briefly discuss three typical Gramine flows, with snippets from Bash
terminal sessions and GDB debugging sessions.

Fixing a bug
^^^^^^^^^^^^

You may have encountered a genuine bug in Gramine, when your application runs
fine on native Linux but fails under Gramine::

    $ ./myapp
    <expected output, terminates normally>

    $ gramine-sgx ./myapp
    <unexpected output, terminates abnormally>

#. Gather more information on the possible bug:

   - If you are not familiar with the application internal logic, try to gather
     as much information on the original app as possible. In particular, read
     its documentation, read its source code, read its GitHub/GitLab/mailing
     list discussions (if available), ask app developers.

   - If you experience the bug in Gramine-SGX, try Gramine in non-SGX mode::

         $ gramine-direct ./myapp

     If the application runs fine with ``gramine-direct``, then you know that
     the problem is in the SGX backend of Gramine (in other words, the problem
     is somewhere in the Linux-SGX PAL).

   - Run Gramine with debug information: use ``loader.log_level = "all"`` in
     your Gramine manifest file, rebuild and restart the app under Gramine. You
     can save the log, either by specifying ``loader.log_file = "gramine.log"``
     or simply by redirecting the output of Gramine, e.g.::

         $ gramine-sgx ./myapp 2>&1 | tee gramine.log

     Try to identify the system call in Gramine that goes wrong (e.g., returns
     an error code whereas it was supposed to finish successfully).

   - Analyze the manifest file carefully. If you suspect a memory allocation
     related bug or a buffer overrun bug, try to increase the following manifest
     options: ``loader.pal_internal_mem_size``, ``sys.stack.size``,
     ``sgx.enclave_size``. If at least one of the binaries spawned during app
     execution is non-PIE, then set ``sgx.nonpie_binary = true``. If you suspect
     problems with environment variables, see if it works with
     ``loader.insecure__use_host_env = true``. If you observe that memory
     addresses change constantly and hinder your debugging, set
     ``loader.insecure__disable_aslr = true``. But don't use the last two
     options in production; use them only for debugging and analysis!

   - Analyze FS mount points (``fs.mount`` fields) in the manifest file
     carefully. Check for duplicate mount points -- remember that a duplicate
     mount point's path *shadows* the previous mount point's path (i.e., if you
     have ``fs.mount.libs.path = "/lib"`` and then ``fs.mount.more_libs.path =
     "/lib"``, then files from the former path will disappear inside Gramine).

   - Collect the strace (system call trace) log on the original (native)
     application, e.g.::

         $ strace -ff ./myapp 2>&1 | tee native.log

     The strace log and the Gramine debug log are somewhat similar. You can
     compare two logs -- the good native one and the failing Gramine one --
     side-by-side in your favorite text editor. Hopefully this comparison will
     point you to the root cause of the Gramine failure.

#. Debug Gramine with GDB:

   - You probably already know which system call in Gramine derails the
     application. So you can put a breakpoint on its emulation function. For
     example, if Gramine's ``read`` emulation fails, then do::

         $ GDB=1 gramine-sgx ./myapp
         (gdb) break shim_do_read
         (gdb) run
         ... breakpoint hit, now look at regs, look at the backtrace, etc. ...

     - When debugging Gramine with GDB, use advices from the previous section.

#. Run your application with Gramine on several machines:

   - If possible, run your application with Gramine on machines with different
     configurations: different OS distros, different host Linux kernels,
     different SGX environments. Make notes of the exact configurations and how
     the application behaves on each of them.

   - Remember to add this information in the GitHub issue or PR description.
     This is very helpful in triaging the bug.

#. Fix the bug itself:

   - Now that you analyzed the bug and understood its root cause, you can change
     Gramine source code to fix the bug. Follow our :doc:`coding-style` when
     modifying the code. Also, write your code in a "similar" way to how it is
     done in the files that you modify. Finally, try to find similar places in
     Gramine code and follow their style.

   - Ask yourself: is your bug fix generic enough, or does it only fix this
     particular instantiation of the bug? It is possible that your fix actually
     introduced another bug in the code or broke some previous functionality.
     Try to make the bug fix "generic" and not "ad-hoc".

   - Ask yourself: is this the only place where this bug could happen? Try to
     find similar places where the same or a similar bug could manifest itself
     and fix all such places. For example, if you found a bug in the TCP stack
     of Gramine, check the UDP stack -- it may contain exactly the same bug.

   - Ask yourself: what do the man pages say about this scenario? For example,
     if you're fixing a bug in the ``read`` syscall emulation, check ``man 2
     read`` page in your Linux terminal (or read
     https://man7.org/linux/man-pages/man2/read.2.html if you prefer web pages).
     Pay attention to ``ERRORS``, ``NOTES`` and ``BUGS`` sections: they contain
     information on Linux pecularities and subtle details.

   - Ask yourself: what does the Linux kernel do in this scenario? It may take
     some effort to find a relevant code snippet in the Linux sources, but
     https://elixir.bootlin.com/ is a great place to analyze Linux internals.
     For example, the ``read`` syscall implementation starts here:
     https://elixir.bootlin.com/linux/v5.14.14/source/fs/read_write.c#L642.
     Remember that in the end, the Linux kernel is the ultimate source of truth.

#. Add tests for this bug fix:

   - The first option is to find an already existing Pal/LibOS regression test
     that works with the buggy Gramine subsystem. For example, if you found a
     bug in the UDP stack, look at the ``LibOS/shim/test/regression/udp.c``
     test. Find a place in this test where you can add the code that triggers
     the bug. Also add the corresponding check (if needed) in the Python test
     script (``LibOS/shim/test/regression/test_libos.py`` in case of LibOS
     regression tests).

   - The second option is to add a completely new test. Sometimes the bug
     reproduction code is too big or too specific to go into one of the already
     existing tests. In this case, just create a new test: the C source file,
     the corresponding manifest file (if the default ``manifest.template`` is
     not sufficient), and add a new test method in the Python test script.

   - The third option is to enable previously-disabled LTP tests. In Gramine,
     some LTP tests are currently disabled because they are known to trigger
     some bugs or unimplemented functionality. When you fix a bug, try to find
     LTP tests that were affected by this bug and re-enable them (see
     ``gramine/LibOS/shim/test/ltp/ltp.cfg`` file for the list of LTP tests).

   - The last option is to *not* add any new tests. This option is quite rare,
     but can be used in case of hard-to-reproduce bugs. For example, some bugs
     only manifest themselves after hours of execution or only in very specific
     environments.

#. Verify that your bug fix didn't break anything:

   - It is quite possible that your bug fix introduced another bug in Gramine,
     or it disabled some functionality, or it didn't take into account some
     corner case. You need to **run the whole test suite** of Gramine before
     publishing your bug fix::

         # build and run PAL regression tests
         $ cd Pal/regression
         $ make SGX=1 sgx-tokens -j
         $ make regression
         $ make regression SGX=1

         # build and run LibOS regression tests
         $ cd LibOS/shim/test/regression
         $ make SGX=1 sgx-tokens -j
         $ make regression
         $ make regression SGX=1

         # build and run LibOS FS tests
         $ cd LibOS/shim/test/fs
         $ make SGX=1 sgx-tokens -j
         $ make test
         $ make test SGX=1

         # build and run LTP tests (only in non-SGX mode)
         $ cd LibOS/shim/test/fs
         $ make -j
         $ make regression

     Verify that **all tests** succeed. If at least one test fails, analyze and
     debug this test. Modify your bug fix based on this analysis and **run the
     whole test suite** again. Repeat this process until all tests succeed.

   - In addition to running the test suite, try several example applications in
     Gramine (at least two apps that seem relevant to your bug fix)::

         # your bug fix pertains to TCP/UDP network stack, so try Redis
         $ cd CI-Examples/redis
         $ make SGX=1
         $ gramine-sgx redis-server --save '' --protected-mode no &
         $ src/src/redis-benchmark
         ... make sure the benchmark succeeds

     If example apps fail, analyze and debug them. Modify your bug fix. Repeat
     this process until all tests and applications succeed.

#. Publish your bug fix on https://github.com/gramineproject/gramine:

   - Git-commit your bug fix locally::

         $ git checkout -b <your-name>/<your-branch-name>
         $ git add <Gramine files that you modified>
         $ git commit --signoff
         ... type the commit title and the commit body message ...
         $ git diff HEAD^ HEAD  # double-check your commit!
         $ git push --set-upstream origin <your-name>/<your-branch-name>

   - Create the corresponding PR on https://github.com/gramineproject/gramine.
     GitHub interface will notice that you pushed a new branch to the repository
     and will automatically suggest to create a new PR. Follow the PR creation
     flow. Use a good PR title (typically the same as your git commit title).
     Add as much information to the PR description as possible.

     If there are open GitHub issues that will be fixed once your PR is merged,
     add ``Fixes #123`` (where "123" is the GitHub issue number) to the PR
     description. Add this line for every issue that will be fixed if there are
     several of them. Also, if there are open GitHub PRs that will be closed
     once your PR is merged (e.g., alternative fixes to the same bug), add
     ``Closes #321`` (where "321" is the GitHub PR number).

   - After you created the PR, open this PR's web page and go into Reviewable.
     Verify again that all the changes look correct, that you didn't
     accidentally add or delete some files, and that your code changes do not
     contain remnants from your debugging sessions. If you find some issues, use
     git fixup commits to fix them and then check the PR again.

#. Wait for reviews from Gramine maintainers. **Always use Reviewable** to read
   review comments and to reply. **Do not use GitHub review interface**.

   - Depending on the complexity of your bug fix and the current load of
     maintainers, the first reviews may come in 1 to 14 days. Please keep in mind
     that the review process is long and tedious. Take comfort in the fact that
     in e.g. the Linux development process, patches may take several years to be
     merged.

   - Be prepared to see quick reviews with "Please read the contributing guide"
     comments if you didn't do your due diligence. Reviewers are busy with other
     tasks (reviewing is never an actual "job"), so they don't have time and
     energy to copy-paste exerpts from documentation. Check the guides carefully
     and fix the identified problems in your submission.

   - Remember that many typos and obvious mistakes in your PR lead to bad first
     impression. Prepare your first submission carefully and verify it a couple
     times before publishing.

#. Work on PR issues identified by reviewers:

   - If you don't understand some comment, ask the reviewer to explain what they
     mean. If you disagree with some comment, explain your position. But be
     mindful -- reviewers typically know the project and the particular
     subsystem better than you, so take a moment to analyze their comment.

   - Fix/augment your code according to comments::

         $ git checkout <your-name>/<your-branch-name>
         ... modify your code ...

     After you finished all your fixups, **run the whole test suite** and
     **several application examples** again, to make sure your fixups didn't
     introduce new problems.

   - Follow the git fixup commit flow to publish the new revision::

         $ git add <fixed/augmented files>
         $ git commit --signoff --fixup=<hash of your main commit>
         $ git diff HEAD^ HEAD  # double-check your fixup commit
         $ git push

   - After you git-pushed the changes, open your PR's web page and go into
     Reviewable. Verify that all changes look correct. Don't forget to reply
     with "Done" (or mark the reviewer's comment as Resolved) in Reviewable.
     Take into account that reviewers only track changes in Reviewable comments;
     reviewers do not track new git commits in PRs!

#. If some reviewers do not reply for a long time (4-5 days), ping them
   explicitly, either on GitHub or via our Gitter messaging system.

#. Continue resolving reviewers' comments until all reviewers explicitly approve
   the PR. Remember that at least two reviewers from two different organizations
   must approve the PR. After your got approvals from all reviewers, there is
   nothing for you to do -- reviewers will rebase and merge your PR themselves.

#. After the PR is merged by reviewers/maintainers, notify all сoncerned parties
   about this fact, so they can test Gramine with your bug fix.

The best way to learn about these flows and requirements on the bug fix PRs is
to look at already-merged PR examples. Here are a few good examples:

- https://github.com/gramineproject/gramine/pull/165
- https://github.com/gramineproject/gramine/pull/76
- https://github.com/gramineproject/gramine/pull/35

Please examine the history of discussions in this PRs, code revisions, and the
final code that was approved and merged. You can use Reviewable for this and
unfold all "resolved discussions" to read the comments.

Implementing a new system call
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You may have encountered a situation when your application depends on some
system call that is not implemented in Gramine (recall that ``-38`` is the
``-ENOSYS`` error code, which means "syscall not implemented")::

    $ gramine-sgx ./myapp
    .. [P1:T1:app] trace: ---- shim_some_syscall(...) = -38 ..
    <unexpected output, terminates abnormally>

#. Make sure this system call is required:

   - It is normal that some system calls are not implemented by the OS kernel.
     In this case, the application detects ``-ENOSYS`` and either falls back to
     another implementation (e.g., app first tries ``flock()`` and then falls
     back to ``fcntl()``) or simply ignores the result of this syscall (e.g.,
     ``shim_ioctl(1, TCGETS, ..)`` can be safely ignored).

     Make sure that ``some_syscall`` returning ``-ENOSYS`` in the log is
     actually the root cause of the application failure. Sometimes such syscalls
     may be a "red herring", and the real root cause lies somewhere else.

   - Ask yourself: does it make sense to implement this system call in Gramine?
     Remember that Gramine is not a general-purpose OS: Gramine is not supposed
     to run e.g. admin tools or tools to inspect other applications. For
     example, it makes no sense to implement ``ptrace()`` or ``syslog()`` in
     Gramine. In case of doubt, ask maintainers of Gramine whether a specific
     system call makes sense for the project.

#. Analyze the implementation of the system call:

   - Understand the purpose, side effects and implications of the system call in
     detail. Read through its man pages (via ``man 2 some_syscall``), google it,
     read relevant blog posts and StackOverflow answers. A particularly useful
     resource for Linux system calls is https://lwn.net/Articles.

   - Read through the Linux sources: how does Linux implement this system call?
     https://elixir.bootlin.com/ is a great place to analyze Linux internals.
     For example, the ``read`` syscall implementation starts here:
     https://elixir.bootlin.com/linux/v5.14.14/source/fs/read_write.c#L642.
     Remember that in the end, the Linux kernel is the ultimate source of truth.

   - Analyze the relevant subsystems of Gramine: which features, tools,
     functions, components of Gramine are required to implement the system call?
     For example, the ``read`` syscall is very generic: it applies to regular
     files, pipes, sockets, eventfd, etc. So the ``read`` syscall touches almost
     all components and file systems of Gramine.

     If the system call requires to implement a whole new sub-system or a
     component in Gramine, do **not** try to implement the system call
     immediately. First discuss this with Gramine maintainers. Implementing such
     a system call may be a very complex task, and you will need help and/or
     guidance from other Gramine developers.

#. Implement the system call in Gramine. If it is a family of related system
   calls, implement all of them.

   - Implement the main emulation function ``shim_do_some_syscall()``. If the
     system call belongs to some family of already-implemented system calls, add
     this function to the already-existing Gramine C file. Otherwise, create a
     new C file under ``LibOS/shim/src/sys/``.

   - Implement the required sub-systems or components in general code of
     Gramine. For example, if you need to add new fields to the thread object,
     modify ``LibOS/shim/include/shim_thread.h`` and
     ``LibOS/shim/src/bookkeep/shim_thread.c``.

   - If the system call cannot be resolved entirely inside the LibOS component
     of Gramine, and the current set of PAL API functions (``Dk..()`` functions)
     is not enough to service this system call, you must add a new PAL API
     function (e.g., ``DkSomeNewFunction``). You will need to implement the
     entry-point function (``DkSomeNewFunction``) in the common PAL code, as
     well as each host-specific function (``_DkSomeNewFunction``) in each
     supported host in PAL.

     Note that Gramine strives to keep the PAL API as small as possible. It is
     highly discouraged to create new PAL functions without real need.

   - You may find more information on syscall implementation in
     :doc:`new-syscall`.

#. Add tests for this system call. See comments in "Fixing a bug" for more
   details.

The rest steps are the same as in "Fixing a bug" section above. You need to
verify your modifications by running all tests and several relevant
applications. Then you publish your code on GitHub and wait for reviews. You
continiously work with reviewers to refine your PR until all reviewers
explicitly approve the PR. After that, you wait for the PR to be merged and then
you notify all concerned parties about this fact.

Below are several good examples of adding a new system call:

- https://github.com/gramineproject/gramine/pull/146
- https://github.com/gramineproject/graphene/pull/2500

Adding new features
^^^^^^^^^^^^^^^^^^^

Sometimes you want to add not a new system call, but rather a new feature to
Gramine. the process of submitting a new feature is the same as in all other
sections -- you add the necessary code locally, then git-commit and git-push
them, then create a PR on GitHub and go through the review process.

One difference is that if the feature is user-visible, then you need to add
documentation about this feature (typically in the "Manifest syntax" page).

Below are several good examples of adding a new feature in Gramine:

- https://github.com/gramineproject/gramine/pull/37
- https://github.com/gramineproject/gramine/pull/56
- https://github.com/gramineproject/gramine/pull/99
- https://github.com/gramineproject/gramine/pull/101
- https://github.com/gramineproject/gramine/pull/169

Writing documentation
^^^^^^^^^^^^^^^^^^^^^

Sources for Gramine documentation are located in the same Gramine repository as
the core Gramine codebase itself: https://github.com/gramineproject/gramine.
The documentation is written in the reStructuredText file format. For more
details, refer to :doc:`howto-doc`.

The documentation sources are located under ``Documentation/``. For example, to
add documentation on the new manifest option, you open
``Documentation/manifest-syntax.rst`` and write the new text.

Since the documentation sources are placed in the same git repository, the
process of submitting new documentation is the same as in all other sections --
you modify the documentation files locally, then git-commit and git-push them,
then create a PR on GitHub and go through the review process.

Below are several good examples of adding documentation in Gramine:

- https://github.com/gramineproject/gramine/pull/114
- https://github.com/gramineproject/gramine/pull/148
- https://github.com/gramineproject/gramine/pull/160

Adding a new app example
^^^^^^^^^^^^^^^^^^^^^^^^

If you enabled some new application in Gramine and believe it may be of use to
others, you are encouraged to submit a PR with this app.

You need to be aware of several pecularities:

- Not all applications deserve to be in the list of curated examples in Gramine.
  For example, if the application is just another Python script, and enabling it
  in Gramine is trivial (by taking the already-existing Python example and
  slightly modifying its manifest file), then there is no sense in adding such
  an example.

- All PRs for new applications must be submitted **not** to the core Gramine
  repository, but into the accompanying repo
  https://github.com/gramineproject/examples.

- A new application must contain all the necessary information to be quickly
  installed and tested with Gramine. A new application is also expected to
  follow the standard used in other Gramine example apps. See
  https://github.com/gramineproject/examples/blob/master/README.rst for more
  details.

The process of submitting a new application example is the same as in all other
sections -- you add the Makefile to download and/or build the application
together with its manifest file locally, then git-commit and git-push them, then
create a PR on GitHub and go through the review process. The obvious difference
from other sections is that you do *not* need to run Gramine tests or other
applications for verification -- enabling your application in Gramine cannot
affect other tests/apps.

Below are several good examples of adding new applications in Gramine (but note
that these examples are slightly outdated and were modified in subsequent PRs):

- https://github.com/gramineproject/graphene/pull/2493
- https://github.com/gramineproject/graphene/pull/2465
