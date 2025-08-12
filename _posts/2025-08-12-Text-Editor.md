---
layout: post
title: Text Editor.
subtitle: Simple text exitor written in C.
tags: [C]
---

### Creating a text editor: kilo.

We are going to write a text editor in C. We are using the following Make file:

```make
kilo: kilo.c 
	$(CC) kilo.c -o kilo -Wall -Wextra -pedantic -std=c99
```

1. We are declaring that we want to build 'kilo' binary from 'kilo.c'.
2. We are declaring the build command through the follwoing statements:

    - $(CC) is a variable that make expands automatically to 'cc'.
	- 'Wall' stands for “all Warnings”, and gets the compiler to warn you when it sees code in your program that might not technically be wrong, but is considered bad or questionable usage of the C language.
	- 'Wextra' and 'pedantic' turn on even more warnings.
	- 'std=c99' specifies the exact version of the C language standard we’re using.

This can be launched by writting 'make' in the terminal on the same folder. Is worth to mention that: It may output make: `kilo' is up to date.. It can tell that the current version of kilo.c has already been compiled by looking at each file’s last-modified timestamp. If kilo was last modified after kilo.c was last modified, then make assumes that kilo.c has already been compiled, and so it doesn’t bother running the compilation command. If kilo.c was last modified after kilo was, then make recompiles kilo.c

This text exitor has been made using the following [guide](https://viewsourcecode.org/snaptoken/kilo/)
<br>

### 1. Entering Raw Mode.

#### 1.1. Explaining Raw Mode.

In a text editor, *raw mode* disables the terminal's standard input processing, allowing the application (the text editor) to directly receive and process each keystroke without any automatic handling like echoing or line buffering. This gives the editor complete control over input, enabling features like real-time cursor movement and custom keybindings.

Let's start with the following code:

```c
#include <unistd.h>

int main() {
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1);
  return 0;
}
```

This code is including *unistd.h* file (which provides access to the POSIX API, the Linux OS API) and uses the *read()* syscall to read byte to byte from the standard input (STDIN_FILENO), this is our keyboard: file descriptor 0, into *c* variable and continuing until there is no more bytes to read. The *read()* function returns the number of bytes read, so in this case, the while loop continue undefinitively until no more data is available and 0 is returned.

If we make the file and try to run it, the terminal gets stucked listening to the standard input. The terminal by default starts in *cannonical mode*, this means that the terminal doesn't send the standard input buffer until a signal is triggered (usually, the user press Enter key). This is pretty useful to go backwards a line and fix errors or delete characters but it doesn't fit well in a text editor enviroment as we want to process each key as soon as is pressed. In summary, we want to achieve *raw mode* which is quite complex. 

<br>

#### 1.2. Turning off echoing. Modifying terminal's attributes.

The 'ECHO' feature causes each typed key to be printed to the terminal. For obvious reasons, this is not ideal if we are trying to build an interface to enter a text editor and we should disable it. There are some programs that disable this feature like SUDO.

We can sett terminal's attributes by:

1. Using *tcgetattr()* to read the current attributes into a termios struct.
2. Modifying the struct by hand.
3. Passing the modified struct to *tcsetattr()* to write the new terminal attributes back out.

All of this implemented in a function after including *termios.h* header file:

```c
#include <termios.h>
#include <unistd.h>

void enableRawMode(){ 
    //Define the termios struct and dump on it the attributes of the terminal referred to the STDIN filde.
    struct termios raw;
    tcgetattr(STDIN_FILENO, &raw);

    //We change the value asigning to it the same value except for the negative of the ECHO one.
    raw.c_lflag &= ~(ECHO);
    
    //Then we apply the attributes of the raw struct to the STDIN filde of the terminal.
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}

int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q');
  return 0;
}
```

The *enableRawMode()* function does exactly what we describe above. 

- First, we define the *termios* struct and filled up with the attributes of the STDIN filde of our terminal (STDIN_FILENO).
- Then, the ECHO feature is a bitflag on *c_lflag* (local flags) field so we change it asigning to it the same value plus the negative value of the ECHO key; ~(ECHO).

    We should slow down in this step, *c_lflag* is a bitmask: It's an integer where each bit represents a different terminal feature (ECHO, ICANON, ISIG, etc.), so, since the ECHO constant is defined in *termios.h* as a specific bit pattern that corresponds to the echo functionality, then only the ECHO feature is turned off and everything else remains the same.

    We can understand this in an ilustrative way by compiling and executing the following code:

    ```c
    #include <stdio.h>
    #include <termios.h>
    #include <unistd.h>

    void print_binary(unsigned int value, int bits) {
        for (int i = bits - 1; i >= 0; i--) {
            printf("%d", (value >> i) & 1);
            if (i % 4 == 0 && i > 0) printf(" ");
        }
    }

    void print_termios_flags(struct termios *raw) {
        printf("=== TERMIOS STRUCTURE ANALYSIS ===\n");
        printf("c_iflag: %u (0x%x)\n", raw->c_iflag, raw->c_iflag);
        printf("c_oflag: %u (0x%x)\n", raw->c_oflag, raw->c_oflag);
        printf("c_cflag: %u (0x%x)\n", raw->c_cflag, raw->c_cflag);
        printf("c_lflag: %u (0x%x)\n", raw->c_lflag, raw->c_lflag);
        
        printf("\n=== c_lflag DETAILED ANALYSIS ===\n");
        printf("c_lflag value: %u\n", raw->c_lflag);
        printf("c_lflag binary: ");
        print_binary(raw->c_lflag, 16);
        printf("\n");
        
        printf("\n=== INDIVIDUAL FLAGS IN c_lflag ===\n");
        printf("ECHO flag value: %u (0x%x)\n", ECHO, ECHO);
        printf("ECHO binary:     ");
        print_binary(ECHO, 16);
        printf("\n");
        
        printf("ICANON flag value: %u (0x%x)\n", ICANON, ICANON);
        printf("ICANON binary:     ");
        print_binary(ICANON, 16);
        printf("\n");
        
        printf("ISIG flag value: %u (0x%x)\n", ISIG, ISIG);
        printf("ISIG binary:       ");
        print_binary(ISIG, 16);
        printf("\n");
        
        printf("\n=== FLAG STATUS ===\n");
        printf("ECHO is %s\n", (raw->c_lflag & ECHO) ? "ON" : "OFF");
        printf("ICANON is %s\n", (raw->c_lflag & ICANON) ? "ON" : "OFF");
        printf("ISIG is %s\n", (raw->c_lflag & ISIG) ? "ON" : "OFF");
    }

    int main() {
        struct termios original, modified;
        
        // Get current terminal settings
        tcgetattr(STDIN_FILENO, &original);
        
        printf("ORIGINAL TERMINAL SETTINGS:\n");
        print_termios_flags(&original);
        
        // Make a copy to modify
        modified = original;
        
        printf("\n\n=== BITWISE OPERATION DEMONSTRATION ===\n");
        printf("We want to turn OFF the ECHO flag using: c_lflag &= ~(ECHO)\n\n");
        
        printf("Step 1: Original c_lflag\n");
        printf("c_lflag:     ");
        print_binary(modified.c_lflag, 16);
        printf(" (%u)\n", modified.c_lflag);
        
        printf("\nStep 2: ECHO flag\n");
        printf("ECHO:        ");
        print_binary(ECHO, 16);
        printf(" (%u)\n", ECHO);
        
        printf("\nStep 3: ~(ECHO) - bitwise NOT of ECHO\n");
        printf("~(ECHO):     ");
        print_binary(~ECHO, 16);
        printf(" (%u)\n", (unsigned short)~ECHO);
        
        printf("\nStep 4: c_lflag & ~(ECHO) - AND operation\n");
        unsigned int result = modified.c_lflag & ~ECHO;
        printf("Result:      ");
        print_binary(result, 16);
        printf(" (%u)\n", result);
        
        // Actually perform the operation
        modified.c_lflag &= ~(ECHO);
        
        printf("\nMODIFIED TERMINAL SETTINGS (ECHO turned OFF):\n");
        print_termios_flags(&modified);
        
        printf("\n\n=== TURNING ECHO BACK ON ===\n");
        printf("To turn ECHO back ON, we use: c_lflag |= ECHO\n");
        
        printf("Current c_lflag: ");
        print_binary(modified.c_lflag, 16);
        printf(" (%u)\n", modified.c_lflag);
        
        printf("ECHO flag:       ");
        print_binary(ECHO, 16);
        printf(" (%u)\n", ECHO);
        
        modified.c_lflag |= ECHO;
        
        printf("After OR:        ");
        print_binary(modified.c_lflag, 16);
        printf(" (%u)\n", modified.c_lflag);
        
        printf("\nFINAL RESULT (ECHO turned back ON):\n");
        printf("ECHO is %s\n", (modified.c_lflag & ECHO) ? "ON" : "OFF");
        
        return 0;
    }
    ```

    This will show the complete structure of the raw termios struct and the logic of the ECHO flag.

    ```less
    c_lflag: 35387 (0x8a3b)
    c_lflag binary: 1000 1010 0011 1011
    ECHO flag value: 8 (0x8)
    ECHO binary: 0000 0000 0000 1000 (8)
    ~(ECHO) binary: 1111 1111 1111 0111 (65527)
    Result:      1000 1010 0011 0011 (35379)
    ```

    So we are seeing the integer, then the binary representation in bits and the bit corresponding to the ECHO functionality is the first bit of the four quart, we get the binary mask of the negative ECHO and combine with the c_lflag by performing an AND (&) operation.

<br>

#### 1.3. Disable raw mode at exit.

After the program quits, depending on your shell, you may find your terminal is still not echoing what you type. We can just press Ctrl-C to start a fresh line of input to your shell, and type in 'reset' and press 'Enter'. This resets your terminal back to normal in most cases. 

However, the ideal scenario is to restore the terminal's original attributes when our program exists. For that, we will save a copu of the termios struct in teh original state and use *tcsetattr()* to reapply it to the terminal.

For that, we:

- Include the *stdlib.h* header file, which provides access to various general-purpose functions, including memory allocation, process control, conversions, and random number generation.
- Then, we modify the *enableRawMode()* function subtly:
    - First with *tcgetattr()* we store the original attributes of the terminal in *orig_termios* struct.
    - Then, we use the *exit handler* *atexit()* which calls the specified function (in this case disableRawMode, which applys original setting and is predefined) when programs exists. This form, we ensure whenever enableRawMode() is called and changes to the terminal are made, original state is restored at the end of the program.
    - Lastly, we form another termios struct, modify manually and apply the changes with tcsetattr().

```c
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>

struct termios orig_termios;

void disableRawMode(){
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios);
}

void enableRawMode(){ 
    tcgetattr(STDIN_FILENO, &orig_termios);
    atexit(disableRawMode);

    struct termios raw = orig_termios;
    tcgetattr(STDIN_FILENO, &raw);
    raw.c_lflag &= ~(ECHO);
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}

int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q');
  return 0;
}
```

<br>

#### 1.4. Turn off canonical mode.

Along with ECHO flag, in stdlib there is also the ICANON constant that hanfles the cannonical mode, remember that this mode makes the terminal to hold the stdin buffer until the ENTER signal is send:

```c
raw.c_lflag &= ~(ECHO | ICANON);
```
Applying logic ~(ECHO | ICANON) == ~(ECHO) && ~(ICANON), so we are setting the c_lflag with ECHO and ICANON with negative values.

With this change, the input buffer of the terminal gets send as soon as gets filled with data, this means we will finally be reading input byte-by-byte, instead of line-by-line after user press ENTER.

<br>

#### 1.4. Display keypresses.

To get an inner understanding of how raw mode works, we can print out every byte we read, to see that first, it dont get printed over the stdout (if we dont say so) and that is not necesary to press enter to send the stdin buffer into the program. 

Let's consider the following code:

```c
#include <ctype.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>

struct termios orig_termios;
void disableRawMode(){
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios);
}

void enableRawMode(){ 
    tcgetattr(STDIN_FILENO, &orig_termios);
    atexit(disableRawMode);

    struct termios raw = orig_termios;
    tcgetattr(STDIN_FILENO, &raw);
    raw.c_lflag &= ~(ECHO|ICANON);
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}

int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q') {
    if (iscntrl(c)) {
      printf("%d\n", c);
    } else {
      printf("%d ('%c')\n", c, c);
    }
  }
  return 0;
}
```

In the code before, we include two new libraries:

- **stdio.h**: that provides functions to handle input/output data to the program.
- **ctype.g**: that provides functions for testing and mapping characters.

Stdio does not need presentation, but *ctype.h* header file declares several functions that are useful for testing and mapping characters. From this library, we import *iscntrl()* tests whether a character is a control character. Control characters are nonprintable characters that we don’t want to print to the screen. ASCII codes 0–31 are all control characters, and 127 is also a control character. ASCII codes 32–126 are all printable. (Check out the ASCII table to see all of the characters.)

*printf()* can print multiple representations of a byte. %d tells it to format the byte as a decimal number (its ASCII code), and %c tells it to write out the byte directly, as a character.

This is a very useful program. It shows us how various keypresses translate into the bytes we read. Most ordinary keys translate directly into the characters they represent. But try seeing what happens when you press the arrow keys, or Escape, or Page Up, or Page Down, or Home, or End, or Backspace, or Delete, or Enter. Try key combinations with Ctrl, like Ctrl-A, Ctrl-B, etc:

```less
$ ./rawmode
27
91 ('[')
65 ('A')
27
91 ('[')
67 ('C')
27
91 ('[')
66 ('B')
27
91 ('[')
68 ('D')
32 (' ')
127
97 ('a')
```

<br>

#### 1.5. Turnoff Ctrl-C and Ctrl-Z signals.

By default, 'Ctrl-C' and 'Ctrl-Z' send both SIGINT (terminate program) and SIGTSTP (suspend program) signals, both of them handled by the ISIG constant in c_lflag of the termios struct and we can disable it as we did above:

```c
raw.c_lflag &= ~(ECHO | ICANON | ISIG);
```

This lines disables those signals that, for now on can be read as 3 and 26 bytes.

<br>

#### 1.6. Disabling Ctrl-S and Ctrl-Q.

By default, Ctrl-S and Ctrl-Q are used for *software flow control* (). Ctrl-S stops data from being transmitted to the terminal until you press Ctrl-Q. This means that, in the code before, if you press Ctrl-S, then the program keep listening to the STDIN but stop sending to the program the incoming input until you press Ctrl-Q, then the buffer gets freed and introduces the data at once.

This feature is now contemplated in the IXON constant of the *c_iflag* (input flag):

```c
raw.c_iflag &= ~(IXON);
```

Now Ctrl-S can be read as a 19 byte and Ctrl-Q can be read as a 17 byte.

<br>

#### 1.7. Disabling Ctrl-v

On some systems, when you type Ctrl-V, the terminal waits for you to type another character and then sends that character literally. For example, before we disabled Ctrl-C, you might’ve been able to type Ctrl-V and then Ctrl-C to input a 3 byte.

We can turn off this feature using the IEXTEN flag in c_lflag.

```c
raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
```

Ctrl-V can now be read as a 22 byte, and Ctrl-O as a 15 byte.

<br>

#### 1.8. Fix Ctrl-M.

If you run the program now and go through the whole alphabet while holding down Ctrl, you should see that we have every letter except M. Ctrl-M is weird: it’s being read as 10, when we expect it to be read as 13, since it is the 13th letter of the alphabet, and Ctrl-J already produces a 10. What else produces 10? The Enter key does.

It turns out that the terminal is helpfully translating any carriage returns (13, '\r') inputted by the user into newlines (10, '\n'). Let’s turn off this feature by switching off the ICRLN constant on *c_iflag*:

```c
raw.c_iflag &= ~(ICRNL | IXON);
```

Now Ctrl-M is read as a 13 (carriage return), and the Enter key is also read as a 13.

<br>

#### 1.9. Turn off all output processing.

Also, as the terminal translates all '\r' input to '\n' it also translates all '\n' outpud as '\r\n'. The terminal requires both of these characters in order to start a new line of text. The carriage return moves the cursor back to the beginning of the current line, and the newline moves the cursor down a line, scrolling the screen if necessary. 

We will turn off all output processing features by turning off the OPOST cosntant of the c_oflag. 

```c
raw.c_oflag &= ~(OPOST);
```

In practice, the "\n" to "\r\n" translation is likely the only output processing feature turned on by default.

If you run the program now, you’ll see that the newline characters we’re printing are only moving the cursor down, and not to the left side of the screen:

```less
$ ./kilo
115 ('s')
         115 ('s')
                  100 ('d')
                           97 ('a')
                                   115 ('s')
                                            100 ('d')
```

To fix that, let’s add carriage returns to our printf() statements:

```c

int main() {
  enableRawMode();
  char c;
  while (read(STDIN_FILENO, &c, 1) == 1 && c != 'q') {
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
  }
  return 0;
}
```

From now on, we’ll have to write out the full "\r\n" whenever we want to start a new line.

<br>

#### 1.10. Miscellaneous flags.

This step probably won’t have any observable effect for you, because these flags are either already turned off, or they don’t really apply to modern terminal emulators. But at one time or another, switching them off was considered (by someone) to be part of enabling “raw mode”, so we carry on the tradition (of whoever that someone was) in our program:

```c
raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
raw.c_cflag |= (CS8);
```

As far as I can tell:

- When BRKINT is turned on, a break condition will cause a SIGINT signal to be sent to the program, like pressing Ctrl-C.
- INPCK enables parity checking, which doesn’t seem to apply to modern terminal emulators.
- ISTRIP causes the 8th bit of each input byte to be stripped, meaning it will set it to 0. This is probably already turned off.
- CS8 is not a flag, it is a bit mask with multiple bits, which we set using the bitwise-OR (|) operator unlike all the flags we are turning off. It sets the character size (CS) to 8 bits per byte. On my system, it’s already set that way.

<br>

#### 1.11. A timeout for read().

Currently, read() will wait indefinitely for input from the keyboard before it returns. What if we want to do something like animate something on the screen while waiting for user input? We can set a timeout, so that read() returns if it doesn’t get any input for a certain amount of time.

VMIN and VTIME come from *termios.h* header file. They are indexes into the c_cc field, which stands for “control characters”, an array of bytes that control various terminal settings.

- VMIN value sets the minimum number of bytes of input needed before read() can return. We set it to 0 so that read() returns as soon as there is any input to be read. 

    ```c
    raw.c_cc[VMIN] = 0;
    ```

- VTIME value sets the maximum amount of time to wait before read() returns. It is in tenths of a second, so we set it to 1/10 of a second, or 100 milliseconds. If read() times out, it will return 0, which makes sense because its usual return value is the number of bytes read.

    ```c
    raw.c_cc[VTIME] = 1;
    ```

```c
void enableRawMode() {
  tcgetattr(STDIN_FILENO, &orig_termios);
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
}


int main() {
  enableRawMode();
  while (1) {
    char c = '\0';
    read(STDIN_FILENO, &c, 1);
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
    if (c == 'q') break;
  }
  return 0;
}
```

Now, the code acts as follows:

- It implements a while loop in which we sett the char 'c' with '\0' value and then read into 'c'. 
- Since we setup a timeout (VTIME), it will listen for a second and returns the number of bytes read
- Then the loops continues and print the 'c' char. 

If we let the terminal go untouched, it will spawn a 0 for a second until we keypress something to continue inmediately after.

<br>

#### 1.12. Error handling.

After all the changes applied, enableRawMode() now gets fully into rawmode. Let's add some error handling.

Most C library functions that fail will set the global *errno* (from errno.h) variable and prints a descriptive error for it. perror() (from stdio.h) looks at the *errno* global variable and printfs a descriptive error for it.


```c
void die(const char *s) {
  perror(s);
  exit(1);
}
```

After print the error, we exit the program.

Now, we have to modify our code to implement this error handling, for that, we know that all the variables returns an error code if something goes wrong, so we can match that code and trigger our error function:

```c
#include <ctype.h>
#include <errno.h> //Error library
#include <stdio.h> //Input-output library
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>

struct termios orig_termios;
void die(const char *s) { 
  perror(s);
  exit(1);
}

void disableRawMode() {
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &orig_termios) == -1) //Error handling
    die("tcsetattr");
}

void enableRawMode() {
  if (tcgetattr(STDIN_FILENO, &orig_termios) == -1) die("tcgetattr"); //Error handling
  atexit(disableRawMode);
  struct termios raw = orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw) == -1) die("tcsetattr"); //Error handling
}

int main() {
  enableRawMode();
  while (1) {
    char c = '\0';
    if (read(STDIN_FILENO, &c, 1) == -1 && errno != EAGAIN) die("read"); //EAGAIN (Resource temporarily unavailable). Handle empty input operation.
    if (iscntrl(c)) {
      printf("%d\r\n", c);
    } else {
      printf("%d ('%c')\r\n", c, c);
    }
    if (c == 'q') break;
  }
  return 0;
}
```

On the last part, we also add a second condition, 'errno != EAGAIN', this is to prevent that read() trying to read an empty resource triggers an error since this is not a code error.
<br>

### 2. Raw Input/Output.

#### 2.1. Press Ctrl-Q to quit.

We see that 'Ctrl' key combined with the alphabetic keys seemed to map to bytes 1–26. We can use this to detect Ctrl key combinations and map them to different operations in our editor.

Our editor closes when we press 'Q', but the ideal sceneario is to close when we press 'Ctrl + Q'. This input the value 17 to the program but a line like:

```c
if (c == 17) break;
```

Would be incomprehensive, so for clearity purposes we define the following macro:

```c
#define CTRL_KEY(k) ((k) & 0x1f)
```

This macro just performs an AND operation over the bytes of an arbitrary key with the mask 0001111 (0x1f in hexadecimal, in C, you generally specify bitmasks using hexadecimal, since C doesn’t have binary literals, and hexadecimal is more concise and readable once you get used to it.). This operation sets the upper 3 bits of the character to 0 mirroring what Ctrl key does in the terminal.

Now, we can replace the quit sentence with a more comprehensive line:

```c
if (c == CTRL_KEY('q')) break;
```

<br>

#### 2.2. Refractor keyboard input.

Now we are going to make a function for low-level keypress reading and another function for mapping keypress to editor operations:

- **editorReadKey()**: This function's job is to wait for one keypress, and return it. Later, we’ll expand this function to handle escape sequences, which involves reading multiple bytes that represent a single keypress, as is the case with the arrow keys.

```c
char editorReadKey() {
  int nread;
  char c;

  //Assign the read output to nread at the time, read waits for user's input.
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  return c;
}
```


- **editorProcessKeypress()**: This functions waits for a keypress, and then handles it. Later, it will map various Ctrl key combinations and other special keys to different editor functions, and insert any alphanumeric and other printable keys’ characters into the text that is being edited.


```c
void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      exit(0);
      break;
  }
}
```

<br>

#### 2.3. Clear the screen.

Now we are going to start the renderization of the editor's user interface after each keypress. Let's start by clearing the terminal. In order to do that, we implement the following function:

```c
#include <unistd.h>

