Download Link: https://assignmentchef.com/product/solved-cs0449-project-4-a-shell
<br>
In this project, you’ll be making a simple Unix shell. A shell is what you interact with when you log into thoth – it’s a command-line interface for running programs.

<strong>You already did some of this project in lab 7</strong> – that’s exactly how you’ll run programs!

<h2>Isn’t a shell a special kind of program?</h2>

Nope! A shell is just a user-mode process that lets you interact with the operating system. The basic operation of a shell can be summed up as:

<ol>

 <li>Read a command from the user</li>

 <li>Parse that command</li>

 <li>If the command is valid, run it</li>

 <li>Go back to step 1</li>

</ol>

The shell you interact with when you log into thoth is called <strong>bash</strong>. You can even run bash inside itself:

(13) thoth $ bash(1)  thoth $ pstree &lt;yourusername&gt;sshd───bash───bash───pstree(2)  thoth $ exit(14) thoth $ _

You’ll see the command numbers change, since you’re running bash inside of bash. pstree will also show this – a bash process nested inside another bash process!

When you write your shell, you can test it like any other program you’ve written.

(22) thoth $ ./myshellmyshell&gt; lsmyshell    myshell.cmyshell&gt; exit(23) thoth $ _

<h2>Input tokenization</h2>

Use fgets() with a generously-sized input buffer, like 300 characters.

Once you have the input, you can <em>tokenize</em> it (split it into “words”) with the strtok() function. It behaves oddly, so be sure to read up on it.

<a href="/teaching/classes/cs0449/projects/strtok.c">Here is a sample program that demonstrates strtok.</a>. Feel free to use it as the basis for your command parsing, but remember…

Since strtok operates in place, you <strong>cannot return the resulting array from a function.</strong> You have to allocate that array in the function that needs it, and pass a char** pointer as an argument.

In the worst case where someone types e.g. a b c d e f g h i…, you could have half as many tokens as the size of your character buffer – so, 150 tokens.

For strtok()’s “delim” parameter, you can give it this string:

” t
”

Get the string tokenization working <em>first.</em> Test it out well, and try edge cases – typing nothing, typing many things, typing several spaces in a row, using tab characters…

<h2>Commands</h2>

Many of the commands you’re used to running in the shell are actually <em>builtins</em> – commands that the shell understands and executes instead of having another program execute them. This makes the shell somewhat faster, because it doesn’t have to start a new process for each command.

Anything that <strong>isn’t</strong> a builtin should be interpreted as a command to run a program.

Following is a list of commands you need to support.

<h2>exit and exit number</h2>

<strong>Functions needed:</strong> exit()

The simplest command is exit, as it just… exits the shell.

<strong>NOTE: In all these examples, </strong><strong>myshell&gt; </strong><strong>indicates your shell program’s prompt, and </strong><strong>$</strong><strong> indicates bash’s prompt.</strong>

$ ./myshellmyshell&gt; exit$ _

You also need to support giving an argument to exit. It should be a number, and it will be returned to bash. You can check it like so:

myshell&gt; exit 45$ echo $?45$ _

The echo $? command <em>in bash</em> will show the exit code from the last program.

If no argument is given to exit, it should return 0:

myshell&gt; exit$ echo $?0$ _

<strong>Hint:</strong> there are a few functions in the C standard library you can use to parse integers from strings. You’ve used at least one before…

<h2>cd dirname</h2>

<strong>Functions needed:</strong> chdir()

You know how cd works! You don’t have to do anything special for the stuff that comes after the cd. chdir() handles it all for you.

Really, chdir() handles it <em>all</em> for you. You don’t have to parse the path, or look for ‘..’, or make sure paths are relative/absolute etc. chdir() is like cd in function form.**

<strong>You do not need to support </strong><strong>cd</strong><strong> without an argument.</strong> Just regular old cd.

<strong>You do not need to support </strong><strong>cd ~</strong><strong>.</strong> This is actually a bash feature, but it’s kind of complicated, so don’t worry about it.

You can see if it works properly using the pwd program, once your shell can run regular programs.

myshell&gt; cd testmyshell&gt; pwd/afs/pitt.edu/home/x/y/xyz00/private/testmyshell&gt; cd ..myshell&gt; pwd/afs/pitt.edu/home/x/y/xyz00/privatemyshell&gt; _

<h2>Regular programs</h2>

<strong>Functions needed:</strong> fork(), execvp(), exit(), waitpid(), signal()

If something doesn’t look like any built-in command, <strong>run it as a regular program.</strong> You should support commands with or without arguments.

You basically did this with <a href="../labs/lab7.html">lab 7</a>! You can use that as a starting point.

<strong>Your shell should support ANY number of arguments to programs, not just zero or one.</strong>

For example, <strong>and these are just examples:</strong> ANY program should be able to be run like this:

myshell&gt; lsmyshell.c    myshell    Makefilemyshell&gt; pwd/afs/pitt.edu/home/x/y/xyz00/privatemyshell&gt; echo “hello””hello”myshell&gt; echo 1 2 3 4 51 2 3 4 5myshell&gt; touch one two threemyshell&gt; ls -lh .total 9K-rw-r–r– 1 xyz00 UNKNOWN1 2.8K Apr  9 22:04 myshell.c-rwxr-xr-x 1 xyz00 UNKNOWN1 4.4K Apr  9 22:04 myshell-rw-r–r– 1 xyz00 UNKNOWN1  319 Apr  9 18:51 Makefile-rw-r–r– 1 xyz00 UNKNOWN1    0 Apr  9 22:05 one-rw-r–r– 1 xyz00 UNKNOWN1    0 Apr  9 22:05 two-rw-r–r– 1 xyz00 UNKNOWN1    0 Apr  9 22:05 threemyshell&gt; _

