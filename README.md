InWee
=====
InWee is a convenience script to send text messages and commands to
WeeChat from shell or from within WeeChat via WeeChat's [FIFO pipe][1].

[1]: http://www.weechat.org/files/doc/stable/weechat_user.en.html#fifo_plugin


Necessity
---------
WeeChat keeps configuration in several .conf files in ~/.weechat
directory. The configuration can be edited with WeeChat `/set` commands.
Unfortunately, the same .conf files contain both the default
configuration options that have not been altered by the user as well as
custom configuration options specified by the user. This makes it
clumsy to pick only the user specific custom configuration, save it in a
separate file as backup, so that it can be applied later when needed,
say after reinstallation of operating system or on a new system.

One solution to this problem is to keep all the user specific
configuration in the form of `/set` commands in a file, say
settings.txt, as shown below.

     /set irc.server.freenode.nicks humpty,humpty_
     /set irc.server.freenode.username humpty
     /set irc.server.freenode.realname "Humpty Dumpty"
     /set irc.server.freenode.autojoin #weechat,#test,#freenode
     /save irc
     /print Saved configuration

This file can be fed into WeeChat's FIFO pipe while WeeChat is running
using the following shell command.

    sed 's/^/*/' settings.txt > ~/.weechat/weechat_fifo_*

The command first prefixes every line in the file with `*` (asterisk)
because that is how WeeChat's FIFO pipe expects the input to be. Then it
feeds the lines to WeeChat's FIFO pipe.

This tool makes the above step more convenient. With this tool, the
commands in settings.txt can be fed to WeeChat with the following
command while WeeChat is running.

    inwee settings.txt

This simple command offers the following advantages over the earlier
command involving sed command's output redirection.

  1. It is simpler and easier to type.
  2. It automatically prefixes every command in the input file with `*`
     (asterisk).
  3. It automatically checks if the FIFO pipe exists in ~/.weechat
     directory. If it does not exist, it quits with an error.
  4. It supports comments in the input file. Any line in the file where
     `#` (hash) is the first non-whitespace character is ignored as
     comment.

WeeChat can execute external commands with the `/exec` command and it
accepts the `-r` option to run commands after startup, so when WeeChat
is not running, it can be made to run the commands in settings.txt after
starting up as follows.

    weechat -r '/exec inwee settings.txt'


Installation
------------
InWee is a one file script. Therefore, installing it simply requires
downloading the file named `inwee` and copying it to some directory that
is present in the `PATH` environment variable. In many Linux systems,
`/usr/local/bin` or `~/bin` is a good location to copy this script.

### Manual ###
Here are the steps to manually download the script and install it.

  1. Visit <https://github.com/susam/inwee>.
  2. Click on 'Download ZIP' button.
  3. Unzip the downloaded file and copy the file named `inwee` into a
     directory in the `PATH` variable, e.g. /usr/local/bin or ~/bin.

### Using wget ###
Here are the shell commands to automatically download the script and
install it.

    wget https://github.com/susam/inwee/archive/master.zip -O inwee.zip
    unzip inwee.zip
    mkdir ~/bin
    cp inwee-master/inwee ~/bin

### Using git ###
If `git` is available, then the script may be obtained by cloning this
project.

    git clone https://github.com/susam/inwee.git
    mkdir ~/bin
    ln -sf "$(pwd)/inwee/inwee" ~/bin

This method of installation offers the benefit of obtaining updates
easily. First get into the directory where this project was cloned. Then
fetch the latest revision of the project.

    cd inwee
    git pull


Usage
-----
The following list shows various ways to use InWee from the shell.

  1. Send text or commands from standard input to the current buffer.

        echo hello world | inwee
        lspci | grep -i network | inwee
        echo '/join #test' | inwee
        echo /part | inwee
        echo /nick newnick | inwee

     Note that comments in shell begin with `#`, therefore the WeeChat
     `join` command in the second shell command above is quoted. Any
     text beginning with `/` (slash), followed by a word and whitespace
     is considered as a WeeChat command. All other text is considered as
     text messages to be sent on chat.

  2. Multiple texts and/or commands must be separated by a newline.

        printf '/join #test\n/join #flood\n' | inwee
        printf '/part #test\n/part #flood\n' | inwee
        printf 'hello world\nhow are you?\n' | inwee

     Every line of text or command must also end with a newline. Since
     `printf` does not append a newline in the end like `echo` does, we
     end the strings passed to `printf` in each command above with a
     newline.

     Blank lines, lines that consist only of whitespace characters and
     lines that contain `#`, i.e. hash, as the first non-whitespace
     character are ignored.

  3. Text or commands may be sent to a specific WeeChat buffer with the
     `-b` or `--buffer` option.

        echo hello world | inwee -b 'irc.freenode.#test'

     With the above command, the text is sent to #test channel even when
     this channel is not in the current buffer.

  4. Send text or commands from a file by specifying the path to the
     file as an argument.

        inwee commands.txt

     The file may contain one or more lines of text or commands. Blank
     lines, lines that consist only of whitespace characters and lines
     that contain `#`, i.e. hash, as the first non-whitespace character
     are ignored. Therefore, `#` can be used to begin single line
     comments.

From WeeChat's perspective, InWee is an external command that can be
invoked with the `inwee` command. WeeChat's `/exec` command executes
external commands inside WeeChat and displays the output locally, or
sends it to a buffer. Therefore, `inwee` can be used within WeeChat
using WeeChat's `/exec` command. This section shows a few such examples.
Note that unlike in the previous section where the command examples are
meant to be run in the shell, the commands below are meant to be run
inside WeeChat directly.

  1. Send text or commands from a file to the current buffer.

        /exec inwee commands.txt

  2. Send text or commands from a file to #test channel.

        /exec inwee -b irc.freenode.#test commands.txt


License
-------
This is free software. You are permitted to redistribute and use it in
source and binary forms, with or without modification, under the terms
of the Simplified BSD License. See LICENSE.md for the complete license.

This software is provided WITHOUT ANY WARRANTY; without even the implied
warranties of MERCHANTABILITY and FITNESS FOR A PARTICULAR PURPOSE. See
LICENSE.md for the complete disclaimer.


Contact
-------
To report any bugs, or suggest any improvements, please create a new
issue at <https://github.com/susam/inwee/issues>.