void editorRefreshScreen() {
  write(STDOUT_FILENO, "\x1b[2J", 4);
}
```

In summary, we are writing 4 bytes out to the terminal. The thing we are writting is called 'ANSI escape sequence'; this a special code that controls terminal behavior instead of displaying text. 

The first byte is '\\x1b', which is the escape character, or 27 in decimal. (Try and remember \x1b, we will be using it a lot.) The other three bytes are '\[2J'.

- The '\[' character is included because the escape character always is continued with '\['. 
- The J is the command we are triggering which clears the screen.
- The 2 is the argument of the J command, which tells J to clear the entire Screen. (1 would clear the screen up to where the cursor is, and 0 would clear the screen from the cursor up to the end of the screen. Also, 0 is the default argument for J).

<br>

#### 2.4. Reposition the cursor.

The '\<esc\>\[2J' command left the cursor at the bottom of the screen. Let’s reposition it at the top-left corner so that we’re ready to draw the editor interface from top to bottom.

The following line reposition the cursor on the left-top side:

```c
write(STDOUT_FILENO, "\x1b[H", 3);
```

This again launch the escape sequence: '\\x1b\[' followed by the command 'H' (Cursor Position) which have two argument, the raw and the column to position the cursor which happen to be by the default 1, the position '\[1,1\]' happens to be the first row and column starting by the top and the left so we don't add arguments.

<br>

#### 2.5. Clear the screen on exit.

Let’s clear the screen and reposition the cursor when our program exits. If an error occurs in the middle of rendering the screen, we don’t want a bunch of garbage left over on the screen, and we don’t want the error to be printed wherever the cursor happens to be at that point.

So we add the two lines defined before in our *die()* function. 

```c
void die(const char *s) {
  write(STDOUT_FILENO, "\x1b[2J", 4);
  write(STDOUT_FILENO, "\x1b[H", 3);
  perror(s);
  exit(1);
}
```

Also, when we exit:

```c
void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
  }
}
```

Is worth to mention that we could use *atexit()* to clear the screen when our program exits, but then the error message printed by *die()* would get erased right after printing it.

<br>

#### 2.6. Tildes.

 Let’s draw a column of tildes (~) on the left hand side of the screen, like vim does. In our text editor, we’ll draw a tilde at the beginning of any lines that come after the end of the file being edited.

```c
void editorDrawRows() {
  int y;
  for (y = 0; y < 24; y++) {
    write(STDOUT_FILENO, "~\r\n", 3); //We remember that since we remove OPOST from c_oflag, '\n' is now '\r\n'.
  }
}

void editorRefreshScreen() {
  write(STDOUT_FILENO, "\x1b[2J", 4);
  write(STDOUT_FILENO, "\x1b[H", 3);
  editorDrawRows();
  write(STDOUT_FILENO, "\x1b[H", 3);
}
```

The function *editorDrawRows()* writes over the terminal 3 bytes '~\r\n' which is in fact a tilde '~' and a salt line.

We reproduce this over 24 lines, then we include this function on the *editorRefreshScreen()* function to get

<br>

#### 2.7. Global State.

Our next goal is to get the size of the terminal, so we know how many rows to draw in editorDrawRows(). But first, let’s set up a global struct that will contain our editor state, which we’ll use to store the width and height of the terminal. For now, let’s just put our orig_termios global into the struct:

```c
struct editorConfig {
  struct termios orig_termios;
};
struct editorConfig E;
```

Now we change our code to apply this change to the *disableRawMode()* and *enableRawMode()* function:

```c
void disableRawMode() {
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &E.orig_termios) == -1)
    die("tcsetattr");
}

void enableRawMode() {
  if (tcgetattr(STDIN_FILENO, &E.orig_termios) == -1) die("tcgetattr");
  atexit(disableRawMode);1

  struct termios raw = E.orig_termios;
  raw.c_iflag &= ~(BRKINT | ICRNL | INPCK | ISTRIP | IXON);
  raw.c_oflag &= ~(OPOST);
  raw.c_cflag |= (CS8);
  raw.c_lflag &= ~(ECHO | ICANON | IEXTEN | ISIG);
  raw.c_cc[VMIN] = 0;
  raw.c_cc[VTIME] = 1;
  if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw) == -1) die("tcsetattr");
}
```

If we fail to get or set the original atributes from the *editorConfig* struct, we display an error alluding to *E.orig_termios*.

<br>

#### 2.8. Window size, the easy way.

On most systems, you should be able to get the size of the terminal by simply calling *ioctl()* with the *TIOCGWINSZ* (Terminal IOCtl (Input/Output Control) Get WINdow SiZe) request.

```c
#include <sys/ioctl.h>
int getWindowSize(int *rows, int *cols) {
  struct winsize ws;
  if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    return -1;
  } else {
    *cols = ws.ws_col;
    *rows = ws.ws_row;
    return 0;
  }
}
```

This syscall is defined in *sys/ioctl.h* header for device-specific input/output operations (a device in linux is any object from we can read or we can write in, typically /dev/). With the *TIOCGWINSZ* request code it will dump the parameters of the deviced select by the first argument (the terminal in this case) on the structure 'ws'. Then, we get the number of columns and rows from the terminal.

On failure, ioctl() returns -1.

Now, we can fill the terminal exactly with the dimension of the terminal window.

```c
#include <sys/ioctl.h>

struct editorConfig {
  int screenrows;
  int screencols;
  struct termios orig_termios;
};

int getWindowSize(int *rows, int *cols) {
  struct winsize ws;
  if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    return -1;
  } else {
    *cols = ws.ws_col;
    *rows = ws.ws_row;
    return 0;
  }
}

void initEditor() {
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}

void editorDrawRows() {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    write(STDOUT_FILENO, "~\r\n", 3);
  }
}

int main() {
  enableRawMode();
  initEditor();
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

<br>

#### 2.9. Windows size, the hard way.

ioctl() isn’t guaranteed to be able to request the window size on all systems, so we are going to provide a fallback method of getting the window size.

The strategy is to position the cursor at the bottom-right of the screen, then use escape sequences that let us query the position of the cursor. That tells us how many rows and columns there must be on the screen.

First, if the *ioctl()* syscalls fails, we use two ANSI escape sequences:

- '\\x1b\[999C'; Which uses the C command after the escape sequence to move 999 positions to the right.
- '\\x1b\[999C'; Which uses the B command after the escape sequence to move 999 positions down.


```c
if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
  if (1 || write(STDOUT_FILENO, "\x1b[999C\x1b[999B", 12) != 12) return -1;
  editorReadKey();
  return -1;
} 
```

With this is almost sure that the cursos will end to the right and down as much as posible, then we only have to query the cursor position.

The reason we don’t use the '999;999H' command is that the documentation doesn’t specify what happens when you try to move the cursor off-screen, but the C and B commands are specifically documented to stop the cursor from going past the edge of the screen

When you run the program, you should see the cursor is positioned at the bottom-right corner of the screen, and then when you press a key you’ll see the error message printed by die() after it clears the screen.

Now we get, the cursor position. The *n* command (Device Status Report) can be used to query the terminal for status information. We want to give it an argument of 6 to ask for the cursor position:

```c
int getCursorPosition(int *rows, int *cols) {
  char buf[32];
  unsigned int i = 0;

  //Enter ANSI escape sequence to the terminal.
  if (write(STDOUT_FILENO, "\x1b[6n", 4) != 4) return -1;

  //Read terminal's response and the ANSI escape sequence reply to buf.
  while (i < sizeof(buf) - 1) {
    if (read(STDIN_FILENO, &buf[i], 1) != 1) break;
    if (buf[i] == 'R') break;
    i++;
  }
  buf[i] = '\0';

  //Check the format of the string stored on buf.
  if (buf[0] != '\x1b' || buf[1] != '[') return -1;

  //Use sscanf() to enter those values on rows and cols.
  if (sscanf(&buf[2], "%d;%d", rows, cols) != 2) return -1;
  return 0;
}

int getWindowSize(int *rows, int *cols) {
  struct winsize ws;
  if (1 || ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1 || ws.ws_col == 0) {
    if (write(STDOUT_FILENO, "\x1b[999C\x1b[999B", 12) != 12) return -1;
    return getCursorPosition(rows, cols);
  } else {
    *cols = ws.ws_col;
    *rows = ws.ws_row;
    return 0;
  }
}
```

First, *getWindowSize()* (which is called with the pointers of rows and cols in order to be able to modify this variables with the result) attempt to use *ioctl()* syscall, if fails calls *getCursorPosition()* and pass to it the pointers it selves.

- This function try to write the following ANSI escape sequence "6n" to the terminal asking for the cursor position:

  ```c
  if (write(STDOUT_FILENO, "\x1b[6n", 4) != 4) return -1;
  ```

- The terminal replies to the STDIN with another escape sequence something "\\x1b\[24;80R". We read this reply with read() function and store it in *buf\[i\]*:

  ```c
  while (i < sizeof(buf) - 1) {
    if (read(STDIN_FILENO, &buf[i], 1) != 1) break;
    if (buf[i] == 'R') break;
    i++;
  }
  buf[i] = '\0';
  ```

  This loop reads from the STDIN to *buf[]* until the R char appears and then breaks, then sets the last slot as the string terminator char: '\0'.


- Lastly, we check the format of the escape sequence string in *buf[]* 

  ```c
  if (buf[0] != '\x1b' || buf[1] != '[') return -1;
  ```

- And use *sscanf()* to read two integers separate by a semicolon ("%d;%d"), from the buf\[2\] position of the array and store each integer in rows and cols respectively:

  ```c
  if (sscanf(&buf[2], "%d;%d", rows, cols) != 2) return -1;
  ```

  sscanf() reads buf starting from the third slot (buf\[2\]) and expect to encounter to integers separate by a semicolon "%d;%d". Then introduce each integer in each variables (rows and cols).

<br>

#### 2.10. Append buffer.

It’s not a good idea to make a whole bunch of small write()’s every time we refresh the screen. It would be better to do one big write(), to make sure the whole screen updates at once. Otherwise there could be small unpredictable pauses between write()’s, which would cause an annoying flicker effect.

We want to replace all our write() calls with code that appends the string to a buffer, and then write() this buffer out at the end. Unfortunately, C doesn’t have dynamic strings, so we’ll create our own dynamic string type that supports one operation: appending.

Let’s start defining the abuf struct under it.

```c
#define ABUF_INIT {NULL, 0}

struct abuf {
  char *b;
  int len;
};
```

Now, we define the abAppend() operation, abFree():


```c
#include <string.h>
#define ABUF_INIT {NULL, 0}

struct abuf {
  char *b;
  int len;
};

void abAppend(struct abuf *ab, const char *s, int len) {
  char *new = realloc(ab->b, ab->len + len);
  if (new == NULL) return;
  memcpy(&new[ab->len], s, len);
  ab->b = new;
  ab->len += len;
}

void abFree(struct abuf *ab) {
  free(ab->b);
}
```

In summary:

- First, we define the struct which consist in a pointer to char (string) and an integer which is presumibly the length of the string.
- Then, we define a function that appends to a struct like before a new. For that:

  - Gets the buff to append, the new string and the length of the new string.
  - Then, with realloc, we ask for a block of memory that is the size of the current string plus the size of the string we are appending:

    ```c
    char *new = realloc(ab->b, ab->len + len);
    if (new == NULL) return; //Check realloc work properly.
    ```

    This gets the pointer 'b' from the struct and redefine the alloced space for 'b' with the space before plus the len of the new string.

  - Now, copy over the new alloced region the contents of the new string using memcpy and change the fields fo the struct:

    ```c
    memcpy(&new[ab->len], s, len);
    ab->b = new;
    ab->len += len;
    ```

This effectively place the string on the buffer in a bigger region and append to it the new string.

On the other hand, abFree() is a destructor that deallocates teh dynamic memory used by an abuf.

Now we can introduce this functionality in *editorDrawRows()* and *editorRefreshScreen()*:


```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    abAppend(ab, "~", 1);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
void editorRefreshScreen() {

  //Define struct.
  struct abuf ab = ABUF_INIT;

  //Append escape sequence to clear screen.
  abAppend(&ab, "\x1b[2J", 4);

  //Append escape sequence to move the cursor top-left.
  abAppend(&ab, "\x1b[H", 3);

  //Append the tildes.
  editorDrawRows(&ab);

  //Append the cursor again to the top-left 
  abAppend(&ab, "\x1b[H", 3);

  //Then, write the appended buffer to the terminal and free.
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

<br>

#### 2.11. Hide the cursor when repainting.

Let’s hide the cursor before refreshing the screen, and show it again immediately after the refresh finishes. In order to do that, we are going to introduce the following lines:

```c
abAppend(&ab, "\x1b[?25l", 6);
abAppend(&ab, "\x1b[?25h", 6);
```

The *h* and *l* commands are used to turn on and turn off various terminal features, the '?25' refers to the cursor. 

<br>

#### 2.12. Clear lines one at a time.

Instead of clearing the entire screen before each refresh, it seems more optimal to clear each line as we redraw them. Let’s remove the '\[2J' (clear entire screen) escape sequence, and instead put a '\[K' sequence at the end of each line we draw.


```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    abAppend(ab, "~", 1);
    abAppend(ab, "\x1b[K", 3); // Write line be line
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}

void editorRefreshScreen() {
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);

  //Rewrite entire screen removed
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  abAppend(&ab, "\x1b[H", 3);
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

<br>

#### 2.13. Welcome message.

Let’s display the name of our editor and a version number a third of the way down the screen:

```c
#define KILO_VERSION "0.0.1"

void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {

    //If the line is a third of the total.
    if (y == E.screenrows / 3) {
      char welcome[80];

      //Introduce the string into a buffer 'welcome' and returns the number of characters that would be written.
      int welcomelen = snprintf(welcome, sizeof(welcome),
        "Kilo editor -- version %s", KILO_VERSION);

      //We check the length of the welcome message fits in one single line, if dont fit we trunct the welcome message to fit within the  current window with.
      if (welcomelen > E.screencols) welcomelen = E.screencols;
      abAppend(ab, welcome, welcomelen);
    } else {
      abAppend(ab, "~", 1);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

snprintf() comes from *stdio.h*.

We use the welcome buffer and snprintf() to interpolate our KILO_VERSION string into the welcome message. We also truncate the length of the string in case the terminal is too tiny to fit our welcome message.

Now, we try to center it:

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    if (y == E.screenrows / 3) {
      char welcome[80];
      int welcomelen = snprintf(welcome, sizeof(welcome),
        "Kilo editor -- version %s", KILO_VERSION);
      if (welcomelen > E.screencols) welcomelen = E.screencols;
      int padding = (E.screencols - welcomelen) / 2;
      if (padding) {
        abAppend(ab, "~", 1);
        padding--;
      }
      while (padding--) abAppend(ab, " ", 1);
      abAppend(ab, welcome, welcomelen);
    } else {
      abAppend(ab, "~", 1);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

<br>

#### 2.14. Move the cursor.

Let’s focus on input now. We want the user to be able to move the cursor around. The first step is to keep track of the cursor’s x and y position in the global editor state.

```c
struct editorConfig {
  int cx, cy;
  int screenrows;
  int screencols;
  struct termios orig_termios;
};

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
```

E.cx is the horizontal coordinate of the cursor (the column) and E.cy is the vertical coordinate (the row). We initialize both of them to 0, as we want the cursor to start at the top-left of the screen. (Since the C language uses indexes that start from 0, we will use 0-indexed values wherever possible.)

Now let’s add code to editorRefreshScreen() to move the cursor to the position stored in E.cx and E.cy.

```c
void editorRefreshScreen() {
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  char buf[32];

  //We introduce on buf the escape sequence of the position of the cursor (top-left), and then append the buf.
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", E.cy + 1, E.cx + 1);
  abAppend(&ab, buf, strlen(buf));

  //Append the escape sequence to return the position of the buff and write the buffer and free.
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

This will be effectively move the cursor to the position stored in E.cy and E.cx.

Now we are going to implement a code that allows the user to move:

```c
void editorMoveCursor(char key) {
  switch (key) {
    case 'a':
      E.cx--;
      break;
    case 'd':
      E.cx++;
      break;
    case 'w':
      E.cy--;
      break;
    case 's':
      E.cy++;
      break;
  }
}

void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case 'w':
    case 's':
    case 'a':
    case 'd':
      editorMoveCursor(c);
      break;
  }
}
```

- First, we read the keypress:

  ```c
  char c = editorReadKey();
  ```

- Then, we implement a switch statement:

  ```c
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case 'w':
    case 's':
    case 'a':
    case 'd':
      editorMoveCursor(c);
      break;
  }
  ```

  - 'Ctrl-q' clears the screen and return the cursor to the top left of the screen.
  - 'w','s','a',d' calls *editorMoveCursor()*

The *editorMoveCursor()* function controls the cursor coordinates by manipulating *E.cx* and *E.cy* by one.

<br>

#### 2.15. Arrow keys.

Now that we have a way of mapping keypresses to move the cursor, let’s replace the wasd keys with the arrow keys. Last chapter we saw that pressing an arrow key sends multiple bytes as input to our program. These bytes are in the form of an escape sequence that starts with '\x1b', '\[', followed by an 'A', 'B', 'C', or 'D' depending on which of the four arrow keys was pressed. 

Let’s modify editorReadKey() to read escape sequences of this form as a single keypress.

```c
char editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      switch (seq[1]) {
        case 'A': return 'w';
        case 'B': return 's';
        case 'C': return 'd';
        case 'D': return 'a';
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
```

The code acts as follows:

- First, read one byte from the STDIN. If the byte readed is the start of an escape sequence then read the next bytes, if not we return the readed char.
- If the second byte is the second part of the escape sequence ('\['), then read the third byte.
- Then, as we said above, if the third byte is A,B,C,D we return w,s,d,a if not if either of these reads time out (after 0.1 seconds), then we assume the user just pressed the Escape key and return that.

Note that the seq buffer is slightly longer than we need (two bytes longer), this is because we will be handling longer escape sequences in the future.

We have basically aliased the arrow keys to the wasd keys. This gets the arrow keys working immediately, but leaves the wasd keys still mapped to the editorMoveCursor() function. What we want is for editorReadKey() to return special values for each arrow key that let us identify that a particular arrow key was pressed.

Let’s start by replacing each instance of the wasd characters with the constants ARROW_UP, ARROW_LEFT, ARROW_DOWN, and ARROW_RIGHT:

```c
/*** defines ***/
enum editorKey {
  ARROW_LEFT = 'a',
  ARROW_RIGHT = 'd',
  ARROW_UP = 'w',
  ARROW_DOWN = 's'
};
```

An enum defines a set of named integer constants, we use them into the code defined before:

```c
char editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      switch (seq[1]) {
        case 'A': return ARROW_UP;
        case 'B': return ARROW_DOWN;
        case 'C': return ARROW_RIGHT;
        case 'D': return ARROW_LEFT;
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}

void editorMoveCursor(char key) {
  switch (key) {
    case ARROW_LEFT:
      E.cx--;
      break;
    case ARROW_RIGHT:
      E.cx++;
      break;
    case ARROW_UP:
      E.cy--;
      break;
    case ARROW_DOWN:
      E.cy++;
      break;
  }
}

void editorProcessKeypress() {
  char c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
```

Now we just have to choose a representation for arrow keys that doesn’t conflict with wasd, in the editorKey enum. We will give them a large integer value that is out of the range of a char, so that they don’t conflict with any ordinary keypresses.

```c
enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN
};
```

By setting the first constant in the enum to 1000, the rest of the constants get incrementing values of 1001, 1002, 1003, and so on.

We will also have to change all variables that store keypresses to be of type int instead of char.

```c
int editorReadKey()
void editorMoveCursor(int key)
int c = editorReadKey();
```

<br>

#### 2.16. Prevent moving the cursor off screen.

Currently, you can cause the E.cx and E.cy values to go into the negatives, or go past the right and bottom edges of the screen. Let’s prevent that by doing some bounds checking in editorMoveCursor():

```c
void editorMoveCursor(int key) {
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      if (E.cx != E.screencols - 1) {
        E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy != E.screenrows - 1) {
        E.cy++;
      }
      break;
  }
}
```

<br>

#### 2.17. The Page Up and Page Down keys.

To complete our low-level terminal code, we need to detect a few more special keypresses that use escape sequences, like the arrow keys did. We’ll start with the Page Up and Page Down keys.

Page Up is sent as '5~' and Page Down is sent as '6~'. First we add to our enum list:

```c
enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN,
  PAGE_UP,
  PAGE_DOWN
};
```

Now we modify the *editorReadKey()* function:

```c
int editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      if (seq[1] >= '0' && seq[1] <= '9') {
        if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
        if (seq[2] == '~') {
          switch (seq[1]) {
            case '5': return PAGE_UP;
            case '6': return PAGE_DOWN;
          }
        }
      } else {
        switch (seq[1]) {
          case 'A': return ARROW_UP;
          case 'B': return ARROW_DOWN;
          case 'C': return ARROW_RIGHT;
          case 'D': return ARROW_LEFT;
        }
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
```

We modify the part in which we detect the user's input is an escape sequence and introduce a new if statement:

```c
if (seq[1] >= '0' && seq[1] <= '9') {
  if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
  if (seq[2] == '~') {
    switch (seq[1]) {
      case '5': return PAGE_UP;
      case '6': return PAGE_DOWN;
    }
  }
} else {
  switch (seq[1]) {
    case 'A': return ARROW_UP;
    case 'B': return ARROW_DOWN;
    case 'C': return ARROW_RIGHT;
    case 'D': return ARROW_LEFT;
  }
}
```

- First, detect the next byte and if is between 0 and 9, we read the next byte, if not, we assume is an ARROW KEY.
- Then, we check the next byte and see if it is a '~'.
- If it is, then read the byte before and check if it is a 5 or a 6 and then we return PAGE_UP and PAGE_DOWN.

This efectively catch when we press PAGE_UP and PAGE_DOWN and makes the function *editorReadKey()* to effectively return it.

Now, we have to define the behaviour of our terminal when we press PAGE_UP and PAGE_DOWN:

```c
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
```

This introduces on the switch statement the PAGE_UP and the PAGE_DOWN cases in the same way (since there is no break between them).

Then, it introduces the number of rows of the screen *E.screenrows* on times and implements a while loop in which, while times is greater than 0 (0 is equals to False), calls *editorMoveCursor()* with the following line:

```c
c == PAGE_UP ? ARROW_UP : ARROW_DOWN
```

This line performs as follows: if 'c' is PAGE_UP, then the statement takes the value of ARROW_UP (which remember is an enum value defined above) and if not, takes the value ARROW_DOWN.

<br>

#### 2.18. The Home and End keys.

Now let’s implement the Home and End keys. Like the previous keys, these keys also send escape sequences. Unlike the previous keys, there are many different escape sequences that could be sent by these keys, depending on your OS, or your terminal emulator. 

The Home key could be sent as '\[1~', '\[7~', '\[H', or 'OH'. Similarly, the End key could be sent as '\[4~', '\[8~', '\[F', or 'OF'. 

First, we define the enum values:

```c
enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN,
  HOME_KEY,
  END_KEY,
  PAGE_UP,
  PAGE_DOWN
};
```

Now we implement the catch of those enum values on *editorReadKey()*:

```c
int editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      if (seq[1] >= '0' && seq[1] <= '9') {
        if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
        if (seq[2] == '~') {
          switch (seq[1]) {
            case '1': return HOME_KEY;
            case '4': return END_KEY;
            case '5': return PAGE_UP;
            case '6': return PAGE_DOWN;
            case '7': return HOME_KEY;
            case '8': return END_KEY;
          }
        }
      } else {
        switch (seq[1]) {
          case 'A': return ARROW_UP;
          case 'B': return ARROW_DOWN;
          case 'C': return ARROW_RIGHT;
          case 'D': return ARROW_LEFT;
          case 'H': return HOME_KEY;
          case 'F': return END_KEY;
        }
      }
    } else if (seq[0] == 'O') {
      switch (seq[1]) {
        case 'H': return HOME_KEY;
        case 'F': return END_KEY;
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
```


We add several lines of codes which can be separates in three blocs:

- First, considering the cases in which can be: '\[1~', '\[7~' and '\[4~', '\[8~', then we reuse the if statement that check if the next user byte is an integer between 1 and 9 and add the following cases.


  ```c
  if (seq[1] >= '0' && seq[1] <= '9') {
    if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
    if (seq[2] == '~') {
      switch (seq[1]) {
        case '1': return HOME_KEY;
        case '4': return END_KEY;
        case '5': return PAGE_UP;
        case '6': return PAGE_DOWN;
        case '7': return HOME_KEY;
        case '8': return END_KEY;
      }
    }
  }
  ```

- Second, we consider the cases in which can be '\[H' and '\[F', and we reuse the else statement of the if before and add the cases:

  ```c
  else {
  switch (seq[1]) {
    case 'A': return ARROW_UP;
    case 'B': return ARROW_DOWN;
    case 'C': return ARROW_RIGHT;
    case 'D': return ARROW_LEFT;
    case 'H': return HOME_KEY;
    case 'F': return END_KEY;
  }
  }
  ```

- In the third case, we consider if can be 'OH' and 'OF', then in this case we can't reuse the if of the escape sequence ('\['), we have to consider if instead receiving '\[' we receive 'O' and then distinguish between 'H' or 'F':

  ```c
  else if (seq[0] == 'O') {
    switch (seq[1]) {
      case 'H': return HOME_KEY;
      case 'F': return END_KEY;
    }
  }
  ```

<br>

#### 2.19. The Delete key.

Lastly, let’s detect when the Delete key is pressed. It simply sends the escape sequence <esc>[3~, so it’s easy to add to our switch statement.


```c

enum editorKey {
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN,
  DEL_KEY,
  HOME_KEY,
  END_KEY,
  PAGE_UP,
  PAGE_DOWN
};

int editorReadKey() {
  int nread;
  char c;
  while ((nread = read(STDIN_FILENO, &c, 1)) != 1) {
    if (nread == -1 && errno != EAGAIN) die("read");
  }
  if (c == '\x1b') {
    char seq[3];
    if (read(STDIN_FILENO, &seq[0], 1) != 1) return '\x1b';
    if (read(STDIN_FILENO, &seq[1], 1) != 1) return '\x1b';
    if (seq[0] == '[') {
      if (seq[1] >= '0' && seq[1] <= '9') {
        if (read(STDIN_FILENO, &seq[2], 1) != 1) return '\x1b';
        if (seq[2] == '~') {
          switch (seq[1]) {
            case '1': return HOME_KEY;
            case '3': return DEL_KEY;
            case '4': return END_KEY;
            case '5': return PAGE_UP;
            case '6': return PAGE_DOWN;
            case '7': return HOME_KEY;
            case '8': return END_KEY;
          }
        }
      } else {
        switch (seq[1]) {
          case 'A': return ARROW_UP;
          case 'B': return ARROW_DOWN;
          case 'C': return ARROW_RIGHT;
          case 'D': return ARROW_LEFT;
          case 'H': return HOME_KEY;
          case 'F': return END_KEY;
        }
      }
    } else if (seq[0] == 'O') {
      switch (seq[1]) {
        case 'H': return HOME_KEY;
        case 'F': return END_KEY;
      }
    }
    return '\x1b';
  } else {
    return c;
  }
}
```

For now we don't make this key to do anything.

<br>

### 3. A text viewer.

#### 3.1. A line viewer

Let’s create a data type for storing a row of text in our editor. This will consist in an integer as *size*, and the string of chars.

```c
typedef struct erow {
  int size;
  char *chars;
} erow;
```

*erow* stands for “editor row”, and stores a line of text as a pointer to the dynamically-allocated character data and a length. The typedef lets us refer to the type as erow instead of struct erow.

Also, we change the editorConfig struct and initEditor:

```c
struct editorConfig {
  int cx, cy;
  int screenrows;
  int screencols;
  int numrows;
  erow row;
  struct termios orig_termios;
};

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.numrows = 0;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
```

Let’s fill that erow with some text now. We won’t worry about reading from a file just yet. Instead, we’ll hardcode a “Hello, world” string into it.

```c
#include <sys/types.h>

/*** file i/o ***/
void editorOpen() {
  char *line = "Hello, world!";
  ssize_t linelen = 13;
  E.row.size = linelen;
  E.row.chars = malloc(linelen + 1);
  memcpy(E.row.chars, line, linelen);
  erow.chars[linelen] = '\0';
  E.numrows = 1;
}

int main() {
  enableRawMode();
  initEditor();
  editorOpen();
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

We include the *sys/types.h* header file to make use of *ssize_t* (an unsigned integer type capable of storing values in the range [0, SIZE_MAX]). 

Then, we define the *editorOpen()* function in which we use *memcpy()* function to copy the contents from a memory block to another taking three arguments, the destination, the source and the size of the contents to be passed.

Line by line:

- We hardcode the string "Hello, world!":

  ```c
  char *line = "Hello, world!";
  ```

- We sett the size of the string:

  ```c
  ssize_t linelen = 13;
  ```

  Note that this lenthg is not the complete lenthg of the string since we are not including the null byte. Is just to pass the size to memcpy.

- We create the memory block for the destination:

  ```c
  E.row.chars = malloc(linelen + 1);
  ```
  
  - Then, we copy the memory block from *line* to *E.row.chars* and include the null byte:


  erow.chars[linelen] = '\0';
  ```

- Lastly we sett the number of bytes to be printed to one, this have more importance in the implementation on the next block of code.

  ```c
  memcpy(E.row.chars, line, linelen);
  E.numrows = 1;
  ```

Now, we display it:

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    if (y >= E.numrows) {
      if (y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row.size;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, E.row.chars, len);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

We modify our *editorDrawRows()* and include a new *if* statement in In which we check first is there are lines to be drawed with the *E.numrows* data setted in the *editorOpen()* section:

```c
if (y >= E.numrows) {
```

If there are we append the contents of the memory block to the buffer:

```c
} else {
  int len = E.row.size;
  if (len > E.screencols) len = E.screencols;
  abAppend(ab, E.row.chars, len);
}
```

This is on the suppose there only is one line.

Let's consider the following code:

```c
void editorOpen(char *filename) {
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  linelen = getline(&line, &linecap, fp);
  if (linelen != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    E.row.size = linelen;
    E.row.chars = malloc(linelen + 1);
    memcpy(E.row.chars, line, linelen);
    E.row.chars[linelen] = '\0';
    E.numrows = 1;
  }
  free(line);
  fclose(fp);
}

int main(int argc, char *argv[]) {
  enableRawMode();
  initEditor();
  if (argc >= 2) {
    editorOpen(argv[1]);
  }
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

This two bloque of codes tell us how the program open a file and read its contens and how this block is implemented in the main function.

The first block applies the logic of the first version of our *openEditor()* function using *memcpy()* to move contents between memory blocks and expand it to read a file line by line getting the necesary information using *getline()* (which comes from *stdio.h*)

- First, we open a file with *fopen* in read mode (with only read permissions). Then check the value of the pointer *fp*; fopen returns NULL if the operation fails and NULL is asociated with FALSE:

  ```c
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  ```

  So, we check if the negative boolean value of the pointer is TRUE, this is, if the pointer is false, we call *die()*

- Then, with *getline()* we get the attributes necesaries to pass the string to *memcpy()*, the *getline()* function reads an entire line from a stream (fp), storing the address of the buffer containing the text into *line* which is line is a pointer to a char pointer (char \*\*) and *linecap* are the bytes in size of the string (If the buffer is not large enough to hold the line, getline() resizes it with realloc(), updating *line* and *linecap* as necessary). The buffer is null-terminated and includes the newline character, if one is found. 

  *getline()* returns the number of characters read (including the newline but excluding the null terminator), or -1 on failure/EOF.

  In our case, when *line* is NULL and *linecap* is 0, *getline()* allocates the buffer automatically.

  ```c
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  linelen = getline(&line, &linecap, fp);
  ```

- Next, we check if the return value of the getline() function to verify if everything has gone alright.

  ```c
   if (linelen != -1) {
  ```
 
- And strip newlines and carriage characters:

  ```c
  while (linelen > 0 && (line[linelen - 1] == '\n' ||
                       line[linelen - 1] == '\r'))
  linelen--;
  ```

- Then, we apply the logic of the previous *editorOpen()* version:

  ```c
  E.row.size = linelen;
  E.row.chars = malloc(linelen + 1);
  memcpy(E.row.chars, line, linelen);
  E.row.chars[linelen] = '\0';
  E.numrows = 1;
  ```

- Finally we free the line pointer and close the file.


Now, on the *main()* function we just leave the user the option to use a file among with the editor. If we detect that the kilo binary has benn called along with an argument, we treat the argument as the file.

```c
if (argc >= 2) {
  editorOpen(argv[1]);
}
```


So, in summary; the core of editorOpen() is the same, we just get the line and linelen values from getline() now, instead of hardcoded values.

*editorOpen()* now takes a filename and opens the file for reading using fopen(). We allow the user to choose a file to open by checking if they passed a filename as a command line argument. If they did, we call editorOpen() and pass it the filename. If they ran ./kilo with no arguments, editorOpen() will not be called and they’ll start with a blank file.

*getline()* is useful for reading lines from a file when we don’t know how much memory to allocate for each line. It takes care of memory management for you. First, we pass it a null line pointer and a linecap (line capacity) of 0. That makes it allocate new memory for the next line it reads, and set line to point to the memory, and set linecap to let you know how much memory it allocated.

Later, when we have editorOpen() read multiple lines of a file, we will be able to feed the new line and linecap values back into getline() over and over, and it will try and reuse the memory that line points to as long as the linecap is big enough to fit the next line it reads. For now, we just copy the one line it reads into E.row.chars, and then free() the line that getline() allocated.

We also strip off the newline or carriage return at the end of the line before copying it into our erow. We know each erow represents one line of text, so there’s no use storing a newline character at the end of each one.

Now let’s fix a quick bug. We want the welcome message to only display when the user starts the program with no arguments, and not when they open a file, as the welcome message could get in the way of displaying the file.

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    if (y >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) { //We change the condition to display if only there is no lines to draw from E.numrows.
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row.size;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, E.row.chars, len);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

<br>

#### 3.2. Multiple lines.

The code before just prints the first line of the document. Let's try to store multiples lines by making *E.row* an array of *erow* structs. 

It will be a dynamically-allocated array, so we’ll make it a pointer to erow, and initialize the pointer to NULL. (This will break a bunch of our code that doesn’t expect E.row to be a pointer, so the program will fail to compile for the next few steps.):

```c
struct editorConfig {
  int cx, cy;
  int screenrows;
  int screencols;
  int numrows;
  erow *row; //Defined as pointer.
  struct termios orig_termios;
};

struct editorConfig E;

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.numrows = 0;
  E.row = NULL; //Initiate as NULL.
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
```

Next, let’s move the code in editorOpen() that initializes E.row to a new function called editorAppendRow() that we call through the *editorOpen()*: 

```c
void editorAppendRow(char *s, size_t len) {
  E.row.size = len;
  E.row.chars = malloc(len + 1);
  memcpy(E.row.chars, s, len);
  E.row.chars[len] = '\0';
  E.numrows = 1;
}

void editorOpen(char *filename) {
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  linelen = getline(&line, &linecap, fp);
  if (linelen != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    editorAppendRow(line, linelen);
  }
  free(line);
  fclose(fp);
}
```
Notice that we renamed the line and linelen variables to s and len, which are now arguments to editorAppendRow().

At this point, editorAppendRow() just works for only one line since the functionality has not been modified. In order to be able to store varios lines from the file, ew want editorAppendRow() to allocate space for a new erow, and then copy the given string to a new erow at the end of the E.row array:

```c
void editorAppendRow(char *s, size_t len) {
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  int at = E.numrows;
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.numrows++;
}
```

Now, we used the first line to realloc the E.row size to adjust several erows (exactly to get one more line). Then, we copy the contents of the string *s* to the last alloced space in the E.row slot. 

Next, let's update *editorDrawRows()* to use "E.row\[y\]":

```c

void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    if (y >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[y].size;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, E.row[y].chars, len);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

And modify *editorOpen()* to keep reading lines from the file descriptor until there is nothing more to read:

```c
void editorOpen(char *filename) {
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  while ((linelen = getline(&line, &linecap, fp)) != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    editorAppendRow(line, linelen);
  }
  free(line);
  fclose(fp);
}
```

For that, we just added a while loop that keep reading from fp until the function getline returns an error. getline() reads from a file descriptor until finds a terminator byte '\n', then save the position and, in the next call it continous from there until the next terminator byte, that is what the function consider different lines

<br>

#### 3.3. Vertical scrolling.

However, there is a problem at this point, despite the lines of the file we try to include in our editor, this one will only show the number of lines of the file that fits with the size of the terminal.

In order to change this, and be able to show more lines in our editor and scroll down the file we add a *rowoff* (row offset) variable to the global editor state, which will keep track of what row of the file the user is currently scrolled to.g

```c
struct editorConfig {
  int cx, cy;
  int rowoff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  struct termios orig_termios;
};

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rowoff = 0; //We initialize it to 0, which means we’ll be scrolled to the top of the file by default.
  E.numrows = 0;
  E.row = NULL;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
```

Also, we have modified *editorDrawRows()* code in order to display the correct range of lines of the file according to the value of rowoff:

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].size;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, E.row[filerow].chars, len);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

- We include a variable; *filerow*:

  ```c
  int filerow = y + E.rowoff;
  ```

- This new variable defines the row of the file we want to display, next, we include an 'if' statement, in which we try to eval if the row to be displayed (filerow) is bigger than the total number of row to be displayed (E.numrows)

  ```c
  if (filerow >= E.numrows) 
  ```

  If thisn is True it means that we already emptied our row buffer and is time now to fill the rest of the empty terminal with the '~' char and the welcome message, but if it is False, we append to the buffer each row to be displayed:

  ```c
  else {
    int len = E.row[filerow].size;
    if (len > E.screencols) len = E.screencols;
    abAppend(ab, E.row[filerow].chars, len);
  }
  ```

  In the code we retrieve the len of the line, we force to fit in the size of our window and then append the line to the buffer calling *abAppend()*.

With this we ensure that all the lines get displayed in the editor but, we still have the issue to scroll down the page to show the contents to the user. In order to do that, we define the following function *editorScroll()*. Our strategy will be to check if the cursor has moved outside of the visible window, and if so, adjust E.rowoff so that the cursor is just inside the visible window. 

```c
void editorScroll() {
  //Check if the cursor is above the visible window and displace it if it is.
  if (E.cy < E.rowoff) {
    E.rowoff = E.cy;
  }

  //Check if the cursor is past the bottom of the visible window.
  if (E.cy >= E.rowoff + E.screenrows) {
    E.rowoff = E.cy - E.screenrows + 1;
  }
}
```

We add this function to the *editorRefreshScreen()* code:

```c
void editorRefreshScreen() {
  editorScroll();
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", E.cy + 1, E.cx + 1);
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

At this point, we allow into the logic of the cursor to displace the window when we need it. Let's implement the functionality of the cursor in *editorMoveCursor()* function:

```c
void editorMoveCursor(int key) {
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      if (E.cx != E.screencols - 1) {
        E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) { //Now, the position of the cursor can be bigger than the size of the screen if there are lines of the file that get outs from the visible window.
        E.cy++;
      }
      break;
  }
}
```

Note that we can't scroll up.

This is because we are trying tom scroll up through the render file, not the screen, this means that we now have to subtract E.rowoff from the value of E.cy to include the range of values of E.cy inside the file in order to be able to scroll over it:

```c
void editorRefreshScreen() {
  editorScroll();
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", (E.cy - E.rowoff) + 1, E.cx + 1); 
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

<br>

#### 3.4. Horizontal scrolling.

Now let’s work on horizontal scrolling. We’ll implement it in just about the same way we implemented vertical scrolling. Start by adding a coloff (column offset) variable to the global editor state.

```c
struct editorConfig {
  int cx, cy;
  int rowoff;
  int coloff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  struct termios orig_termios;
};
struct editorConfig E;

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
```

To display each row at the column offset, we’ll use E.coloff as an index into the chars of each erow we display, and subtract the number of characters that are to the left of the offset from the length of the row.

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].size - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, &E.row[filerow].chars[E.coloff], len);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

Note that when subtracting E.coloff from the length, len can now be a negative number, meaning the user scrolled horizontally past the end of the line. In that case, we set len to 0 so that nothing is displayed on that line.

Now let’s update editorScroll():

```c
void editorScroll() {
  if (E.cy < E.rowoff) {
    E.rowoff = E.cy;
  }
  if (E.cy >= E.rowoff + E.screenrows) {
    E.rowoff = E.cy - E.screenrows + 1;
  }
  if (E.cx < E.coloff) {
    E.coloff = E.cx;
  }
  if (E.cx >= E.coloff + E.screencols) {
    E.coloff = E.cx - E.screencols + 1;
  }
}
```

As you can see, it is exactly parallel to the vertical scrolling code. We just replace E.cy with E.cx, E.rowoff with E.coloff, and E.screenrows with E.screencols.

Now let’s allow the user to scroll past the right edge of the screen:

```c
void editorMoveCursor(int key) {
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      if (E.cx != E.screencols - 1) {
      E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) {
        E.cy++;
      }
      break;
  }
}
```

Now let’s allow the user to scroll past the right edge of the screen.

```c
void editorMoveCursor(int key) {
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      E.cx++;
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) {
        E.cy++;
      }
      break;
  }
}
```

Next, let’s fix the cursor positioning, just like we did with vertical scrolling.

```c
void editorRefreshScreen() {
  editorScroll();
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", (E.cy - E.rowoff) + 1, (E.cx - E.coloff) + 1);
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

Now both E.cx and E.cy refer to the cursor’s position within the file, not its position on the editor's screen.

<br>

#### 3.5. Limit scrolling to the right.

So our goal with the next few steps is to limit the values of E.cx and E.cy to only ever point to valid positions in the file. Otherwise, the user could move the cursor way off to the right of a line and start inserting text there, which wouldn’t make much sense. (The only exceptions to this rule are that E.cx can point one character past the end of a line so that characters can be inserted at the end of the line, and E.cy can point one line past the end of the file so that new lines at the end of the file can be added easily.)

Let’s start by not allowing the user to scroll past the end of the current line.

```c
void editorMoveCursor(int key) {
  erow *row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      if (row && E.cx < row->size) {
        E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) {
        E.cy++;
      }
      break;
  }
}
```

- The line:

  ```c
  erow *row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];   
  ```

  Take the ternary operator '?' to perform an if operation, if E.cy >= E.numrows (which means, if the cursor is above the number of total rows) then row points to NULL, if not, it points to the current row.

- On another part;

  ```c
  case ARROW_RIGHT:
  if (row && E.cx < row->size) {
    E.cx++;
  }
  break;
  ```

  We check if row is not NULL and if the cursor is between the end and the start of a rown, then we add 1 when we press ARROW_RIGHT.

<br>

#### 3.6. Snap cursor to end of line.

The user is still able to move the cursor past the end of a line, however. They can do it by moving the cursor to the end of a long line, then moving it down to the next line, which is shorter. The E.cx value won’t change, and the cursor will be off to the right of the end of the line it’s now on.

Let’s add some code to editorMoveCursor() that corrects E.cx if it ends up past the end of the line it’s on.

```c
void editorMoveCursor(int key) {
  erow *row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      }
      break;
    case ARROW_RIGHT:
      if (row && E.cx < row->size) {
        E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) {
        E.cy++;
      }
      break;
  }

  //We are add a correction in which after every arrow keypressing, we evaluate if the cursor is within the length of the current row. 

  //First we check if the cursor is within a valid row index.
  row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];

  //If it is, we assign rowlen the size of the row in which lays the cursor, if not, the size of the row is 0 (which means is an invalid row).
  int rowlen = row ? row->size : 0;

  //Then, we evaluate if the horizontal position of the cursor fits in the length of the line, if is bigger we truncate it.
  if (E.cx > rowlen) {
    E.cx = rowlen;
  }
}
```

<br>

#### 3.7. Moving left at the start of a line.

Let’s allow the user to press ← at the beginning of the line to move to the end of the previous line.

```c
void editorMoveCursor(int key) {
  erow *row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;

      //If E.cx = 0, then we go upn and right to end of the row.
      } else if (E.cy > 0) {
        E.cy--;
        E.cx = E.row[E.cy].size;
      }
      break;
    case ARROW_RIGHT:
      if (row && E.cx < row->size) {
        E.cx++;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) {
        E.cy++;
      }
      break;
  }
  row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];
  int rowlen = row ? row->size : 0;
  if (E.cx > rowlen) {
    E.cx = rowlen;
  }
}
```

<br>

#### 3.8. Moving right at the end of a line.

Similarly, let’s allow the user to press → at the end of a line to go to the beginning of the next line.

```c
void editorMoveCursor(int key) {
  erow *row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];
  switch (key) {
    case ARROW_LEFT:
      if (E.cx != 0) {
        E.cx--;
      } else if (E.cy > 0) {
        E.cy--;
        E.cx = E.row[E.cy].size;
      }
      break;
    case ARROW_RIGHT:
      if (row && E.cx < row->size) {
        E.cx++;
      } else if (row && E.cx == row->size) {
        E.cy++;
        E.cx = 0;
      }
      break;
    case ARROW_UP:
      if (E.cy != 0) {
        E.cy--;
      }
      break;
    case ARROW_DOWN:
      if (E.cy < E.numrows) {
        E.cy++;
      }
      break;
  }
  row = (E.cy >= E.numrows) ? NULL : &E.row[E.cy];
  int rowlen = row ? row->size : 0;
  if (E.cx > rowlen) {
    E.cx = rowlen;
  }
}
```

<br>

#### 3.9. Rendering tabs.

Our code doesn't render properly the TABs. The length of a tab is up to the terminal being used and its settings and inside the editor doesn't erase any character, it only move the cursor forward to the next tab stop. 

We want to know the length of each tab, and we also want control over how to render tabs, so we’re going to add a second string to the erow struct called *render*, which will contain the actual characters to draw on the screen for that row of text. We’ll only use render for tabs for now, but in the future it could be used to render nonprintable control characters as a ^ character followed by another character, such as ^A for the Ctrl-A character.

Let’s start by adding render and *rsize* (which contains the size of the contents of render) to the erow struct, and initializing them in *editorAppendRow()*:

```c