<h3>Catching Ctrl+C</h3>

Ctrl+C is a useful way to stop a running process. However by default, if you Ctrl+C while a child process is running, the parent will terminate too. So if you try to use it while running a program in your shell…

$ ./myshellmyshell&gt; cattyping stuff here…typing stuff here…cat just copies everything I type.cat just copies everything I type.&lt;ctrl+C&gt;$ _

I tried to exit cat by using Ctrl+C but it exited my shell too!

Making this work right is pretty easy.

<ul>

 <li>At the beginning of main, set it to <em>ignore</em> SIGINT.</li>

 <li>In the child process <strong>(after </strong><strong>fork</strong><strong> but before </strong><strong>exec</strong><strong>)</strong>, set its SIGINT behavior to the <em>default.</em></li>

</ul>

Once that’s done, you can use Ctrl+C with abandon:

$ ./myshellmyshell&gt; catblahblahblahhhhhblahhhhh&lt;ctrl+C&gt;myshell&gt; exit$ _

<h3>The Parent Process</h3>

After using fork(), the parent process should wait for its child to complete. Things to make sure to implement:

<ul>

 <li>Make sure to check the return value from waitpid to see if it failed. (This is just like in lab7.)</li>

 <li>If the child did <em>not</em> exit normally (WIFEXITED gives false):

  <ul>

   <li>if the child terminated due to a signal, print out which signal killed it. (Again, lab7!)</li>

   <li>otherwise, just say it terminated abnormally.</li>

  </ul></li>

</ul>

If you get errors about “implicit declaration of function ‘strsignal’” then add #define _GNU_SOURCE to the very top of your code, before any #include lines.

<h3>The Child Process</h3>

After using fork(), the child process is responsible for running the program. Things to make sure to implement:

<ul>

 <li>Set the SIGINT behavior to the default (as explained above in the Ctrl+C section).</li>

 <li>Use execvp to run the program.</li>

 <li>Print an error if execvp failed.</li>

</ul>

AND THEN…. exit() after you print the error. DON’T FORGET TO EXIT HERE. This is how you forkbomb. <em>If you forkbomb thoth multiple times, even if by accident, you may have your login privileges revoked.</em>

Notes on using execvp:

<ul>

 <li>The way execvp detects how many arguments you’ve given it is by <strong>putting a NULL string pointer as the “last” argument.</strong> You must put the NULL in your arguments array yourself, after parsing the user input. (The strtok example above does this.)</li>

 <li>execvp <em>only</em> returns if it failed. So you don’t technically need to check its return value.</li>

</ul>

<h2>Input and Output redirection</h2>

<strong>Functions needed:</strong> freopen()

Any regular program should also support having its stdin, stdout, <strong>or both</strong> redirected with the &lt; and &gt; symbols.

The redirections can come in either order, like cat &lt; input &gt; output or cat &gt; output &lt; input. Do not hardcode your shell to assume one will come before the other.

Your shell should support using input and output redirection on any <em>non-builtin command</em> with <em>any number of parameters.</em>

This means you should look for the redirections by looking starting at the <em>last</em> tokens. Then you can replace each redirection token (&lt; and &gt;) with NULL to ensure the right arguments get passed to the program.

bash lets you write ls&gt;out without spaces, but you don’t have to support that. ls &gt; out is fine for your shell.

myshell&gt; ls &gt; outputmyshell&gt; cat outputmyshell.cmyshellMakefileoutputmyshell&gt; less &lt; Makefile &lt;then less runs and shows the makefile&gt; myshell&gt; cat &lt; Makefile &gt; copymyshell&gt; lsmyshell.c    myshell    Makefile    output    copymyshell&gt; less copy &lt;then less runs and shows that ‘copy’ is identical to the original makefile&gt; myshell&gt; ls -lh . &gt; outputmyshell&gt; cat outputtotal 31K-rw-r–r– 1 xyz00 UNKNOWN1 2.8K Apr  9 23:18 myshell.c-rwxr-xr-x 1 xyz00 UNKNOWN1 4.4K Apr  9 23:18 myshell-rw-r–r– 1 xyz00 UNKNOWN1  319 Apr  9 18:51 Makefile-rw-r–r– 1 xyz00 UNKNOWN1   39 Apr  9 23:20 output-rw-r–r– 1 xyz00 UNKNOWN1  319 Apr  9 23:21 copymyshell&gt; _

Input and output redirection should detect and report the following errors:

<ul>

 <li>If the user tried to redirect stdin or stdout <strong>more than once</strong>

  <ul>

   <li>e.g. ls &gt; out1 &gt; out2</li>

   <li>or cat &lt; file1 &lt; file2</li>

  </ul></li>

 <li>If the file to read or write could not be opened</li>

</ul>

<h3>Opening the redirection files</h3>

You should open the redirection files <strong>in the child process after using </strong><strong>fork</strong><strong>,</strong> but <strong>before using </strong><strong>execvp()</strong><strong>.</strong>

In order to redirect stdin and stdout, you have to open new files to take their place. freopen() is the right choice for this.

<ul>

 <li>When opening a file for redirecting stdin, you want to open the file as read-only.</li>

 <li>When opening a file for redirecting stdout, you want to open the file as write-only.</li>

</ul>