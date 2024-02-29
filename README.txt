JiayiSong #2079007
XunGong#2055079


README -- Simplified xv6 Shell


This project implements a simplified version of the xv6 shell. It is designed to provide a basic understanding of how a shell interprets commands, manages processes, and handles input/output redirection and piping. The shell is capable of executing commands, changing directories, managing input/output redirection, and creating pipelines between commands.


Features Implemented:
1.Command Execution: It supports executing basic commands by invoking execvp, which replaces the current process image with a new process image specified by the command. This is represented by the execcmd struct.
2.Input/Output Redirection: It handles input (<) and output (>) redirection, allowing a command to read input from a file or write output to a file, respectively. This is managed by the redircmd struct, which includes details about the file, mode of opening the file, and the file descriptor.
3.Piping: The shell supports piping (|) between commands, enabling the output of one command to be used as input for another. This is facilitated by the pipecmd struct, which contains pointers to the commands on the left and right sides of the pipe.
4.Command Parsing: The shell parses the command line input to understand which operations to perform (execution, redirection, or piping). This involves breaking down the input into tokens and constructing the appropriate command structures (execcmd, redircmd, pipecmd).
5.Error Handling: It includes basic error handling for system calls like fork, pipe, open, dup2, and execvp, ensuring that the shell exits gracefully if an error occurs.


Execution: 
Implemented via execvp, allowing the execution of system commands.
Redirection: Managed by fileop, which opens files with appropriate permissions and duplicates file descriptors.
Implementation as follows:
1.Open the specified file with appropriate flags:For output redirection (>), open the file for writing. If the file does not exist, create it. If it does exist, truncate it. Use O_WRONLY | O_CREAT | O_TRUNC flags.For input redirection (<), open the file for reading with O_RDONLY.
2.Duplicate file descriptors:Use dup2 to duplicate the file descriptor of the opened file to standard input or standard output, as appropriate. This ensures that the command reads from or writes to the specified file instead of the console. 
3. Close the original file descriptor to clean up and avoid resource leaks.
4.Execute the command that the redirection applies to by recursively calling runcmd with the command part of the redircmd structure.
Piping: Utilizes UNIX pipes to connect the standard output of one command to the standard input of another.
Implementation as follows:
1.Struct Definition: The pipecmd struct represents a pipe command, holding pointers to the left and right commands around the pipe symbol.
2.Pipe Execution in runcmd Function:The execution of a pipe command is handled in the runcmd function. When a pipecmd is encountered (case '|':)
      2.1 Create a Pipe: It calls pipe(p) to create a new pipe, where p[0] is the read end and p[1] is the write end.
      2.2 Forking: It forks the process using fork1(), creating a parent and a child process. The child process will execute the left side of the pipe, and the parent will execute the right side.
      2.3 In the Child Process (Left side):
            Close the read end of the pipe (close(p[0])) since it's not needed.
            Redirect standard output to the write end of the pipe using dup2(p[1], STDOUT_FILENO).
            Close the write end of the pipe (close(p[1])) after duplication.
            Recursively call runcmd to execute the command on the left side of the pipe.
      2.4 In the Parent Process (Right side):
            Close the write end of the pipe (close(p[1])) since it's not needed.
            Redirect standard input to the read end of the pipe using dup2(p[0], STDIN_FILENO).
            Close the read end of the pipe (close(p[0])) after duplication.
            Recursively call runcmd to execute the command on the right side of the pipe.
            Wait for the child process to finish using waitpid(ppid, &r, 0).
Command Parsing: parsecmd and related functions analyze the user input and construct the appropriate command structures.


Testing Process:
1.Basic Command Execution: Test by entering simple commands like ls, echo hello, and cat filename. Verify that the shell executes these commands correctly and displays the expected output.
2.Redirection: Test input and output redirection using commands like echo hello > outfile.txt to write output to a file and cat < infile.txt to read input from a file. Verify the contents of the files before and after executing these commands.
3.Piping: Test piping between commands with examples like cat infile.txt | grep searchterm or ls | wc -l to ensure the output of the first command is correctly passed as input to the second command.
4.Error Cases: Intentionally use commands with errors, such as nonexistent commands, to verify that the shell handles these gracefully, displaying appropriate error messages without crashing.
5.Complex Combinations: Test combinations of features, such as using piping and redirection together, e.g., grep searchterm < infile.txt | wc -l > outfile.txt, to ensure the shell can handle complex command lines.


How to Use：
Compilation: Compile the shell program using gcc with standard flags.
Running the Shell: Execute the compiled binary to start the shell. You will be greeted with a $ prompt awaiting your commands.
> $ gcc sh.c
> $ ./a.out 
> $ ./a.out < t.sh
    