typedef struct erow {
  int size;
  int rsize;
  char *chars;
  char *render;
} erow;

void editorAppendRow(char *s, size_t len) {
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  int at = E.numrows;
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  E.numrows++;
}
```

Next, let’s make an *editorUpdateRow()* function that uses the chars string of an erow to fill in the contents of the render string. 

```c
void editorUpdateRow(erow *row) {
  free(row->render);
  row->render = malloc(row->size + 1);
  int j;
  int idx = 0;
  for (j = 0; j < row->size; j++) {
    row->render[idx++] = row->chars[j];
  }
  row->render[idx] = '\0';
  row->rsize = idx;
}
```

This function, performs as follows:

- First, we free 'row->render' since we are updating an existing value and that pointer can contain garbage. Then, we alloc memory for the pointer again of the size of the row:

  ```c
  free(row->render);
  row->render = malloc(row->size + 1);
  ```

- Then, we copy byte to byte the contents of the row to render and add the terminator byte and add the size:

  ```c
  int j;
  int idx = 0;
  for (j = 0; j < row->size; j++) {
    row->render[idx++] = row->chars[j];
  }
  row->render[idx] = '\0';
  row->rsize = idx; //idx contains the number of characters we copied into row->render, so we assign it to row->rsize
  ```

Now, we call this function in *editorAppendRow()*.

```c
void editorAppendRow(char *s, size_t len) {
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  int at = E.numrows;
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  editorUpdateRow(&E.row[at]);
  E.numrows++;
}
```

Now let’s replace chars and size with render and rsize in editorDrawRows(), when we display each erow.

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, &E.row[filerow].render[E.coloff], len);
    }
    abAppend(ab, "\x1b[K", 3);
    if (y < E.screenrows - 1) {
      abAppend(ab, "\r\n", 2);
    }
  }
}
```

Whit this change, the text viewer is displaying the characters in render. We now control what text is displaying.

The nexst step is to add code to *editorUpdateRow()* in order to render the TAB as multiple spaces.

```c
void editorUpdateRow(erow *row) {
  int tabs = 0;
  int j;
  for (j = 0; j < row->size; j++)

    //First check if the receive char is a TAB, which is represented as the \t escape sequence. If it is, add one to the counter.
    if (row->chars[j] == '\t') tabs++;
  free(row->render);
  row->render = malloc(row->size + tabs*7 + 1); //Alloc 7 bytes more per TAB, since a TAB is 8 spaces and one byte per TAB is consider in row->size.
  int idx = 0;
  for (j = 0; j < row->size; j++) {
    //After allocating memory, we place in the TAB slot a 8-divisible chunk of spaces (formely, a TAB).
    if (row->chars[j] == '\t') {
      row->render[idx++] = ' ';
      while (idx % 8 != 0) row->render[idx++] = ' ';
    } else {
      row->render[idx++] = row->chars[j];
    }
  }
  row->render[idx] = '\0';
  row->rsize = idx;
}
```

At this point, we should probably make the length of a tab stop a constant.

```c
#define KILO_TAB_STOP 8

void editorUpdateRow(erow *row) {
  int tabs = 0;
  int j;
  for (j = 0; j < row->size; j++)
    if (row->chars[j] == '\t') tabs++;
  free(row->render);
  row->render = malloc(row->size + tabs*(KILO_TAB_STOP - 1) + 1);
  int idx = 0;
  for (j = 0; j < row->size; j++) {
    if (row->chars[j] == '\t') {
      row->render[idx++] = ' ';
      while (idx % KILO_TAB_STOP != 0) row->render[idx++] = ' ';
    } else {
      row->render[idx++] = row->chars[j];
    }
  }
  row->render[idx] = '\0';
  row->rsize = idx;
}
```

<br>

#### 3.10. Tabs and the cursor.

The cursor doesn’t currently interact with tabs very well. When we position the cursor on the screen, we’re still assuming each character takes up only one column on the screen while TAB in fact occupies 8 slots. 

To fix this, let’s introduce a new horizontal coordinate variable, *E.rx*. While *E.cx* is an index into the chars field of an erow, the E.rx variable will be an index into the render field. If there are no tabs on the current line, then E.rx will be the same as E.cx. If there are tabs, then E.rx will be greater than E.cx by however many extra spaces those tabs take up when rendered.

Start by adding rx to the global state struct, and initializing it to 0.

```c
struct editorConfig {
  int cx, cy;
  int rx;
  int rowoff;
  int coloff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  struct termios orig_termios;
};

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
}
```

We’ll set the value of E.rx at the top of editorScroll(). For now we’ll just set it to be the same as E.cx. Then we’ll replace all instances of E.cx with E.rx in editorScroll(), because scrolling should take into account the characters that are actually rendered to the screen, and the rendered position of the cursor.

```c
void editorScroll() {
  E.rx = E.cx;
  if (E.cy < E.rowoff) {
    E.rowoff = E.cy;
  }
  if (E.cy >= E.rowoff + E.screenrows) {
    E.rowoff = E.cy - E.screenrows + 1;
  }
  if (E.rx < E.coloff) {
    E.coloff = E.rx;
  }
  if (E.rx >= E.coloff + E.screencols) {
    E.coloff = E.rx - E.screencols + 1;
  }
}
```

Now change E.cx to E.rx in editorRefreshScreen() where we set the cursor position:

```c
void editorRefreshScreen() {
  editorScroll();
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", (E.cy - E.rowoff) + 1,
                                            (E.rx - E.coloff) + 1);
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

All that’s left to do is calculate the value of E.rx properly in editorScroll()

Let’s create an *editorRowCxToRx()* function that converts a chars index into a render index. 
We’ll need to loop through all the characters to the left of cx, and figure out how many spaces each tab takes up.

```c
int editorRowCxToRx(erow *row, int cx) {
  int rx = 0;
  int j;
  for (j = 0; j < cx; j++) {
    if (row->chars[j] == '\t')
      rx += (KILO_TAB_STOP - 1) - (rx % KILO_TAB_STOP);
    rx++;
  }
  return rx;
}
```

For each character, if it’s a tab we use rx % KILO_TAB_STOP to find out how many columns we are to the right of the last tab stop, and then subtract that from KILO_TAB_STOP - 1 to find out how many columns we are to the left of the next tab stop. We add that amount to rx to get just to the left of the next tab stop, and then the unconditional rx++ statement gets us right on the next tab stop. Notice how this works even if we are currently on a tab stop.

Let’s call editorRowCxToRx() at the top of editorScroll() to finally set E.rx to its proper value.

```c
void editorScroll() {
  E.rx = 0;
  if (E.cy < E.numrows) {
    E.rx = editorRowCxToRx(&E.row[E.cy], E.cx);
  }
  if (E.cy < E.rowoff) {
    E.rowoff = E.cy;
  }
  if (E.cy >= E.rowoff + E.screenrows) {
    E.rowoff = E.cy - E.screenrows + 1;
  }
  if (E.rx < E.coloff) {
    E.coloff = E.rx;
  }
  if (E.rx >= E.coloff + E.screencols) {
    E.coloff = E.rx - E.screencols + 1;
  }
}
```

<br>

#### 3.11. Scrollin with Page Up and Page Down.

Now that we have scrolling, let’s make the Page Up and Page Down keys scroll up or down an entire page.

```c
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      E.cx = E.screencols - 1;
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
```

<br>

#### 3.12. Move to the end of the line with End.

Now let’s have the End key move the cursor to the end of the current line. (The Home key already moves the cursor to the beginning of the line, since we made E.cx relative to the file instead of relative to the screen.)

```c
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
  }
}
```

<br>

#### 3.13. Status bar.

The last thing we’ll add before finally getting to text editing is a status bar. This will show useful information such as the filename, how many lines are in the file, and what line you’re currently on.

Later we’ll add a marker that tells you whether the file has been modified since it was last saved, and we’ll also display the filetype when we implement syntax highlighting.

First, we will make room for a simply one-line status bar at the bottom of the screen:

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      abAppend(ab, &E.row[filerow].render[E.coloff], len);
    }
    abAppend(ab, "\x1b[K", 3);
    abAppend(ab, "\r\n", 2);
  }
}
```

Also, we decrement E.screenrows so that editorDrawRows() doesn’t try to draw a line of text at the bottom of the screen. We have editorDrawRows() print a newline after the last row it draws, since the status bar is now the final line being drawn on the screen:

```c
void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
  E.screenrows -= 1;
}
```

Now, to make the status bar stand out, we’re going to display it with inverted colors: black text on a white background. The escape sequence '\[7m' switches to inverted colors, and '\[m' switches back to normal formatting. Let’s draw a blank white status bar of inverted space characters.

```c
void editorDrawStatusBar(struct abuf *ab) {
  abAppend(ab, "\x1b[7m", 4);
  int len = 0;
  while (len < E.screencols) {
    abAppend(ab, " ", 1);
    len++;
  }
  abAppend(ab, "\x1b[m", 3);
}
```

Then, we include the function on editorRefreshScreen():

```c
void editorRefreshScreen() {
  editorScroll();
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  editorDrawStatusBar(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", (E.cy - E.rowoff) + 1,
                                            (E.rx - E.coloff) + 1);
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

Since we want to display the filename in the status bar, let’s add a filename string to the global editor state, and save a copy of the filename there when a file is opened.

```c
struct editorConfig {
  int cx, cy;
  int rx;
  int rowoff;
  int coloff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  char *filename;
  struct termios orig_termios;
};

void editorOpen(char *filename) {
  free(E.filename);
  E.filename = strdup(filename);
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  while ((linelen = getline(&line, &linecap, fp)) != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    editorAppendRow(line, linelen);
  }
  free(line);
  fclose(fp);
}

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  E.filename = NULL;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
  E.screenrows -= 1;
}
```

strdup() comes from *string.h*. It makes a copy of the given string, allocating the required memory and assuming you will free() that memory.

We initialize E.filename to the NULL pointer, and it will stay NULL if a file isn’t opened (which is what happens when the program is run without arguments).Now we’re ready to display some information in the status bar. We’ll display up to 20 characters of the filename, followed by the number of lines in the file. If there is no filename, we’ll display '[No Name]' instead.

```c
void editorDrawStatusBar(struct abuf *ab) {
  abAppend(ab, "\x1b[7m", 4);
  char status[80];
  int len = snprintf(status, sizeof(status), "%.20s - %d lines",
    E.filename ? E.filename : "[No Name]", E.numrows);
  if (len > E.screencols) len = E.screencols;
  abAppend(ab, status, len);
  while (len < E.screencols) {
    abAppend(ab, " ", 1);
    len++;
  }
  abAppend(ab, "\x1b[m", 3);
}
```

Now let’s show the current line number, and align it to the right edge of the screen.

```c
void editorDrawStatusBar(struct abuf *ab) {
  abAppend(ab, "\x1b[7m", 4);
  char status[80], rstatus[80];
  int len = snprintf(status, sizeof(status), "%.20s - %d lines",
    E.filename ? E.filename : "[No Name]", E.numrows);
  int rlen = snprintf(rstatus, sizeof(rstatus), "%d/%d",
    E.cy + 1, E.numrows);
  if (len > E.screencols) len = E.screencols;
  abAppend(ab, status, len);
  while (len < E.screencols) {
    if (E.screencols - len == rlen) {
      abAppend(ab, rstatus, rlen);
      break;
    } else {
      abAppend(ab, " ", 1);
      len++;
    }
  }
  abAppend(ab, "\x1b[m", 3);
}
```

<br>

#### 3.14. Status message.

We’re going to add one more line below our status bar. 

This will be for displaying messages to the user, and prompting the user for input when doing a search, for example. 

We’ll store the current message in a string called *statusmsg*, which we’ll put in the global editor state. 

We’ll also store a timestamp for the message, so that we can erase it a few seconds after it’s been displayed.

```c
#include <time.h>

struct editorConfig {
  int cx, cy;
  int rx;
  int rowoff;
  int coloff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  char *filename;
  char statusmsg[80];
  time_t statusmsg_time;
  struct termios orig_termios;
};

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  E.filename = NULL;
  E.statusmsg[0] = '\0';
  E.statusmsg_time = 0;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
  E.screenrows -= 1;
}
```

We initialize E.statusmsg to an empty string, so no message will be displayed by default. E.statusmsg_time will contain the timestamp when we set a status message.

Let’s define an *editorSetStatusMessage()* function. 
This function will take a format string and a variable number of arguments, like the printf() family of functions:

```c
#include <stdarg.h>

void editorSetStatusMessage(const char *fmt, ...) {
  va_list ap;
  va_start(ap, fmt);
  vsnprintf(E.statusmsg, sizeof(E.statusmsg), fmt, ap);
  va_end(ap);
  E.statusmsg_time = time(NULL);
}

