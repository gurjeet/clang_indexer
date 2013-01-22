CLIC: Clang Indexer for C/C++
=============================

This is an indexer for C and C++ based on the libclang library.

There are several parts to it:

* The clic_add, clic_rm, clic_clear utilities for updating a database of symbols
* The clic_update.sh bash script, which runs the aformentioned utilities as required when the
  sources change
* A Vim plugin, hosted at https://github.com/exclipy/clang_complete, which let's the user query the
  database for references to the symbol under the cursor in Vim.

For further instructions, please consult this blog post:
http://blog.wuwon.id.au/2011/10/vim-plugin-for-navigating-c-with.html

Please note that this is an early stage of a little project that I've been doing just to make my own
life easier.  It has very sharp edges and I haven't really invested much effort in making it easy to
use.  Use at your own peril, it will destroy your code, format your hard drive, eat your children,
etc.

Ubuntu Build
============
sudo apt-get install cmake libclang-dev libboost-iostreams-dev libdb++-dev

Debug Build
===========
cmake -DCMAKE_BUILD_TYPE=Debug

Parsing Postgres sources
========================
These steps allow one to use the standard make scripts and launch `clic_add` just as if launched by the make utility.

Pseudocode:
.) Perform a `make clean` step so that the --dry-run can process _all_ files.
.) Use dry-run to generate the compiler commands.
.) Extract the lines that contain 'gcc', 'Entering directory', 'Leaving directory'
.) Discard any gcc lines that don't process a .c file
.) Replace the 'gcc' token with 'clic_add index.db tmp_index'
.) Reorder the command to place the .c file name at the end of the command, because clic_add expects .c file name as the last parameter.
.) Strip any -fexcess-precision=standard option from the command.
.) Strip any '-MF .deps/xxx.Po' flags from the command.
.) Replace 'Entering directory <dirname>' with 'pushd <dirname>'
.) Replace 'Leaving Directory' with 'popd'
.) Save the resulting commands in a file
.) Run 'make' so that the build process will actually generate intermediate files, such as errcodes.h.
.) Execute the file contaning the commands.

The last step of execting these commands takes more than 30 minutes, and the resulting index.db file is over 100 MB.


pgmake clean

pgmake --dry-run 2> /dev/null | grep -E '^ccache gcc|Entering directory|Leaving directory' | grep -E 'Entering|Leaving|\.c' | sed 's|ccache gcc|clic_add /home/gurjeet/Desktop/clic_text.db /tmp/clic_dummy_index|' | sed 's/\(.*\)\( .*\.c\)\( .*\)/\1\3\2/' | sed 's/\(.*\) -fexcess-precision=standard\(.*\)/\1\2/' | sed 's/\(.*\) -MF .*\.Po\(.*\)/\1\2/' | sed "s/.*Entering directory \`\(.*\)'/pushd \1/" | sed 's/.*Leaving directory.*/popd/' > ~/clic_test.sh

pgmake

~/clic_test.sh

The above version of the --dry-run command contains logic to strip -fexcess-precision=standard flag, which hasn't been tested. The following pipeline has been tested successfully.
pgmake --dry-run 2> /dev/null | grep -E '^ccache gcc|Entering directory|Leaving directory' | grep -E 'Entering|Leaving|\.c' | sed 's|ccache gcc|clic_add /home/gurjeet/Desktop/clic_text.db /tmp/clic_dummy_index|' | sed 's/\(.*\)\( .*\.c\)\( .*\)/\1\3\2/' | sed 's/\(.*\) -MF .*\.Po\(.*\)/\1\2/'| sed "s/.*Entering directory \`\(.*\)'/pushd \1/" | sed 's/.*Leaving directory.*/popd/' > ~/clic_test.sh

