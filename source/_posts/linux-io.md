---
title: Linux I/O
date: 2020-01-31 15:50:50
categories:
tags:
---

A command expects the first three **`file descriptors`** to be available. The first, fd 0 (standard input, stdin), is for reading. The other two (fd 1, stdout and fd 2, stderr) are for writing.

There is a stdin, stdout, and a stderr associated with each command. **`ls 2>&1`** means temporarily connecting the stderr of the ls command to the same "resource" as the shell's stdout.

By convention, a command reads its input from fd 0 (stdin), prints normal output to fd 1 (stdout), and error ouput to fd 2 (stderr). If one of those three fd's is not open, you may encounter problems.

# Redirection

Multiple output streams may be redirected to one file.

``` bash
ls -yz >> command.log 2>&1
#  Capture result of illegal options "yz" in file "command.log."
#  Because stderr is redirected to the file,
#+ any error messages will also be there.

#  Note, however, that the following does *not* give the same result.
ls -yz 2>&1 >> command.log
#  Outputs an error message, but does not write to file.
#  More precisely, the command output (in this case, null)
#+ writes to the file, but the error message goes only to stdout.

#  If redirecting both stdout and stderr,
#+ the order of the commands makes a difference.
```

<!--more-->

``` bash
COMMAND_OUTPUT >
  # Redirect stdout to a file.
  # Creates the file if not present, otherwise overwrites it.

  ls -lR > dir-tree.list
  # Creates a file containing a listing of the directory tree.

: > filename
  # The > truncates file "filename" to zero length.
  # If file not present, creates zero-length file (same effect as 'touch').
  # The : serves as a dummy placeholder, producing no output.

> filename    
  # The > truncates file "filename" to zero length.
  # If file not present, creates zero-length file (same effect as 'touch').
  # (Same result as ": >", above, but this does not work with some shells.)

COMMAND_OUTPUT >>
  # Redirect stdout to a file.
  # Creates the file if not present, otherwise appends to it.


  # Single-line redirection commands (affect only the line they are on):
  # --------------------------------------------------------------------

1>filename
  # Redirect stdout to file "filename."
1>>filename
  # Redirect and append stdout to file "filename."
2>filename
  # Redirect stderr to file "filename."
2>>filename
  # Redirect and append stderr to file "filename."
&>filename
  # Redirect both stdout and stderr to file "filename."
  # This operator is now functional, as of Bash 4, final release.
2>&1
  # Redirects stderr to stdout.
  # Error messages get sent to same place as standard output.
    >>filename 2>&1
        bad_command >>filename 2>&1
        # Appends both stdout and stderr to the file "filename" ...
    2>&1 | [command(s)]
        bad_command 2>&1 | awk '{print $5}'   # found
        # Sends stderr through a pipe.
        # |& was added to Bash 4 as an abbreviation for 2>&1 |.
0< FILENAME
  < FILENAME
    # Accept input from a file.
    # Companion command to ">", and often used in combination with it.
    #
    # grep search-word <filename
```

# Pipe

``` bash
|
    # Pipe.
    # General purpose process and command chaining tool.
    # Similar to ">", but more general in effect.
    # Useful for chaining commands, scripts, files, and programs together.
    cat *.txt | sort | uniq > result-file
    # Sorts the output of all the .txt files and deletes duplicate lines,
    # finally saves results to "result-file".
```


# Useful Sites

http://tldp.org/LDP/abs/html