int main(int argc, char *argv[]) {
  enableRawMode();
  initEditor();
  if (argc >= 2) {
    editorOpen(argv[1]);
  }
  editorSetStatusMessage("HELP: Ctrl-Q = quit");
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

va_list, va_start(), and va_end() come from *stdarg.h*. vsnprintf() comes from *stdio.h*. time() comes from *time.h*.

In main(), we set the initial status message to a help message with the key bindings that our text editor uses (currently, just Ctrl-Q to quit).

vsnprintf() helps us make our own printf()-style function. We store the resulting string in E.statusmsg, and set E.statusmsg_time to the current time, which can be gotten by passing NULL to time(). (It returns the number of seconds that have passed since midnight, January 1, 1970 as an integer.)

The ... argument makes editorSetStatusMessage() a variadic function, meaning it can take any number of arguments. C’s way of dealing with these arguments is by having you call va_start() and va_end() on a value of type va_list. The last argument before the ... (in this case, fmt) must be passed to va_start(), so that the address of the next arguments is known. Then, between the va_start() and va_end() calls, you would call va_arg() and pass it the type of the next argument (which you usually get from the given format string) and it would return the value of that argument. In this case, we pass fmt and ap to vsnprintf() and it takes care of reading the format string and calling va_arg() to get each argument.

Now that we have a status message to display, let’s make room for a second line beneath our status bar where we’ll display the message.

```c
void editorDrawStatusBar(struct abuf *ab) {
  abAppend(ab, "\x1b[7m", 4);
  char status[80], rstatus[80];
  int len = snprintf(status, sizeof(status), "%.20s - %d lines",
    E.filename ? E.filename : "[No Name]", E.numrows);
  int rlen = snprintf(rstatus, sizeof(rstatus), "%d/%d",
    E.cy + 1, E.numrows);
  if (len > E.screencols) len = E.screencols;
  abAppend(ab, status, len);
  while (len < E.screencols) {
    if (E.screencols - len == rlen) {
      abAppend(ab, rstatus, rlen);
      break;
    } else {
      abAppend(ab, " ", 1);
      len++;
    }
  }
  abAppend(ab, "\x1b[m", 3);
  abAppend(ab, "\r\n", 2);
}

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  E.filename = NULL;
  E.statusmsg[0] = '\0';
  E.statusmsg_time = 0;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
  E.screenrows -= 2;
}
```

We decrement E.screenrows again, and print a newline after the first status bar. We now have a blank final line once again.

Let’s draw the message bar in a new editorDrawMessageBar() function.

```c
void editorDrawMessageBar(struct abuf *ab) {
  abAppend(ab, "\x1b[K", 3);
  int msglen = strlen(E.statusmsg);
  if (msglen > E.screencols) msglen = E.screencols;
  if (msglen && time(NULL) - E.statusmsg_time < 5)
    abAppend(ab, E.statusmsg, msglen);
}
void editorRefreshScreen() {
  editorScroll();
  struct abuf ab = ABUF_INIT;
  abAppend(&ab, "\x1b[?25l", 6);
  abAppend(&ab, "\x1b[H", 3);
  editorDrawRows(&ab);
  editorDrawStatusBar(&ab);
  editorDrawMessageBar(&ab);
  char buf[32];
  snprintf(buf, sizeof(buf), "\x1b[%d;%dH", (E.cy - E.rowoff) + 1,
                                            (E.rx - E.coloff) + 1);
  abAppend(&ab, buf, strlen(buf));
  abAppend(&ab, "\x1b[?25h", 6);
  write(STDOUT_FILENO, ab.b, ab.len);
  abFree(&ab);
}
```

First we clear the message bar with the '\[K' escape sequence. Then we make sure the message will fit the width of the screen, and then display the message, but only if the message is less than 5 seconds old.

When you start up the program now, you should see the help message at the bottom. It will disappear when you press a key after 5 seconds. Remember, we only refresh the screen after each keypress.

<br>

### 4. A Text Editor.

#### 4.1. Insert Ordinary characters.

In the code of the previous section, we create a program that modify some terminal characteristics and render a file on it.

Now, we want over the terminal in order to transform our text viewer in a text editor. Let's start writing a function that insert a single character into an erow at a given position.

```c
void editorRowInsertChar(erow *row, int at, int c) {
  if (at < 0 || at > row->size) at = row->size;
  row->chars = realloc(row->chars, row->size + 2);
  memmove(&row->chars[at + 1], &row->chars[at], row->size - at + 1);
  row->size++;
  row->chars[at] = c;
  editorUpdateRow(row);
}
```

The function perform as follows:

- First, we validate the index 'at' which marks the position in which we want to insert the character. Notice that at is allowed to go one character past the end of the string, in which case the character should be inserted at the end of the string.

- Then we allocate one more byte for the chars of the erow (we add 2 because we also have to make room for the null byte).

  ```c
  row->chars = realloc(row->chars, row->size + 2);
  ```

- Now we use *memmove()* to shift the entire row and make space for a new byte on the desired location:

  ```c
  void *memmove(void *dest, const void *src, size_t n);
  ```

  which is a function that copies *n* bytes from a memory area designated by an starting pointer (void* src) to a destination memory area *dest* again following an starting pointer. 
  
  The memory areas may overlap and return a void pointer to dest area. In this case, the memory area coincide in a way that we expand the original area so doing something with the return value is not required.

  ```c
  memmove(&row->chars[at + 1], &row->chars[at], row->size - at + 1);
  ```

  So with *memmove()*, we take the bytes starting from *&row->chars\[at\]* this is, from row->chars buffer at the position 'at' and move it over the region starting one byte to the right creating one byte gap between the two regions.

- Then, we update the size of the row, place the new character on the new allocated byte and call *editorUpdateRow()*. 

Now, we’ll add a function to this section called editorInsertChar() which will take a character and use *editorRowInsertChar()* to insert that character into the position that the cursor is at.

```c
void editorInsertChar(int c) {

  //Insert an empty row if the position of the cursor is at the edge of the screen.
  if (E.cy == E.numrows) {
    editorAppendRow("", 0);
  }

  //We call the function described above to insert a char on the row mark at E.cy on position E.cx.
  editorRowInsertChar(&E.row[E.cy], E.cx, c);
  E.cx++;
}
```

Notice that editorInsertChar() doesn’t have to worry about the details of modifying an erow, and editorRowInsertChar() doesn’t have to worry about where the cursor is. 

Let’s call editorInsertChar() in the default: case of the switch statement in editorProcessKeypress(). This will allow any keypress that isn’t mapped to another editor function to be inserted directly into the text being edited.

```c
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    //default is the case triggered wheb any of the previous case matchs. Thus, when we press any key on the keyboard we insert it on the text editor. 
    default:
      editorInsertChar(c);
      break;
  }
}
```
<br>

#### 4.2. Prevent inserting special characters.

Currently, if you press keys like Backspace or Enter, those characters will be inserted directly into the text, which we certainly don’t want. 

Let’s handle a bunch of these special keys in *editorProcessKeypress()*, so that they don’t fall through to the default case of calling *editorInsertChar()*.

First, 'BACKSPACE' doesn’t have a human-readable backslash-escape representation in C (like \n, \r, and so on), so we make it part of the editorKey enum and assign it its ASCII value of 127:

```c
enum editorKey {
  BACKSPACE = 127, 
  ARROW_LEFT = 1000,
  ARROW_RIGHT,
  ARROW_UP,
  ARROW_DOWN,
  DEL_KEY,
  HOME_KEY,
  END_KEY,
  PAGE_UP,
  PAGE_DOWN
};
```

Then, in editorProcessKeypress():

```c
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case '\r':
      /* TODO */
      break;
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case BACKSPACE:
    case CTRL_KEY('h'):
    case DEL_KEY:
      /* TODO */
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    case CTRL_KEY('l'):
    case '\x1b':
      break;
    default:
      editorInsertChar(c);
      break;
  }
}
```

- The first new key we add to the switch statement is '\r', which is the Enter key. For now we want to ignore it, but obviously we’ll be making it do something later, so we mark it with a TODO comment.
- We handle Backspace and Delete in a similar way, marking them with a TODO.
- We also handle the Ctrl-H key combination, which sends the control code 8, which is originally what the Backspace character would send back in the day. 
- Lastly, we handle Ctrl-L and Escape by not doing anything when those keys are pressed. Ctrl-L is traditionally used to refresh the screen in terminal programs. In our text editor, the screen refreshes after any keypress, so we don’t have to do anything else to implement that feature. 

We ignore the Escape key because there are many key escape sequences that we aren’t handling (such as the F1–F12 keys), and the way we wrote editorReadKey(), pressing those keys will be equivalent to pressing the Escape key. We don’t want the user to unwittingly insert the escape character 27 into their text, so we ignore those keypresses.

<br>

#### 4.3. Save to disk.

Now that we’ve finally made text editable, let’s implement saving to disk. First we’ll write a function that converts our array of erow structs into a single string that is ready to be written out to a file.

```c
char *editorRowsToString(int *buflen) {
  int totlen = 0;
  int j;
  for (j = 0; j < E.numrows; j++)
    totlen += E.row[j].size + 1;
  *buflen = totlen;
  char *buf = malloc(totlen);
  char *p = buf;
  for (j = 0; j < E.numrows; j++) {
    memcpy(p, E.row[j].chars, E.row[j].size);
    p += E.row[j].size;
    *p = '\n';
    p++;
  }
  return buf;
}
```

This function returns a pointer to a char and needs a pointer to an integer, the buffer length and performs as follows:

- First we calculate the length of the total rows adding the row's size for each row in a variable:

  ```c
    int totlen = 0;
    int j;
    for (j = 0; j < E.numrows; j++)
      totlen += E.row[j].size + 1;
    *buflen = totlen;
  ```

- Then, we alloc the same amount of space for a pointer to a char (a string to chars):

  ```c
  char *buf = malloc(totlen);
  ```

- Then, we copy the contents of each row into buff making pointer arithmetic:

  - First, we use *memcpy()*:

    ```c
      char *p = buf;
      for (j = 0; j < E.numrows; j++) {
        memcpy(p, E.row[j].chars, E.row[j].size);
    ```

  - Then, we use pointer arithmetic to move the pointer 'p' up to the final of the just added row and add a salt line escape sequence, and then move it one position more:

    ```c
    p += E.row[j].size;
    *p = '\n';
    p++;
    ```

  - Then we return a pointer to the initial position of the buffer.

    ```c
    return buf;
    ```

With this; this function converts the editor's internal row structure into a single string buffer.

Now we’ll implement the editorSave() function, which will actually write the string returned by editorRowsToString() to disk.

```c
#include <fcntl.h>

void editorSave() {
  //If it’s a new file, then E.filename will be NULL, and we won’t know where to save the file, so we just return without doing anything for now. 
  if (E.filename == NULL) return;

  //Call the function to create the buffer.
  int len;
  char *buf = editorRowsToString(&len);

  //Crete the file if not exists for READ&WRITE operations and with default permissions.
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);

  //ftruncate() sets the file’s size to the specified length.
  ftruncate(fd, len);

  //Write the contents, close the file and free the buffer pointer.
  write(fd, buf, len);
  close(fd);
  free(buf);
}
```

Anyways, all we have to do now is map a key to editorSave(). We’ll use Ctrl-S.

```c
void editorProcessKeypress() {
  int c = editorReadKey();
  switch (c) {
    case '\r':
      /* TODO */
      break;
    case CTRL_KEY('q'):
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case CTRL_KEY('s'):
      editorSave(); //Save file.
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case BACKSPACE:
    case CTRL_KEY('h'):
    case DEL_KEY:
      /* TODO */
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    case CTRL_KEY('l'):
    case '\x1b':
      break;
    default:
      editorInsertChar(c);
      break;
  }
}
```

Now, let's add some error handling to *editorSave()* function:

```c
void editorSave() {
  if (E.filename == NULL) return;
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) { //Check every function doesn't return an error value.
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        return;
      }
    }
    close(fd);
  }
  free(buf);
}
```

open() and ftruncate() both return -1 on error. We expect write() to return the number of bytes we told it to write. Whether or not an error occurred, we ensure that the file is closed and the memory that buf points to is freed.

Let’s use *editorSetStatusMessage()* to notify the user whether the save succeeded or not. While we’re at it, we’ll add the Ctrl-S key binding to the help message that’s set in main():

```c
void editorSave() {
  if (E.filename == NULL) return;
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) {
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        editorSetStatusMessage("%d bytes written to disk", len);
        return;
      }
    }
    close(fd);
  }
  free(buf);
  editorSetStatusMessage("Can't save! I/O error: %s", strerror(errno));
}

int main(int argc, char *argv[]) {
  enableRawMode();
  initEditor();
  if (argc >= 2) {
    editorOpen(argv[1]);
  }
  editorSetStatusMessage("HELP: Ctrl-S = save | Ctrl-Q = quit");
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

The above code doesn’t actually compile, because we are trying to call editorSetStatusMessage() before it is defined in the file.

<br>

#### 4.4. Dirty flag.

We’d like to keep track of whether the text loaded in our editor differs from what’s in the file. Then we can warn the user that they might lose unsaved changes when they try to quit.

We call a text buffer “dirty” if it has been modified since opening or saving the file. Let’s add a dirty variable to the global editor state, and initialize it to 0.

```c
struct editorConfig {
  int cx, cy;
  int rx;
  int rowoff;
  int coloff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  int dirty;
  char *filename;
  char statusmsg[80];
  time_t statusmsg_time;
  struct termios orig_termios;
};

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  E.dirty = 0;
  E.filename = NULL;
  E.statusmsg[0] = '\0';
  E.statusmsg_time = 0;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
  E.screenrows -= 2;
}
```

Let’s show the state of E.dirty in the status bar, by displaying (modified) after the filename if the file has been modified.

```c
void editorDrawStatusBar(struct abuf *ab) {
  abAppend(ab, "\x1b[7m", 4);
  char status[80], rstatus[80];
  int len = snprintf(status, sizeof(status), "%.20s - %d lines %s",
    E.filename ? E.filename : "[No Name]", E.numrows,
    E.dirty ? "(modified)" : ""); //If change has been made, then we modify dirty.
  int rlen = snprintf(rstatus, sizeof(rstatus), "%d/%d",
    E.cy + 1, E.numrows);
  if (len > E.screencols) len = E.screencols;
  abAppend(ab, status, len);
  while (len < E.screencols) {
    if (E.screencols - len == rlen) {
      abAppend(ab, rstatus, rlen);
      break;
    } else {
      abAppend(ab, " ", 1);
      len++;
    }
  }
  abAppend(ab, "\x1b[m", 3);
  abAppend(ab, "\r\n", 2);
}
```

Now let’s increment E.dirty in each row operation that makes a change to the text.

```c
void editorAppendRow(char *s, size_t len) {
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  int at = E.numrows;
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  editorUpdateRow(&E.row[at]);
  E.numrows++;
  E.dirty++;
}
void editorRowInsertChar(erow *row, int at, int c) {
  if (at < 0 || at > row->size) at = row->size;
  row->chars = realloc(row->chars, row->size + 2);
  memmove(&row->chars[at + 1], &row->chars[at], row->size - at + 1);
  row->size++;
  row->chars[at] = c;
  editorUpdateRow(row);
  E.dirty++;
}
```

We could have used E.dirty = 1 instead of E.dirty++, but by incrementing it we can have a sense of “how dirty” the file is, which could be useful. (We’ll just be treating E.dirty as a boolean value in this tutorial, so it doesn’t really matter.)

If you open a file at this point, you’ll see that (modified) appears right away, before you make any changes. That’s because editorOpen() calls editorAppendRow(), which increments E.dirty. To fix that, let’s reset E.dirty to 0 at the end of editorOpen(), and also in editorSave().

```c
void editorOpen(char *filename) {
  free(E.filename);
  E.filename = strdup(filename);
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  while ((linelen = getline(&line, &linecap, fp)) != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    editorAppendRow(line, linelen);
  }
  free(line);
  fclose(fp);
  E.dirty = 0;
}

void editorSave() {
  if (E.filename == NULL) return;
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) {
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        E.dirty = 0;
        editorSetStatusMessage("%d bytes written to disk", len);
        return;
      }
    }
    close(fd);
  }
  free(buf);
  editorSetStatusMessage("Can't save! I/O error: %s", strerror(errno));
}
```

Now you should see (modified) appear in the status bar when you first insert a character, and you should see it disappear when you save the file to disk.

<br>

#### 4.5. Quit confirmation.

Now we’re ready to warn the user about unsaved changes when they try to quit. If E.dirty is set, we will display a warning in the status bar, and require the user to press Ctrl-Q three more times in order to quit without saving.

```c
#define KILO_QUIT_TIMES 3

void editorProcessKeypress() {
  static int quit_times = KILO_QUIT_TIMES;
  int c = editorReadKey();
  switch (c) {
    case '\r':
      /* TODO */
      break;
    case CTRL_KEY('q'):
      if (E.dirty && quit_times > 0) {
        editorSetStatusMessage("WARNING!!! File has unsaved changes. "
          "Press Ctrl-Q %d more times to quit.", quit_times);
        quit_times--;
        return;
      }
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case CTRL_KEY('s'):
      editorSave();
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case BACKSPACE:
    case CTRL_KEY('h'):
    case DEL_KEY:
      /* TODO */
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    case CTRL_KEY('l'):
    case '\x1b':
      break;
    default:
      editorInsertChar(c);
      break;
  }
  quit_times = KILO_QUIT_TIMES;
}
```

We edit *editorProcessKeypress()*, in order to introduce some check on the case *CTRL_KEY('q')*  

```c
static int quit_times = KILO_QUIT_TIMES;

case CTRL_KEY('q'):
  if (E.dirty && quit_times > 0) {
    editorSetStatusMessage("WARNING!!! File has unsaved changes. "
      "Press Ctrl-Q %d more times to quit.", quit_times);
    quit_times--;
    return;
  }
  write(STDOUT_FILENO, "\x1b[2J", 4);
  write(STDOUT_FILENO, "\x1b[H", 3);
  exit(0);
  break;

quit_times = KILO_QUIT_TIMES;
```

Each time they press Ctrl-Q with unsaved changes, we set the status message and decrement quit_times. When quit_times gets to 0, we finally allow the program to exit. When they press any key other than Ctrl-Q, then quit_times gets reset back to 3 at the end of the editorProcessKeypress() function.

<br>

#### 4.6. Simple backspacing.

Let’s implement backspacing next. First we’ll create an editorRowDelChar() function, which deletes a character in an erow.

```c
void editorRowDelChar(erow *row, int at) {
  if (at < 0 || at >= row->size) return;
  memmove(&row->chars[at], &row->chars[at + 1], row->size - at);
  row->size--;
  editorUpdateRow(row);
  E.dirty++;
}
```

This function is pretty similar to *editorRowInsertChar()*, which shift par of a row to create an empty byte in which we insert a char. In this case, we make the same strategy but we not make space for a new byte, instead we just shift part of the row at the position of the cursor (at) to the left one time effectively deleting the char located before the cursor.

```less
- HELLO -> editorRowInsertChar('C','2') -> HE_LLO -> HECLLO
- HELLO -> editorRowDelChar('2') -> HLLO
```

As we did with *editorRowInsertChar()*, we also create a function to call *editorRowDelChar()*:

```c
void editorDelChar() {
  if (E.cy == E.numrows) return;
  erow *row = &E.row[E.cy];
  if (E.cx > 0) {
    editorRowDelChar(row, E.cx - 1);
    E.cx--;
  }
}
```

If the cursor’s past the end of the file, then there is nothing to delete, and we return immediately. Otherwise, we get the erow the cursor is on, and if there is a character to the left of the cursor, we delete it and move the cursor one to the left.

Let’s map the Backspace, Ctrl-H, and Delete keys to editorDelChar().

```c
void editorProcessKeypress() {
  static int quit_times = KILO_QUIT_TIMES;
  int c = editorReadKey();
  switch (c) {
    case '\r':
      /* TODO */
      break;
    case CTRL_KEY('q'):
      if (E.dirty && quit_times > 0) {
        editorSetStatusMessage("WARNING!!! File has unsaved changes. "
          "Press Ctrl-Q %d more times to quit.", quit_times);
        quit_times--;
        return;
      }
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case CTRL_KEY('s'):
      editorSave();
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case BACKSPACE:
    case CTRL_KEY('h'):
    case DEL_KEY:
      if (c == DEL_KEY) editorMoveCursor(ARROW_RIGHT);
      editorDelChar();
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    case CTRL_KEY('l'):
    case '\x1b':
      break;
    default:
      editorInsertChar(c);
      break;
  }
  quit_times = KILO_QUIT_TIMES;
}
```

It so happens that in our editor, pressing the → key and then Backspace is equivalent to what you would expect from pressing the Delete key in a text editor: it deletes the character to the right of the cursor. So that is how we implement the Delete key above.

<br>

#### 4.7. Backspaceing at the start of a line.

Currently, editorDelChar() doesn’t do anything when the cursor is at the beginning of a line. When the user backspaces at the beginning of a line, we want to append the contents of that line to the previous line, and then delete the current line. This effectively backspaces the implicit \n character in between the two lines to join them into one line. So we need two new row operations: one for appending a string to a row, and one for deleting a row. 

Let’s start by implementing editorDelRow(), which will also require an editorFreeRow() function for freeing the memory owned by the erow we are deleting.

```c
void editorFreeRow(erow *row) {
  free(row->render);
  free(row->chars);
}
void editorDelRow(int at) {
  if (at < 0 || at >= E.numrows) return;
  editorFreeRow(&E.row[at]);
  memmove(&E.row[at], &E.row[at + 1], sizeof(erow) * (E.numrows - at - 1));
  E.numrows--;
  E.dirty++;
}
```

- First we validate the at index. 

- Then we free the memory owned by the row using editorFreeRow(). 

- We then use memmove() to overwrite the deleted row struct with the rest of the rows that come after it, and decrement numrows. Finally, we increment E.dirty.

Now let’s implement editorRowAppendString(), which appends a string to the end of a row.


```c

void editorRowAppendString(erow *row, char *s, size_t len) {
  row->chars = realloc(row->chars, row->size + len + 1);
  memcpy(&row->chars[row->size], s, len);
  row->size += len;
  row->chars[row->size] = '\0';
  editorUpdateRow(row);
  E.dirty++;
}
```

The mecanic of the function is simple, we realloc the size, copy the string with memcpy() and append terminator byte.

Now we’re ready to get editorDelChar() to handle the case where the cursor is at the beginning of a line.

```c
void editorDelChar() {
  if (E.cy == E.numrows) return;
  if (E.cx == 0 && E.cy == 0) return;
  erow *row = &E.row[E.cy];
  if (E.cx > 0) {
    editorRowDelChar(row, E.cx - 1);
    E.cx--;
  } else {
    E.cx = E.row[E.cy - 1].size;
    editorRowAppendString(&E.row[E.cy - 1], row->chars, row->size);
    editorDelRow(E.cy);
    E.cy--;
  }
}
```

<br>

#### 4.8. The Enter key

The last editor operation we have to implement is the Enter key. The Enter key allows the user to insert new lines into the text, or split a line into two lines. The first thing we need to do is rename the editorAppendRow(...) function to editorInsertRow(int at, ...). It will now be able to insert a row at the index specified by the new at argument.

```c
void editorInsertRow(int at, char *s, size_t len) {
  if (at < 0 || at > E.numrows) return;

  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1)); //Expand the number of available line by 1.
  memmove(&E.row[at + 1], &E.row[at], sizeof(erow) * (E.numrows - at)); //At the desired position, we shift the entire block one line creating a salt between two lines. 
  
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  editorUpdateRow(&E.row[at]);
  E.numrows++;
  E.dirty++;
}
```

We now have to replace all calls to editorAppendRow(...) with calls to editorInsertRow(E.numrows, ...).

```c
void editorInsertChar(int c) {
  if (E.cy == E.numrows) {
    editorInsertRow(E.numrows, "", 0);
  }
  editorRowInsertChar(&E.row[E.cy], E.cx, c);
  E.cx++;
}


void editorOpen(char *filename) {
  free(E.filename);
  E.filename = strdup(filename);
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  while ((linelen = getline(&line, &linecap, fp)) != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    editorInsertRow(E.numrows, line, linelen);
  }
  free(line);
  fclose(fp);
  E.dirty = 0;
}
```

Now we are ready to implement *editorInsertNewline()*, which handles an Enter keypress:

```c
void editorInsertNewline() {
  if (E.cx == 0) {
    editorInsertRow(E.cy, "", 0);
  } else {
    erow *row = &E.row[E.cy];
    editorInsertRow(E.cy + 1, &row->chars[E.cx], row->size - E.cx);
    row = &E.row[E.cy];
    row->size = E.cx;
    row->chars[row->size] = '\0';
    editorUpdateRow(row);
  }
  E.cy++;
  E.cx = 0;
}
```
`
The function just distinguish between if we insert a new line at the beginning of the row or within a row. In the second case we split the row at the position E.cy at the E.cx position and insert a newline calling *editorInsertRow()*.

Finally, let’s actually map the Enter key to the editorInsertNewline() operation.

```c
void editorProcessKeypress() {
  static int quit_times = KILO_QUIT_TIMES;
  int c = editorReadKey();
  switch (c) {
    case '\r':
      editorInsertNewline();
      break;
    case CTRL_KEY('q'):
      if (E.dirty && quit_times > 0) {
        editorSetStatusMessage("WARNING!!! File has unsaved changes. "
          "Press Ctrl-Q %d more times to quit.", quit_times);
        quit_times--;
        return;
      }
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case CTRL_KEY('s'):
      editorSave();
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case BACKSPACE:
    case CTRL_KEY('h'):
    case DEL_KEY:
      if (c == DEL_KEY) editorMoveCursor(ARROW_RIGHT);
      editorDelChar();
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    case CTRL_KEY('l'):
    case '\x1b':
      break;
    default:
      editorInsertChar(c);
      break;
  }
  quit_times = KILO_QUIT_TIMES;
}
```

<br>

#### 4.9. Save as.

Currently, when the user runs ./kilo with no arguments, they get a blank file to edit but have no way of saving. We need a way of prompting the user to input a filename when saving a new file.

Let’s make an editorPrompt() function that displays a prompt in the status bar, and lets the user input a line of text after the prompt.

```c
void editorRefreshScreen();
```

```c
char *editorPrompt(char *prompt) {
  size_t bufsize = 128;
  char *buf = malloc(bufsize);
  size_t buflen = 0;
  buf[0] = '\0';
  while (1) {
    editorSetStatusMessage(prompt, buf);
    editorRefreshScreen();
    int c = editorReadKey();
    if (c == '\r') {
      if (buflen != 0) {
        editorSetStatusMessage("");
        return buf;
      }
    } else if (!iscntrl(c) && c < 128) {
      if (buflen == bufsize - 1) {
        bufsize *= 2;
        buf = realloc(buf, bufsize);
      }
      buf[buflen++] = c;
      buf[buflen] = '\0';
    }
  }
}
```

The user’s input is stored in buf, which is a dynamically allocated string that we initalize to the empty string. We then enter an infinite loop that repeatedly sets the status message, refreshes the screen, and waits for a keypress to handle. The prompt is expected to be a format string containing a %s, which is where the user’s input will be displayed.


When the user presses Enter, and their input is not empty, the status message is cleared and their input is returned. Otherwise, when they input a printable character, we append it to buf. If buflen has reached the maximum capacity we allocated (stored in bufsize), then we double bufsize and allocate that amount of memory before appending to buf. We also make sure that buf ends with a \0 character, because both editorSetStatusMessage() and the caller of editorPrompt() will use it to know where the string ends.

Now let’s prompt the user for a filename in editorSave(), when E.filename is NULL.

```c
void editorSave() {
  if (E.filename == NULL) {
    E.filename = editorPrompt("Save as: %s");
  }
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) {
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        E.dirty = 0;
        editorSetStatusMessage("%d bytes written to disk", len);
        return;
      }
    }
    close(fd);
  }
  free(buf);
  editorSetStatusMessage("Can't save! I/O error: %s", strerror(errno));
}
```

Next, let’s allow the user to press Escape to cancel the input prompt.

```c
char *editorPrompt(char *prompt) {
  size_t bufsize = 128;
  char *buf = malloc(bufsize);
  size_t buflen = 0;
  buf[0] = '\0';
  while (1) {
    editorSetStatusMessage(prompt, buf);
    editorRefreshScreen();
    int c = editorReadKey();
    if (c == '\x1b') {
      editorSetStatusMessage("");
      free(buf);
      return NULL;
    } else if (c == '\r') {
      if (buflen != 0) {
        editorSetStatusMessage("");
        return buf;
      }
    } else if (!iscntrl(c) && c < 128) {
      if (buflen == bufsize - 1) {
        bufsize *= 2;
        buf = realloc(buf, bufsize);
      }
      buf[buflen++] = c;
      buf[buflen] = '\0';
    }
  }
}
```

When an input prompt is cancelled, we free() the buf ourselves and return NULL. So let’s handle a return value of NULL in editorSave() by aborting the save operation and displaying a “Save aborted” message to the user.

```c
void editorSave() {
  if (E.filename == NULL) {
    E.filename = editorPrompt("Save as: %s (ESC to cancel)");
    if (E.filename == NULL) {
      editorSetStatusMessage("Save aborted");
      return;
    }
  }
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) {
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        E.dirty = 0;
        editorSetStatusMessage("%d bytes written to disk", len);
        return;
      }
    }
    close(fd);
  }
  free(buf);
  editorSetStatusMessage("Can't save! I/O error: %s", strerror(errno));
}
```

(Note: If you’re using Bash on Windows, you will have to press Escape 3 times to get one Escape keypress to register in our program, because the read() calls in editorReadKey() that look for an escape sequence never time out.)

Now let’s allow the user to press Backspace (or Ctrl-H, or Delete) in the input prompt.

```c
char *editorPrompt(char *prompt) {
  size_t bufsize = 128;
  char *buf = malloc(bufsize);
  size_t buflen = 0;
  buf[0] = '\0';
  while (1) {
    editorSetStatusMessage(prompt, buf);
    editorRefreshScreen();
    int c = editorReadKey();
    if (c == DEL_KEY || c == CTRL_KEY('h') || c == BACKSPACE) {
      if (buflen != 0) buf[--buflen] = '\0';
    } else if (c == '\x1b') {
      editorSetStatusMessage("");
      free(buf);
      return NULL;
    } else if (c == '\r') {
      if (buflen != 0) {
        editorSetStatusMessage("");
        return buf;
      }
    } else if (!iscntrl(c) && c < 128) {
      if (buflen == bufsize - 1) {
        bufsize *= 2;
        buf = realloc(buf, bufsize);
      }
      buf[buflen++] = c;
      buf[buflen] = '\0';
    }
  }
}
```

<br>

### 5. Search.

#### 5.1. Implemenmt a search functionality.

Let’s use *editorPrompt()* to implement a minimal search feature. 

When the user types a search query and presses Enter, we’ll loop through all the rows of the file, and if a row contains their query string, we’ll move the cursor to the match.

```c
void editorFind() {
  char *query = editorPrompt("Search: %s (ESC to cancel)");
  if (query == NULL) return;
  int i;
  for (i = 0; i < E.numrows; i++) {
    erow *row = &E.row[i];
    char *match = strstr(row->render, query);
    if (match) {
      E.cy = i;
      E.cx = match - row->render;
      E.rowoff = E.numrows;
      break;
    }
  }
  free(query);
}
```

The strstr() function finds the first occurrence of the substring *query* in the string *row->render*.

Then, we evaluate if match is not NULL and move the cursor (E.cy, E.cx) to that position. Realize that  we set "E.cx" substratic the pointer 'row->render' to 'match' giving as result the number of offsets between the two of them in the row, this is; the offset in which 'match' starts.

Lastly, we set E.rowoff so that we are scrolled to the very bottom of the file, which will cause editorScroll() to scroll upwards at the next screen refresh so that the matching line will be at the very top of the screen.

We assigned a render index to E.cx, but E.cx is an index into chars. If there are tabs to the left of the match, the cursor is going to be in the wrong position. We need to convert the render index into a chars index before assigning it to E.cx. Let’s create an editorRowRxToCx() function, which is the opposite of the editorRowCxToRx()

```c
int editorRowRxToCx(erow *row, int rx) {
  int cur_rx = 0;
  int cx;
  for (cx = 0; cx < row->size; cx++) {
    if (row->chars[cx] == '\t')
      cur_rx += (KILO_TAB_STOP - 1) - (cur_rx % KILO_TAB_STOP);
    cur_rx++;
    if (cur_rx > rx) return cx;
  }
  return cx;
}
```

And call it in *editorFind()*:

```c
void editorFind() {
  char *query = editorPrompt("Search: %s (ESC to cancel)");
  if (query == NULL) return;
  int i;
  for (i = 0; i < E.numrows; i++) {
    erow *row = &E.row[i];
    char *match = strstr(row->render, query);
    if (match) {
      E.cy = i;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      break;
    }
  }
  free(query);
}
```

Finally, let’s map Ctrl-F to the *editorFind()* function, and add it to the help message we set in main():

```c
void editorProcessKeypress() {
  static int quit_times = KILO_QUIT_TIMES;
  int c = editorReadKey();
  switch (c) {
    case '\r':
      editorInsertNewline();
      break;
    case CTRL_KEY('q'):
      if (E.dirty && quit_times > 0) {
        editorSetStatusMessage("WARNING!!! File has unsaved changes. "
          "Press Ctrl-Q %d more times to quit.", quit_times);
        quit_times--;
        return;
      }
      write(STDOUT_FILENO, "\x1b[2J", 4);
      write(STDOUT_FILENO, "\x1b[H", 3);
      exit(0);
      break;
    case CTRL_KEY('s'):
      editorSave();
      break;
    case HOME_KEY:
      E.cx = 0;
      break;
    case END_KEY:
      if (E.cy < E.numrows)
        E.cx = E.row[E.cy].size;
      break;
    case CTRL_KEY('f'):
      editorFind();
      break;
    case BACKSPACE:
    case CTRL_KEY('h'):
    case DEL_KEY:
      if (c == DEL_KEY) editorMoveCursor(ARROW_RIGHT);
      editorDelChar();
      break;
    case PAGE_UP:
    case PAGE_DOWN:
      {
        if (c == PAGE_UP) {
          E.cy = E.rowoff;
        } else if (c == PAGE_DOWN) {
          E.cy = E.rowoff + E.screenrows - 1;
          if (E.cy > E.numrows) E.cy = E.numrows;
        }
        int times = E.screenrows;
        while (times--)
          editorMoveCursor(c == PAGE_UP ? ARROW_UP : ARROW_DOWN);
      }
      break;
    case ARROW_UP:
    case ARROW_DOWN:
    case ARROW_LEFT:
    case ARROW_RIGHT:
      editorMoveCursor(c);
      break;
    case CTRL_KEY('l'):
    case '\x1b':
      break;
    default:
      editorInsertChar(c);
      break;
  }
  quit_times = KILO_QUIT_TIMES;
}
```

```c
int main(int argc, char *argv[]) {
  enableRawMode();
  initEditor();
  if (argc >= 2) {
    editorOpen(argv[1]);
  }
  editorSetStatusMessage(
    "HELP: Ctrl-S = save | Ctrl-Q = quit | Ctrl-F = find");
  while (1) {
    editorRefreshScreen();
    editorProcessKeypress();
  }
  return 0;
}
```

<br>

#### 5.2. Incremental Search.

Now, let’s make our search feature fancy. We want to support incremental search, meaning the file is searched after each keypress when the user is typing in their search query.

To implement this, we’re going to get editorPrompt() to take a callback function as an argument. We’ll have it call this function after each keypress, passing the current search query inputted by the user and the last key they pressed:

```c
char *editorPrompt(char *prompt, void (*callback)(char *, int)) {
  size_t bufsize = 128;
  char *buf = malloc(bufsize);
  size_t buflen = 0;
  buf[0] = '\0';
  while (1) {
    editorSetStatusMessage(prompt, buf);
    editorRefreshScreen();
    int c = editorReadKey();
    if (c == DEL_KEY || c == CTRL_KEY('h') || c == BACKSPACE) {
      if (buflen != 0) buf[--buflen] = '\0';
    } else if (c == '\x1b') {
      editorSetStatusMessage("");
      if (callback) callback(buf, c);
      free(buf);
      return NULL;
    } else if (c == '\r') {
      if (buflen != 0) {
        editorSetStatusMessage("");
        if (callback) callback(buf, c);
        return buf;
      }
    } else if (!iscntrl(c) && c < 128) {
      if (buflen == bufsize - 1) {
        bufsize *= 2;
        buf = realloc(buf, bufsize);
      }
      buf[buflen++] = c;
      buf[buflen] = '\0';
    }
    if (callback) callback(buf, c);
  }
}
```

We will start saying that a "callback function" is a function that gets passed as an argument to another function, which then "calls back" (invokes) that function at specific points during its execution. It's a way to customize behavior without modifying the original function's code.

We implement the callback function at several points on the code:

- After each character is typed or deleted
- When user presses Escape (canceling)
- When user presses Enter (confirming)

This design enables incremental search; as the user types each character, the callback can immediately search the file and highlight matches.

The if statements allow the caller to pass NULL for the callback, in case they don’t want to use a callback. This is the case when we prompt the user for a filename, so let’s pass NULL to editorPrompt() when we do that. We’ll also pass NULL to editorPrompt() in editorFind() for now, to get the code to compile.

```c
void editorSave() {
  if (E.filename == NULL) {
    E.filename = editorPrompt("Save as: %s (ESC to cancel)", NULL);
    if (E.filename == NULL) {
      editorSetStatusMessage("Save aborted");
      return;
    }
  }
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) {
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        E.dirty = 0;
        editorSetStatusMessage("%d bytes written to disk", len);
        return;
      }
    }
    close(fd);
  }
  free(buf);
  editorSetStatusMessage("Can't save! I/O error: %s", strerror(errno));
}

void editorFind() {
  char *query = editorPrompt("Search: %s (ESC to cancel)", NULL);
  if (query == NULL) return;
  int i;
  for (i = 0; i < E.numrows; i++) {
    erow *row = &E.row[i];
    char *match = strstr(row->render, query);
    if (match) {
      E.cy = i;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      break;
    }
  }
  free(query);
}
```

Now that we handle those cases in which we dont want to perform a callback when the user enters input on the prompt, lets define the actual callback function and implements in *editorFind()*:

```c
void editorFindCallback(char *query, int key) {
  if (key == '\r' || key == '\x1b') {
    return;
  }
  int i;
  for (i = 0; i < E.numrows; i++) {
    erow *row = &E.row[i];
    char *match = strstr(row->render, query);
    if (match) {
      E.cy = i;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      break;
    }
  }
}

void editorFind() {
  char *query = editorPrompt("Search: %s (ESC to cancel)", editorFindCallback);
  if (query) {
    free(query);
  }
}
```

Lets check that in *editorFindCallback()* function, we just reused the search codeblock and we just check if the user press Enter or Escape. Thus, every time a user press a key, the callback function will peroform a search of the query string of chars.

<br>

#### 5.3. Restore cursor position when cancelling search.

When the user presses Escape to cancel a search, we want the cursor to go back to where it was when they started the search. To do that, we’ll have to save their cursor position and scroll position, and restore those values after the search is cancelled:

```c
void editorFind() {
  int saved_cx = E.cx;
  int saved_cy = E.cy;
  int saved_coloff = E.coloff;
  int saved_rowoff = E.rowoff;
  char *query = editorPrompt("Search: %s (ESC to cancel)", editorFindCallback);
  if (query) {
    free(query);
  } else {
    E.cx = saved_cx;
    E.cy = saved_cy;
    E.coloff = saved_coloff;
    E.rowoff = saved_rowoff;
  }
}
```

If query is NULL, that means they pressed Escape, so in that case we restore the values we saved.

The last feature we’d like to add is to allow the user to advance to the next or previous match in the file using the arrow keys. The ↑ and ← keys will go to the previous match, and the ↓ and → keys will go to the next match.

We’ll implement this feature using two static variables in our callback. last_match will contain the index of the row that the last match was on, or -1 if there was no last match. And direction will store the direction of the search: 1 for searching forward, and -1 for searching backward.

```c
void editorFindCallback(char *query, int key) {
  static int last_match = -1;
  static int direction = 1;
  if (key == '\r' || key == '\x1b') {
    last_match = -1;
    direction = 1;
    return;
  } else if (key == ARROW_RIGHT || key == ARROW_DOWN) {
    direction = 1;
  } else if (key == ARROW_LEFT || key == ARROW_UP) {
    direction = -1;
  } else {
    last_match = -1;
    direction = 1;
  }
  int i;
  for (i = 0; i < E.numrows; i++) {
    erow *row = &E.row[i];
    char *match = strstr(row->render, query);
    if (match) {
      E.cy = i;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      break;
    }
  }
}
```

As you can see, we always reset last_match to -1 unless an arrow key was pressed. So we’ll only advance to the next or previous match when an arrow key is pressed. You can also see that we always set direction to 1 unless the ← or ↑ key was pressed. So we always search in the forward direction unless the user specifically asks to search backwards from the last match.

If key is '\r' (Enter) or '\x1b' (Escape), that means we’re about to leave search mode. So we reset last_match and direction to their initial values to get ready for the next search operation.

Now that we have those variables all set up, let’s put them to use.

```c
void editorFindCallback(char *query, int key) {
  static int last_match = -1;
  static int direction = 1;
  if (key == '\r' || key == '\x1b') {
    last_match = -1;
    direction = 1;
    return;
  } else if (key == ARROW_RIGHT || key == ARROW_DOWN) {
    direction = 1;
  } else if (key == ARROW_LEFT || key == ARROW_UP) {
    direction = -1;
  } else {
    last_match = -1;
    direction = 1;
  }
  if (last_match == -1) direction = 1;
  int current = last_match;
  int i;
  for (i = 0; i < E.numrows; i++) {
    current += direction;
    if (current == -1) current = E.numrows - 1;
    else if (current == E.numrows) current = 0;
    erow *row = &E.row[current];
    char *match = strstr(row->render, query);
    if (match) {
      last_match = current;
      E.cy = current;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      break;
    }
  }
}
```

current is the index of the current row we are searching. If there was a last match, it starts on the line after (or before, if we’re searching backwards). If there wasn’t a last match, it starts at the top of the file and searches in the forward direction to find the first match.

The if ... else if causes current to go from the end of the file back to the beginning of the file, or vice versa, to allow a search to “wrap around” the end of a file and continue from the top (or bottom).

When we find a match, we set last_match to current, so that if the user presses the arrow keys, we’ll start the next search from that point.

Finally, let’s not forget to update the prompt text to let the user know they can use the arrow keys.

```c
void editorFind() {
  int saved_cx = E.cx;
  int saved_cy = E.cy;
  int saved_coloff = E.coloff;
  int saved_rowoff = E.rowoff;
  char *query = editorPrompt("Search: %s (Use ESC/Arrows/Enter)",
                             editorFindCallback);
  if (query) {
    free(query);
  } else {
    E.cx = saved_cx;
    E.cy = saved_cy;
    E.coloff = saved_coloff;
    E.rowoff = saved_rowoff;
  }
}
```

<br>

### 6. Syntax highlighting.

#### 6.1. Colorful digits-

Let’s start by just getting some color on the screen, as simply as possible. We’ll attempt to highlight numbers by coloring each digit character red.

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      char *c = &E.row[filerow].render[E.coloff];
      int j;
      for (j = 0; j < len; j++) {
        if (isdigit(c[j])) {
          abAppend(ab, "\x1b[31m", 5);
          abAppend(ab, &c[j], 1);
          abAppend(ab, "\x1b[39m", 5);
        } else {
          abAppend(ab, &c[j], 1);
        }
      }
    }
    abAppend(ab, "\x1b[K", 3);
    abAppend(ab, "\r\n", 2);
  }
}
```

Realize that we can no longer just feed the substring of render that we want to print right into abAppend(). We’ll have to do it character-by-character from now on.  So we loop through the characters and use *isdigit()* on each one to test if it is a digit character. If it is, we precede it with the '31m' escape sequence and follow it by the '39m' sequence.

We previously used the m command to draw the status bar using inverted colors. Now we are using it to set the text color. The VT100 User Guide doesn’t document color, so let’s turn to the Wikipedia article on ANSI escape codes. It includes a large table containing all the different argument codes you can use with the m command on various terminals. It also includes the ANSI color table with the 8 foreground/background colors available.

<br>

#### 6.2. Refractor highlighting.

Now we know how to color text, but we’re going to have to do a lot more work to actually highlight entire strings, keywords, comments, and so on. We can’t just decide what color to use based on the class of each character, like we’re doing with digits currently. 

What we want to do is figure out the highlighting for each row of text before we display it, and then rehighlight a line whenever it gets changed. To do that, we need to store the highlighting of each line in an array. Let’s add an array to the erow struct named hl, which stands for “highlight”.

```c
typedef struct erow {
  int size;
  int rsize;
  char *chars;
  char *render;
  unsigned char *hl;
} erow;

void editorInsertRow(int at, char *s, size_t len) {
  if (at < 0 || at > E.numrows) return;
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  memmove(&E.row[at + 1], &E.row[at], sizeof(erow) * (E.numrows - at));
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  E.row[at].hl = NULL;
  editorUpdateRow(&E.row[at]);
  E.numrows++;
  E.dirty++;
}

void editorFreeRow(erow *row) {
  free(row->render);
  free(row->chars);
  free(row->hl);
}
```

hl is an array of unsigned char values, meaning integers in the range of 0 to 255. Each value in the array will correspond to a character in render, and will tell you whether that character is part of a string, or a comment, or a number, and so on. Let’s create an enum containing the possible values that the hl array can contain.

```c
enum editorHighlight {
  HL_NORMAL = 0,
  HL_NUMBER
};
```

For now, we’ll focus on highlighting numbers only. 

So we want every character that’s part of a number to have a corresponding HL_NUMBER value in the hl array, and we want every other value in hl to be HL_NORMAL.

Let’s create a new *editorUpdateSyntax()* function. 

This function will go through the characters of an erow and highlight them by setting each value in the hl array.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  int i;
  for (i = 0; i < row->rsize; i++) {
    if (isdigit(row->render[i])) {
      row->hl[i] = HL_NUMBER;
    }
  }
}
```

- First, we realloc 'hl' array to fulfill the size of the row.
- Then, with memset we fill al the slots of the char's array with HL_NORMAL value.
- Then, we loop every char in the row to identify digts with isdigit() and we register the matchs in hl with HL_NUMBER value.


Lastly, we call *editorUpdateSyntax()* in editorUpdateRow():

```c
void editorUpdateRow(erow *row) {
  int tabs = 0;
  int j;
  for (j = 0; j < row->size; j++)
    if (row->chars[j] == '\t') tabs++;
  free(row->render);
  row->render = malloc(row->size + tabs*(KILO_TAB_STOP - 1) + 1);
  int idx = 0;
  for (j = 0; j < row->size; j++) {
    if (row->chars[j] == '\t') {
      row->render[idx++] = ' ';
      while (idx % KILO_TAB_STOP != 0) row->render[idx++] = ' ';
    } else {
      row->render[idx++] = row->chars[j];
    }
  }
  row->render[idx] = '\0';
  row->rsize = idx;
  editorUpdateSyntax(row);
}
```

editorUpdateRow() already has the job of updating the render array whenever the text of the row changes, so it makes sense that that’s where we want to update the hl array. So after updating render, we call editorUpdateSyntax() at the end.

Next, let’s make an editorSyntaxToColor() function that maps values in hl to the actual ANSI color codes we want to draw them with.

```c
int editorSyntaxToColor(int hl) {
  switch (hl) {
    case HL_NUMBER: return 31;
    default: return 37;
  }
}
```

We return the ANSI code for “foreground red” for numbers, and “foreground white” for anything else that might slip through. (We’ll be handling HL_NORMAL separately, so editorSyntaxToColor() doesn’t need to handle it.)

Now, lets draw the highlighted text to the screen.

```c

void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      char *c = &E.row[filerow].render[E.coloff];
      unsigned char *hl = &E.row[filerow].hl[E.coloff];
      int j;
      for (j = 0; j < len; j++) {
        if (hl[j] == HL_NORMAL) {
          abAppend(ab, "\x1b[39m", 5);
          abAppend(ab, &c[j], 1);
        } else {
          int color = editorSyntaxToColor(hl[j]);
          char buf[16];
          int clen = snprintf(buf, sizeof(buf), "\x1b[%dm", color);
          abAppend(ab, buf, clen);
          abAppend(ab, &c[j], 1);
        }
      }
      abAppend(ab, "\x1b[39m", 5);
    }
    abAppend(ab, "\x1b[K", 3);
    abAppend(ab, "\r\n", 2);
  }
}
```

- First we get a pointer, hl, to the slice of the hl array that corresponds to the slice of render that we are printing. 

- Then, for each character, if it’s an HL_NORMAL character, we use '39m' to make sure we’re using the default text color before printing it. If it’s not HL_NORMAL, we use snprintf() to write the escape sequence into a buffer which we pass to abAppend() before appending the actual character. 

- Finally, after we’re done looping through all the characters and displaying them, we print a final '39m' escape sequence to make sure the text color is reset to default.


In practice, most characters are going to be the same color as the previous character, so most of the escape sequences are redundant. Let’s keep track of the current text color as we loop through the characters, and only print out an escape sequence when the color changes.

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      char *c = &E.row[filerow].render[E.coloff];
      unsigned char *hl = &E.row[filerow].hl[E.coloff];
      int current_color = -1;
      int j;
      for (j = 0; j < len; j++) {
        if (hl[j] == HL_NORMAL) {
          if (current_color != -1) {
            abAppend(ab, "\x1b[39m", 5);
            current_color = -1;
          }
          abAppend(ab, &c[j], 1);
        } else {
          int color = editorSyntaxToColor(hl[j]);
          if (color != current_color) {
            current_color = color;
            char buf[16];
            int clen = snprintf(buf, sizeof(buf), "\x1b[%dm", color);
            abAppend(ab, buf, clen);
          }
          abAppend(ab, &c[j], 1);
        }
      }
      abAppend(ab, "\x1b[39m", 5);
    }
    abAppend(ab, "\x1b[K", 3);
    abAppend(ab, "\r\n", 2);
  }
}
```

current_color is -1 when we want the default text color, otherwise it is set to the value that editorSyntaxToColor() last returned. When the color changes, we print out the escape sequence for that color and set current_color to the new color.

<br>

#### 6.3. Colorful search results.

Before we start highlighting strings and keywords and all that, let’s use our highlighting system to highlight search results. We’ll start by adding HL_MATCH to the editorHighlight enum, and mapping it to the color blue (34) in editorSyntaxToColor().

```c
enum editorHighlight {
  HL_NORMAL = 0,
  HL_NUMBER,
  HL_MATCH
};

int editorSyntaxToColor(int hl) {
  switch (hl) {
    case HL_NUMBER: return 31;
    case HL_MATCH: return 34;
    default: return 37;
  }
}
```

Now all we have to do is memset() the matched substring to HL_MATCH in our search code.

```c
void editorFindCallback(char *query, int key) {
  static int last_match = -1;
  static int direction = 1;
  if (key == '\r' || key == '\x1b') {
    last_match = -1;
    direction = 1;
    return;
  } else if (key == ARROW_RIGHT || key == ARROW_DOWN) {
    direction = 1;
  } else if (key == ARROW_LEFT || key == ARROW_UP) {
    direction = -1;
  } else {
    last_match = -1;
    direction = 1;
  }
  if (last_match == -1) direction = 1;
  int current = last_match;
  int i;
  for (i = 0; i < E.numrows; i++) {
    current += direction;
    if (current == -1) current = E.numrows - 1;
    else if (current == E.numrows) current = 0;
    erow *row = &E.row[current];
    char *match = strstr(row->render, query);
    if (match) {
      last_match = current;
      E.cy = current;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      memset(&row->hl[match - row->render], HL_MATCH, strlen(query));
      break;
    }
  }
}
```

<br>

#### 6.4. Restore syntax highlighting after search.

Currently, search results stay highlighted in blue even after the user is done using the search feature. We want to restore hl to its previous value after each search.

To do that, we’ll save the original contents of hl in a static variable named *saved_hl* in editorFindCallback(), and restore hl to the contents of saved_hl at the top of the callback.

```c
void editorFindCallback(char *query, int key) {
  static int last_match = -1;
  static int direction = 1;
  static int saved_hl_line;
  static char *saved_hl = NULL;
  if (saved_hl) {
    memcpy(E.row[saved_hl_line].hl, saved_hl, E.row[saved_hl_line].rsize);
    free(saved_hl);
    saved_hl = NULL;
  }
  if (key == '\r' || key == '\x1b') {
    last_match = -1;
    direction = 1;
    return;
  } else if (key == ARROW_RIGHT || key == ARROW_DOWN) {
    direction = 1;
  } else if (key == ARROW_LEFT || key == ARROW_UP) {
    direction = -1;
  } else {
    last_match = -1;
    direction = 1;
  }
  if (last_match == -1) direction = 1;
  int current = last_match;
  int i;
  for (i = 0; i < E.numrows; i++) {
    current += direction;
    if (current == -1) current = E.numrows - 1;
    else if (current == E.numrows) current = 0;
    erow *row = &E.row[current];
    char *match = strstr(row->render, query);
    if (match) {
      last_match = current;
      E.cy = current;
      E.cx = editorRowRxToCx(row, match - row->render);
      E.rowoff = E.numrows;
      saved_hl_line = current;
      saved_hl = malloc(row->rsize);
      memcpy(saved_hl, row->hl, row->rsize);
      memset(&row->hl[match - row->render], HL_MATCH, strlen(query));
      break;
    }
  }
}
```

We use another static variable named saved_hl_line to know which line’s hl needs to be restored. saved_hl is a dynamically allocated array which points to NULL when there is nothing to restore. If there is something to restore, we memcpy() it to the saved line’s hl and then deallocate saved_hl and set it back to NULL.

<br>

#### 6.5. Colorful numbers.

Alright, let’s start working on highlighting numbers properly. First, we’ll change our for loop in editorUpdateSyntax() to a while loop, to allow us to consume multiple characters each iteration. (We’ll only consume one character at a time for numbers, but this will be useful for later.)

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    if (isdigit(c)) {
      row->hl[i] = HL_NUMBER;
    }
    i++;
  }
}
```

Now let’s define an is_separator() function that takes a character and returns true if it’s considered a separator character.

```c
int is_separator(int c) {
  return isspace(c) || c == '\0' || strchr(",.()+-/*=~%<>[];", c) != NULL;
}
```

strchr() comes from *string.h*. It looks for the first occurrence of a character in a string, and returns a pointer to the matching character in the string. If the string doesn’t contain the character, strchr() returns NULL.

Right now, numbers are highlighted even if they’re part of an identifier, such as the 32 in int32_t. To fix that, we’ll require that numbers are preceded by a separator character, which includes whitespace or punctuation characters. We also include the null byte ('\0'), because then we can count the null byte at the end of each line as a separator, which will make some of our code simpler in the future.

Let’s add a prev_sep variable to editorUpdateSyntax() that keeps track of whether the previous character was a separator. Then let’s use it to recognize and highlight numbers properly.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  int prev_sep = 1;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) {
      row->hl[i] = HL_NUMBER;
      i++;
      prev_sep = 0;
      continue;
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

We initialize prev_sep to 1 (meaning true) because we consider the beginning of the line to be a separator. (Otherwise numbers at the very beginning of the line wouldn’t be highlighted.)

prev_hl is set to the highlight type of the previous character. To highlight a digit with HL_NUMBER, we now require the previous character to either be a separator, or to also be highlighted with HL_NUMBER.

When we decide to highlight the current character a certain way (HL_NUMBER in this case), we increment i to “consume” that character, set prev_sep to 0 to indicate we are in the middle of highlighting something, and then continue the loop. We will use this pattern for each thing that we highlight.

If we end up not highlighting the current character, then we’ll end up at the bottom of the while loop, where we set prev_sep according to whether the current character is a separator, and we increment i to consume the character. The memset() we did at the top of the function means that an unhighlighted character will have a value of HL_NORMAL in hl.

Now let’s support highlighting numbers that contain decimal points.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  int prev_sep = 1;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
        (c == '.' && prev_hl == HL_NUMBER)) {
      row->hl[i] = HL_NUMBER;
      i++;
      prev_sep = 0;
      continue;
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

<br>

#### 6.6. Detect filetype.

Before we go on to highlight other things, we’re going to add filetype detection to our editor. This will allow us to have different rules for how to highlight different types of files. For example, text files shouldn’t have any highlighting, and C files should highlight numbers, strings, C/C++-style comments, and many different keywords specific to C.

Let’s create an editorSyntax struct that will contain all the syntax highlighting information for a particular filetype.

```c
#define HL_HIGHLIGHT_NUMBERS (1<<0)

struct editorSyntax {
  char *filetype;
  char **filematch;
  int flags;
};
```

- The filetype field is the name of the filetype that will be displayed to the user in the status bar. 
- The filematch is an array of strings, where each string contains a pattern to match a filename against.

Then, if the filename matches, then the file will be recognized as having that filetype. Finally, flags is a bit field that will contain flags for whether to highlight numbers and whether to highlight strings for that filetype. For now, we define just the HL_HIGHLIGHT_NUMBERS flag bit.

Now let’s make an array of built-in editorSyntax structs, and add one for the C language to it:

```c

#define HLDB_ENTRIES (sizeof(HLDB) / sizeof(HLDB[0]))

char *C_HL_extensions[] = { ".c", ".h", ".cpp", NULL };
struct editorSyntax HLDB[] = {
  {
    "c",
    C_HL_extensions,
    HL_HIGHLIGHT_NUMBERS
  },
};
```

Thus, HLDB stands for “highlight database”. Our editorSyntax struct for the C language contains the string "c" for the filetype field, the extensions ".c", ".h", and ".cpp" for the filematch field (the array must be terminated with NULL), and the HL_HIGHLIGHT_NUMBERS flag turned on in the flags field.

We then define an HLDB_ENTRIES constant to store the length of the HLDB array.

Now let’s add a pointer to the current editorSyntax struct in our global editor state, and initialize it to NULL.

```c
struct editorConfig {
  int cx, cy;
  int rx;
  int rowoff;
  int coloff;
  int screenrows;
  int screencols;
  int numrows;
  erow *row;
  int dirty;
  char *filename;
  char statusmsg[80];
  time_t statusmsg_time;
  struct editorSyntax *syntax;
  struct termios orig_termios;
};

struct editorConfig E;

void initEditor() {
  E.cx = 0;
  E.cy = 0;
  E.rx = 0;
  E.rowoff = 0;
  E.coloff = 0;
  E.numrows = 0;
  E.row = NULL;
  E.dirty = 0;
  E.filename = NULL;
  E.statusmsg[0] = '\0';
  E.statusmsg_time = 0;
  E.syntax = NULL;
  if (getWindowSize(&E.screenrows, &E.screencols) == -1) die("getWindowSize");
  E.screenrows -= 2;
}
```

When E.syntax is NULL, that means there is no filetype for the current file, and no syntax highlighting should be done.

Let’s show the current filetype in the status bar. If E.syntax is NULL, then we’ll display no ft (“no filetype”) instead.

```c
void editorDrawStatusBar(struct abuf *ab) {
  abAppend(ab, "\x1b[7m", 4);
  char status[80], rstatus[80];
  int len = snprintf(status, sizeof(status), "%.20s - %d lines %s",
    E.filename ? E.filename : "[No Name]", E.numrows,
    E.dirty ? "(modified)" : "");
  int rlen = snprintf(rstatus, sizeof(rstatus), "%s | %d/%d",
    E.syntax ? E.syntax->filetype : "no ft", E.cy + 1, E.numrows);
  if (len > E.screencols) len = E.screencols;
  abAppend(ab, status, len);
  while (len < E.screencols) {
    if (E.screencols - len == rlen) {
      abAppend(ab, rstatus, rlen);
      break;
    } else {
      abAppend(ab, " ", 1);
      len++;
    }
  }
  abAppend(ab, "\x1b[m", 3);
  abAppend(ab, "\r\n", 2);
}
```

Now let’s change editorUpdateSyntax() to take the current E.syntax value into account.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  int prev_sep = 1;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

If no filetype is set, we return immediately after memset()ting the entire line to HL_NORMAL. We also wrap the number-highlighting code in an if statement that checks to see if numbers should be highlighted for the current filetype.

Now we’ll create an editorSelectSyntaxHighlight() function that tries to match the current filename to one of the filematch fields in the HLDB. If one matches, it’ll set E.syntax to that filetype.

```c
void editorSelectSyntaxHighlight() {
  E.syntax = NULL;
  if (E.filename == NULL) return;
  char *ext = strrchr(E.filename, '.');
  for (unsigned int j = 0; j < HLDB_ENTRIES; j++) {
    struct editorSyntax *s = &HLDB[j];
    unsigned int i = 0;
    while (s->filematch[i]) {
      int is_ext = (s->filematch[i][0] == '.');
      if ((is_ext && ext && !strcmp(ext, s->filematch[i])) ||
          (!is_ext && strstr(E.filename, s->filematch[i]))) {
        E.syntax = s;
        return;
      }
      i++;
    }
  }
}
```

strrchr() and strcmp() come from <string.h>. strrchr() returns a pointer to the last occurrence of a character in a string, and strcmp() returns 0 if two given strings are equal.

First we set E.syntax to NULL, so that if nothing matches or if there is no filename, then there is no filetype.

Then we get a pointer to the extension part of the filename by using strrchr() to find the last occurrence of the . character. If there is no extension, then ext will be NULL.

Finally, we loop through each editorSyntax struct in the HLDB array, and for each one of those, we loop through each pattern in its filematch array. If the pattern starts with a ., then it’s a file extension pattern, and we use strcmp() to see if the filename ends with that extension. If it’s not a file extension pattern, then we just check to see if the pattern exists anywhere in the filename, using strstr(). If the filename matched according to those rules, then we set E.syntax to the current editorSyntax struct, and return.

We want to call editorSelectSyntaxHighlight() wherever E.filename changes. This is in editorOpen() and editorSave().

```c
void editorOpen(char *filename) {
  free(E.filename);
  E.filename = strdup(filename);
  editorSelectSyntaxHighlight();
  FILE *fp = fopen(filename, "r");
  if (!fp) die("fopen");
  char *line = NULL;
  size_t linecap = 0;
  ssize_t linelen;
  while ((linelen = getline(&line, &linecap, fp)) != -1) {
    while (linelen > 0 && (line[linelen - 1] == '\n' ||
                           line[linelen - 1] == '\r'))
      linelen--;
    editorInsertRow(E.numrows, line, linelen);
  }
  free(line);
  fclose(fp);
  E.dirty = 0;
}

void editorSave() {
  if (E.filename == NULL) {
    E.filename = editorPrompt("Save as: %s (ESC to cancel)", NULL);
    if (E.filename == NULL) {
      editorSetStatusMessage("Save aborted");
      return;
    }
    editorSelectSyntaxHighlight();
  }
  int len;
  char *buf = editorRowsToString(&len);
  int fd = open(E.filename, O_RDWR | O_CREAT, 0644);
  if (fd != -1) {
    if (ftruncate(fd, len) != -1) {
      if (write(fd, buf, len) == len) {
        close(fd);
        free(buf);
        E.dirty = 0;
        editorSetStatusMessage("%d bytes written to disk", len);
        return;
      }
    }
    close(fd);
  }
  free(buf);
  editorSetStatusMessage("Can't save! I/O error: %s", strerror(errno));
}
```

At this point, when you open a C file in the editor, you should see numbers getting highlighted, and you should see c in the status bar where we display the filetype. When you start up the editor with no arguments and save the file with a filename that ends in .c, you should see the filetype in the status bar change satisfyingly from no ft to c. However, any numbers you might have in the file will not be highlighted! Very unsatisfying!

Let’s rehighlight the entire file after setting E.syntax in editorSelectSyntaxHighlight().

```c
void editorSelectSyntaxHighlight() {
  E.syntax = NULL;
  if (E.filename == NULL) return;
  char *ext = strrchr(E.filename, '.');
  for (unsigned int j = 0; j < HLDB_ENTRIES; j++) {
    struct editorSyntax *s = &HLDB[j];
    unsigned int i = 0;
    while (s->filematch[i]) {
      int is_ext = (s->filematch[i][0] == '.');
      if ((is_ext && ext && !strcmp(ext, s->filematch[i])) ||
          (!is_ext && strstr(E.filename, s->filematch[i]))) {
        E.syntax = s;
        int filerow;
        for (filerow = 0; filerow < E.numrows; filerow++) {
          editorUpdateSyntax(&E.row[filerow]);
        }
        return;
      }
      i++;
    }
  }
}
```

<br>

#### 6.7. Colorful strings.

With all that out of the way, we can finally get to highlighting more things! Let’s start with strings.

```c
enum editorHighlight {
  HL_NORMAL = 0,
  HL_STRING,
  HL_NUMBER,
  HL_MATCH
};

int editorSyntaxToColor(int hl) {
  switch (hl) {
    case HL_STRING: return 35;
    case HL_NUMBER: return 31;
    case HL_MATCH: return 34;
    default: return 37;
  }
}
```

We’re coloring strings magenta (35).

Now let’s add an HL_HIGHLIGHT_STRINGS bit flag to the flags field of the editorSyntax struct, and turn on the flag when highlighting C files.

```c
#define HL_HIGHLIGHT_STRINGS (1<<1)

struct editorSyntax HLDB[] = {
  {
    "c",
    C_HL_extensions,
    HL_HIGHLIGHT_NUMBERS | HL_HIGHLIGHT_STRINGS
  },
};
```

Now for the actual highlighting code. We will use an in_string variable to keep track of whether we are currently inside a string. If we are, then we’ll keep highlighting the current character as a string until we hit the closing quote.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  int prev_sep = 1;
  int in_string = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

As you can see, we highlight both double-quoted strings and single-quoted strings (sorry Lispers/Rustaceans). We actually store either a double-quote (") or a single-quote (') character as the value of in_string, so that we know which one closes the string.

So, going through the code from top to bottom: If in_string is set, then we know the current character can be highlighted with HL_STRING. Then we check if the current character is the closing quote (c == in_string), and if so, we reset in_string to 0. Then, since we highlighted the current character, we have to consume it by incrementing i and continueing out of the current loop iteration. We also set prev_sep to 1 so that if we’re done highlighting the string, the closing quote is considered a separator.

If we’re not currently in a string, then we have to check if we’re at the beginning of one by checking for a double- or single-quote. If we are, we store the quote in in_string, highlight it with HL_STRING, and consume it.

We should probably take escaped quotes into account when highlighting strings. If the sequence \' or \" occurs in a string, then the escaped quote doesn’t close the string in the vast majority of languages.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  int prev_sep = 1;
  int in_string = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

<br>

#### 6.8. Colorful single-luine comments.

Next let’s highlight single-line comments. (We’ll leave multi-line comments until the end, because they’re complicated.)

```c
enum editorHighlight {
  HL_NORMAL = 0,
  HL_COMMENT,
  HL_STRING,
  HL_NUMBER,
  HL_MATCH
};

int editorSyntaxToColor(int hl) {
  switch (hl) {
    case HL_COMMENT: return 36;
    case HL_STRING: return 35;
    case HL_NUMBER: return 31;
    case HL_MATCH: return 34;
    default: return 37;
  }
}
```

Comments will be highlighted in cyan (36).

We’ll let each language specify its own single-line comment pattern, as they differ a lot between languages. Let’s add a singleline_comment_start string to the editorSyntax struct, and set it to "//" for the C filetype.

```c
struct editorSyntax {
  char *filetype;
  char **filematch;
  char *singleline_comment_start;
  int flags;
};

struct editorSyntax HLDB[] = {
  {
    "c",
    C_HL_extensions,
    "//",
    HL_HIGHLIGHT_NUMBERS | HL_HIGHLIGHT_STRINGS
  },
};
```

Now for the highlighting code:

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  char *scs = E.syntax->singleline_comment_start;
  int scs_len = scs ? strlen(scs) : 0;
  int prev_sep = 1;
  int in_string = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (scs_len && !in_string) {
      if (!strncmp(&row->render[i], scs, scs_len)) {
        memset(&row->hl[i], HL_COMMENT, row->rsize - i);
        break;
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

strncmp() comes from *string.h*.

If you don’t want single-line comment highlighting for a particular filetype, you should be able to set singleline_comment_start either to NULL or to the empty string (""). We make scs an alias for E.syntax->singleline_comment_start for easier typing (and readability, perhaps?). We then set scs_len to the length of the string, or 0 if the string is NULL. This lets us use scs_len as a boolean to know whether we should highlight single-line comments.

So we wrap our comment highlighting code in an if statement that checks scs_len and also makes sure we’re not in a string, since we’re placing this code above the string highlighting code (order matters a lot in this function).

If those checks passed, then we use strncmp() to check if this character is the start of a single-line comment. If so, then we simply memset() the whole rest of the line with HL_COMMENT and break out of the syntax highlighting loop. Just like that, we’re done highlighting the line.

<br>

#### 6.9. Colorful keywords.

Now let’s turn to highlighting keywords. We’re going to allow languages to specify two types of keywords that will be highlighted in different colors. (In C, we’ll highlight actual keywords in one color and common type names in the other color.)

```c
enum editorHighlight {
  HL_NORMAL = 0,
  HL_COMMENT,
  HL_KEYWORD1,
  HL_KEYWORD2,
  HL_STRING,
  HL_NUMBER,
  HL_MATCH
};

int editorSyntaxToColor(int hl) {
  switch (hl) {
    case HL_COMMENT: return 36;
    case HL_KEYWORD1: return 33;
    case HL_KEYWORD2: return 32;
    case HL_STRING: return 35;
    case HL_NUMBER: return 31;
    case HL_MATCH: return 34;
    default: return 37;
  }
}
```

The two colors we’ll use for keywords are yellow (33) and green (32).

Let’s add a keywords array to the editorSyntax struct. This will be a NULL-terminated array of strings, each string containing a keyword. To differentiate between the two types of keywords, we’ll terminate the second type of keywords with a pipe (|) character (also known as a vertical bar).

```c
struct editorSyntax {
  char *filetype;
  char **filematch;
  char **keywords;
  char *singleline_comment_start;
  int flags;
};

char *C_HL_keywords[] = {
  "switch", "if", "while", "for", "break", "continue", "return", "else",
  "struct", "union", "typedef", "static", "enum", "class", "case",
  "int|", "long|", "double|", "float|", "char|", "unsigned|", "signed|",
  "void|", NULL
};

struct editorSyntax HLDB[] = {
  {
    "c",
    C_HL_extensions,
    C_HL_keywords,
    "//",
    HL_HIGHLIGHT_NUMBERS | HL_HIGHLIGHT_STRINGS
  },
};
```

As mentioned earlier, we’ll highlight common C types as secondary keywords, so we end each one with a | character.

Now let’s highlight them.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  char **keywords = E.syntax->keywords;
  char *scs = E.syntax->singleline_comment_start;
  int scs_len = scs ? strlen(scs) : 0;
  int prev_sep = 1;
  int in_string = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (scs_len && !in_string) {
      if (!strncmp(&row->render[i], scs, scs_len)) {
        memset(&row->hl[i], HL_COMMENT, row->rsize - i);
        break;
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    if (prev_sep) {
      int j;
      for (j = 0; keywords[j]; j++) {
        int klen = strlen(keywords[j]);
        int kw2 = keywords[j][klen - 1] == '|';
        if (kw2) klen--;
        if (!strncmp(&row->render[i], keywords[j], klen) &&
            is_separator(row->render[i + klen])) {
          memset(&row->hl[i], kw2 ? HL_KEYWORD2 : HL_KEYWORD1, klen);
          i += klen;
          break;
        }
      }
      if (keywords[j] != NULL) {
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

First, at the top of the function we make keywords an alias for E.syntax->keywords since we’ll be using it a lot, and in some pretty dense code.

Keywords require a separator both before and after the keyword. Otherwise, the void in avoid, voided, or avoidable would be highlighted as a keyword, which is definitely a problem we want to, uh, circumnavigate.

So we check prev_sep to make sure a separator came before the keyword, before looping through each possible keyword. For each keyword, we store the length in klen and whether it’s a secondary keyword in kw2, in which case we decrement klen to account for the extraneous | character.

We then use strncmp() to check if the keyword exists at our current position in the text, and we check to see if a separator character comes after the keyword. Since \0 is considered a separator character, this works if the keyword is at the very end of the line.

If all that passed, then we have a keyword to highlight. We use memset() to highlight the whole keyword at once, highlighting it with HL_KEYWORD1 or HL_KEYWORD2 depending on the value of kw2. We then consume the entire keyword by incrementing i by the length of the keyword. Then we break instead of continueing, because we are in an inner loop, so we have to break out of that loop before continueing the outer loop. That is why, after the for loop, we check if the loop was broken out of by seeing if it got to the terminating NULL value, and if it was broken out of, we continue.

<br>

#### 6.10. Nonprintable characters.

Before we tackle highlighting multi-line comments, let’s take a quick break from editorUpdateSyntax().

We’re going to display nonprintable characters in a more user-friendly way. Currently, nonprintable characters completely mess up the rendering that our editor does. Just try running kilo and passing itself in as an argument. That is, open the kilo executable file using kilo. And try moving the cursor around, and typing. It’s not pretty. Every keypress causes the terminal to ding, because the audible bell character (7) is being printed out. Strings containing terminal escape sequences in our code are being printed out as actual escape sequences, because that’s how they’re stored in a raw executable.

To prevent all that, we’re going to translate nonprintable characters into printable ones. We’ll render the alphabetic control characters (Ctrl-A = 1, Ctrl-B = 2, …, Ctrl-Z = 26) as the capital letters A through Z. We’ll also render the 0 byte like a control character. Ctrl-@ = 0, so we’ll render it as an @ sign. Finally, any other nonprintable characters we’ll render as a question mark (?). And to differentiate these characters from their printable counterparts, we’ll render them using inverted colors (black on white).

```c

void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      char *c = &E.row[filerow].render[E.coloff];
      unsigned char *hl = &E.row[filerow].hl[E.coloff];
      int current_color = -1;
      int j;
      for (j = 0; j < len; j++) {
        if (iscntrl(c[j])) {
          char sym = (c[j] <= 26) ? '@' + c[j] : '?';
          abAppend(ab, "\x1b[7m", 4);
          abAppend(ab, &sym, 1);
          abAppend(ab, "\x1b[m", 3);
        } else if (hl[j] == HL_NORMAL) {
          if (current_color != -1) {
            abAppend(ab, "\x1b[39m", 5);
            current_color = -1;
          }
          abAppend(ab, &c[j], 1);
        } else {
          int color = editorSyntaxToColor(hl[j]);
          if (color != current_color) {
            current_color = color;
            char buf[16];
            int clen = snprintf(buf, sizeof(buf), "\x1b[%dm", color);
            abAppend(ab, buf, clen);
          }
          abAppend(ab, &c[j], 1);
        }
      }
      abAppend(ab, "\x1b[39m", 5);
    }
    abAppend(ab, "\x1b[K", 3);
    abAppend(ab, "\r\n", 2);
  }
}
```

We use iscntrl() to check if the current character is a control character. If so, we translate it into a printable character by adding its value to '@' (in ASCII, the capital letters of the alphabet come after the @ character), or using the '?' character if it’s not in the alphabetic range.

We then use the '7m' escape sequence to switch to inverted colors before printing the translated symbol. We use <esc>[m to turn off inverted colors again.

Unfortunately, 'm' turns off all text formatting, including colors. So let’s print the escape sequence for the current color afterwards.

```c
void editorDrawRows(struct abuf *ab) {
  int y;
  for (y = 0; y < E.screenrows; y++) {
    int filerow = y + E.rowoff;
    if (filerow >= E.numrows) {
      if (E.numrows == 0 && y == E.screenrows / 3) {
        char welcome[80];
        int welcomelen = snprintf(welcome, sizeof(welcome),
          "Kilo editor -- version %s", KILO_VERSION);
        if (welcomelen > E.screencols) welcomelen = E.screencols;
        int padding = (E.screencols - welcomelen) / 2;
        if (padding) {
          abAppend(ab, "~", 1);
          padding--;
        }
        while (padding--) abAppend(ab, " ", 1);
        abAppend(ab, welcome, welcomelen);
      } else {
        abAppend(ab, "~", 1);
      }
    } else {
      int len = E.row[filerow].rsize - E.coloff;
      if (len < 0) len = 0;
      if (len > E.screencols) len = E.screencols;
      char *c = &E.row[filerow].render[E.coloff];
      unsigned char *hl = &E.row[filerow].hl[E.coloff];
      int current_color = -1;
      int j;
      for (j = 0; j < len; j++) {
        if (iscntrl(c[j])) {
          char sym = (c[j] <= 26) ? '@' + c[j] : '?';
          abAppend(ab, "\x1b[7m", 4);
          abAppend(ab, &sym, 1);
          abAppend(ab, "\x1b[m", 3);
          if (current_color != -1) {
            char buf[16];
            int clen = snprintf(buf, sizeof(buf), "\x1b[%dm", current_color);
            abAppend(ab, buf, clen);
          }
        } else if (hl[j] == HL_NORMAL) {
          if (current_color != -1) {
            abAppend(ab, "\x1b[39m", 5);
            current_color = -1;
          }
          abAppend(ab, &c[j], 1);
        } else {
          int color = editorSyntaxToColor(hl[j]);
          if (color != current_color) {
            current_color = color;
            char buf[16];
            int clen = snprintf(buf, sizeof(buf), "\x1b[%dm", color);
            abAppend(ab, buf, clen);
          }
          abAppend(ab, &c[j], 1);
        }
      }
      abAppend(ab, "\x1b[39m", 5);
    }
    abAppend(ab, "\x1b[K", 3);
    abAppend(ab, "\r\n", 2);
  }
}
```

<br>

#### 6.11. Colorful multiline comments.

Okay, we have one last feature to implement: multi-line comment highlighting. Let’s start by adding HL_MLCOMMENT to the editorHighlight enum.

```c
enum editorHighlight {
  HL_NORMAL = 0,
  HL_COMMENT,
  HL_MLCOMMENT,
  HL_KEYWORD1,
  HL_KEYWORD2,
  HL_STRING,
  HL_NUMBER,
  HL_MATCH
};

int editorSyntaxToColor(int hl) {
  switch (hl) {
    case HL_COMMENT:
    case HL_MLCOMMENT: return 36;
    case HL_KEYWORD1: return 33;
    case HL_KEYWORD2: return 32;
    case HL_STRING: return 35;
    case HL_NUMBER: return 31;
    case HL_MATCH: return 34;
    default: return 37;
  }
}
```

We’ll highlight multi-line comments to be the same color as single-line comments (cyan).

Now we’ll add two strings to editorSyntax: multiline_comment_start and multiline_comment_end. In C, these will be "/*" and "*/".

```c
struct editorSyntax {
  char *filetype;
  char **filematch;
  char **keywords;
  char *singleline_comment_start;
  char *multiline_comment_start;
  char *multiline_comment_end;
  int flags;
};

struct editorSyntax HLDB[] = {
  {
    "c",
    C_HL_extensions,
    C_HL_keywords,
    "//", "/*", "*/",
    HL_HIGHLIGHT_NUMBERS | HL_HIGHLIGHT_STRINGS
  },
};
```

Now let’s open editorUpdateSyntax() up once again. We’ll add mcs and mce aliases that are analogous to the scs alias we already have for single-line comments. We’ll also add mcs_len and mce_len.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  char **keywords = E.syntax->keywords;
  char *scs = E.syntax->singleline_comment_start;
  char *mcs = E.syntax->multiline_comment_start;
  char *mce = E.syntax->multiline_comment_end;
  int scs_len = scs ? strlen(scs) : 0;
  int mcs_len = mcs ? strlen(mcs) : 0;
  int mce_len = mce ? strlen(mce) : 0;
  int prev_sep = 1;
  int in_string = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (scs_len && !in_string) {
      if (!strncmp(&row->render[i], scs, scs_len)) {
        memset(&row->hl[i], HL_COMMENT, row->rsize - i);
        break;
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    if (prev_sep) {
      int j;
      for (j = 0; keywords[j]; j++) {
        int klen = strlen(keywords[j]);
        int kw2 = keywords[j][klen - 1] == '|';
        if (kw2) klen--;
        if (!strncmp(&row->render[i], keywords[j], klen) &&
            is_separator(row->render[i + klen])) {
          memset(&row->hl[i], kw2 ? HL_KEYWORD2 : HL_KEYWORD1, klen);
          i += klen;
          break;
        }
      }
      if (keywords[j] != NULL) {
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

Now for highliting the code:

```c

void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  char **keywords = E.syntax->keywords;
  char *scs = E.syntax->singleline_comment_start;
  char *mcs = E.syntax->multiline_comment_start;
  char *mce = E.syntax->multiline_comment_end;
  int scs_len = scs ? strlen(scs) : 0;
  int mcs_len = mcs ? strlen(mcs) : 0;
  int mce_len = mce ? strlen(mce) : 0;
  int prev_sep = 1;
  int in_string = 0;
  int in_comment = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (scs_len && !in_string) {
      if (!strncmp(&row->render[i], scs, scs_len)) {
        memset(&row->hl[i], HL_COMMENT, row->rsize - i);
        break;
      }
    }
    if (mcs_len && mce_len && !in_string) {
      if (in_comment) {
        row->hl[i] = HL_MLCOMMENT;
        if (!strncmp(&row->render[i], mce, mce_len)) {
          memset(&row->hl[i], HL_MLCOMMENT, mce_len);
          i += mce_len;
          in_comment = 0;
          prev_sep = 1;
          continue;
        } else {
          i++;
          continue;
        }
      } else if (!strncmp(&row->render[i], mcs, mcs_len)) {
        memset(&row->hl[i], HL_MLCOMMENT, mcs_len);
        i += mcs_len;
        in_comment = 1;
        continue;
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    if (prev_sep) {
      int j;
      for (j = 0; keywords[j]; j++) {
        int klen = strlen(keywords[j]);
        int kw2 = keywords[j][klen - 1] == '|';
        if (kw2) klen--;
        if (!strncmp(&row->render[i], keywords[j], klen) &&
            is_separator(row->render[i + klen])) {
          memset(&row->hl[i], kw2 ? HL_KEYWORD2 : HL_KEYWORD1, klen);
          i += klen;
          break;
        }
      }
      if (keywords[j] != NULL) {
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

First we add an in_comment boolean variable to keep track of whether we’re currently inside a multi-line comment (this variable isn’t used for single-line comments).

Moving down into the while loop, we require both mcs and mce to be non-NULL strings of length greater than 0 in order to turn on multi-line comment highlighting. We also check to make sure we’re not in a string, because having /* inside a string doesn’t start a comment in most languages. Okay, I’ll say it: all languages.

If we’re currently in a multi-line comment, then we can safely highlight the current character with HL_MLCOMMENT. Then we check if we’re at the end of a multi-line comment by using strncmp() with mce. If so, we use memset() to highlight the whole mce string with HL_MLCOMMENT, and then we consume it. If we’re not at the end of the comment, we simply consume the current character which we already highlighted.

If we’re not currently in a multi-line comment, then we use strncmp() with mcs to check if we’re at the beginning of a multi-line comment. If so, we use memset() to highlight the whole mcs string with HL_MLCOMMENT, set in_comment to true, and consume the whole mcs string.

Now let’s fix a bit of a complication that multi-line comments add: single-line comments should not be recognized inside multi-line comments.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  char **keywords = E.syntax->keywords;
  char *scs = E.syntax->singleline_comment_start;
  char *mcs = E.syntax->multiline_comment_start;
  char *mce = E.syntax->multiline_comment_end;
  int scs_len = scs ? strlen(scs) : 0;
  int mcs_len = mcs ? strlen(mcs) : 0;
  int mce_len = mce ? strlen(mce) : 0;
  int prev_sep = 1;
  int in_string = 0;
  int in_comment = 0;
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (scs_len && !in_string && !in_comment) {
      if (!strncmp(&row->render[i], scs, scs_len)) {
        memset(&row->hl[i], HL_COMMENT, row->rsize - i);
        break;
      }
    }
    if (mcs_len && mce_len && !in_string) {
      if (in_comment) {
        row->hl[i] = HL_MLCOMMENT;
        if (!strncmp(&row->render[i], mce, mce_len)) {
          memset(&row->hl[i], HL_MLCOMMENT, mce_len);
          i += mce_len;
          in_comment = 0;
          prev_sep = 1;
          continue;
        } else {
          i++;
          continue;
        }
      } else if (!strncmp(&row->render[i], mcs, mcs_len)) {
        memset(&row->hl[i], HL_MLCOMMENT, mcs_len);
        i += mcs_len;
        in_comment = 1;
        continue;
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    if (prev_sep) {
      int j;
      for (j = 0; keywords[j]; j++) {
        int klen = strlen(keywords[j]);
        int kw2 = keywords[j][klen - 1] == '|';
        if (kw2) klen--;
        if (!strncmp(&row->render[i], keywords[j], klen) &&
            is_separator(row->render[i + klen])) {
          memset(&row->hl[i], kw2 ? HL_KEYWORD2 : HL_KEYWORD1, klen);
          i += klen;
          break;
        }
      }
      if (keywords[j] != NULL) {
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
}
```

Okay, now let’s work on highlighting multi-line comments that actually span over multiple lines. To do this, we need to know if the previous line is part of an unclosed multi-line comment. Let’s add an hl_open_comment boolean variable to the erow struct. Let’s also add an idx integer variable, so that each erow knows its own index within the file. That will allow each row to examine the previous row’s hl_open_comment value.

```c
typedef struct erow {
  int idx;
  int size;
  int rsize;
  char *chars;
  char *render;
  unsigned char *hl;
  int hl_open_comment;
} erow;

void editorInsertRow(int at, char *s, size_t len) {
  if (at < 0 || at > E.numrows) return;
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  memmove(&E.row[at + 1], &E.row[at], sizeof(erow) * (E.numrows - at));
  E.row[at].idx = at;
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  E.row[at].hl = NULL;
  E.row[at].hl_open_comment = 0;
  editorUpdateRow(&E.row[at]);
  E.numrows++;
  E.dirty++;
}
```

We initialize idx to the row’s index in the file at the time it is inserted. Let’s make sure to update the idx of each row whenever a row is inserted into or removed from the file.

```c
void editorInsertRow(int at, char *s, size_t len) {
  if (at < 0 || at > E.numrows) return;
  E.row = realloc(E.row, sizeof(erow) * (E.numrows + 1));
  memmove(&E.row[at + 1], &E.row[at], sizeof(erow) * (E.numrows - at));
  for (int j = at + 1; j <= E.numrows; j++) E.row[j].idx++;
  E.row[at].idx = at;
  E.row[at].size = len;
  E.row[at].chars = malloc(len + 1);
  memcpy(E.row[at].chars, s, len);
  E.row[at].chars[len] = '\0';
  E.row[at].rsize = 0;
  E.row[at].render = NULL;
  E.row[at].hl = NULL;
  E.row[at].hl_open_comment = 0;
  editorUpdateRow(&E.row[at]);
  E.numrows++;
  E.dirty++;
}

void editorDelRow(int at) {
  if (at < 0 || at >= E.numrows) return;
  editorFreeRow(&E.row[at]);
  memmove(&E.row[at], &E.row[at + 1], sizeof(erow) * (E.numrows - at - 1));
  for (int j = at; j < E.numrows - 1; j++) E.row[j].idx--;
  E.numrows--;
  E.dirty++;
}
```

The for loops update the index of each row that was displaced by the insert or delete operation.

Now, the final step.

```c
void editorUpdateSyntax(erow *row) {
  row->hl = realloc(row->hl, row->rsize);
  memset(row->hl, HL_NORMAL, row->rsize);
  if (E.syntax == NULL) return;
  char **keywords = E.syntax->keywords;
  char *scs = E.syntax->singleline_comment_start;
  char *mcs = E.syntax->multiline_comment_start;
  char *mce = E.syntax->multiline_comment_end;
  int scs_len = scs ? strlen(scs) : 0;
  int mcs_len = mcs ? strlen(mcs) : 0;
  int mce_len = mce ? strlen(mce) : 0;
  int prev_sep = 1;
  int in_string = 0;
  int in_comment = (row->idx > 0 && E.row[row->idx - 1].hl_open_comment);
  int i = 0;
  while (i < row->rsize) {
    char c = row->render[i];
    unsigned char prev_hl = (i > 0) ? row->hl[i - 1] : HL_NORMAL;
    if (scs_len && !in_string && !in_comment) {
      if (!strncmp(&row->render[i], scs, scs_len)) {
        memset(&row->hl[i], HL_COMMENT, row->rsize - i);
        break;
      }
    }
    if (mcs_len && mce_len && !in_string) {
      if (in_comment) {
        row->hl[i] = HL_MLCOMMENT;
        if (!strncmp(&row->render[i], mce, mce_len)) {
          memset(&row->hl[i], HL_MLCOMMENT, mce_len);
          i += mce_len;
          in_comment = 0;
          prev_sep = 1;
          continue;
        } else {
          i++;
          continue;
        }
      } else if (!strncmp(&row->render[i], mcs, mcs_len)) {
        memset(&row->hl[i], HL_MLCOMMENT, mcs_len);
        i += mcs_len;
        in_comment = 1;
        continue;
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_STRINGS) {
      if (in_string) {
        row->hl[i] = HL_STRING;
        if (c == '\\' && i + 1 < row->rsize) {
          row->hl[i + 1] = HL_STRING;
          i += 2;
          continue;
        }
        if (c == in_string) in_string = 0;
        i++;
        prev_sep = 1;
        continue;
      } else {
        if (c == '"' || c == '\'') {
          in_string = c;
          row->hl[i] = HL_STRING;
          i++;
          continue;
        }
      }
    }
    if (E.syntax->flags & HL_HIGHLIGHT_NUMBERS) {
      if ((isdigit(c) && (prev_sep || prev_hl == HL_NUMBER)) ||
          (c == '.' && prev_hl == HL_NUMBER)) {
        row->hl[i] = HL_NUMBER;
        i++;
        prev_sep = 0;
        continue;
      }
    }
    if (prev_sep) {
      int j;
      for (j = 0; keywords[j]; j++) {
        int klen = strlen(keywords[j]);
        int kw2 = keywords[j][klen - 1] == '|';
        if (kw2) klen--;
        if (!strncmp(&row->render[i], keywords[j], klen) &&
            is_separator(row->render[i + klen])) {
          memset(&row->hl[i], kw2 ? HL_KEYWORD2 : HL_KEYWORD1, klen);
          i += klen;
          break;
        }
      }
      if (keywords[j] != NULL) {
        prev_sep = 0;
        continue;
      }
    }
    prev_sep = is_separator(c);
    i++;
  }
  int changed = (row->hl_open_comment != in_comment);
  row->hl_open_comment = in_comment;
  if (changed && row->idx + 1 < E.numrows)
    editorUpdateSyntax(&E.row[row->idx + 1]);
}
```

Near the top of editorUpdateSyntax(), we initialize in_comment to true if the previous row has an unclosed multi-line comment. If that’s the case, then the current row will start out being highlighted as a multi-line comment.

At the bottom of editorUpdateSyntax(), we set the value of the current row’s hl_open_comment to whatever state in_comment got left in after processing the entire row. That tells us whether the row ended as an unclosed multi-line comment or not.


<br>