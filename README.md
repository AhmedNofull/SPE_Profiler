# Path profiler for C programs

CSE433 Software Performance Evaluation, Major Assignment, Fall 2026
Dr. Islam El-Maddah

| ID | Name | Program |
|----|------|---------|
| 23P0439 | Karim Samer Mostafa Ragab | CAIE |
| 23P0438 | Omar Gamil Anwar Osman | CAIE |
| 2301081 | Youssef Maged Morees Ageb Farag | CAIE |
| 2301144 | Mohamed Amir Kamal | CAIE |
| 22P0255 | Ahmed Abdallah Hassan Ahmed Nofull | CESS |

You add two macros to every function of a C program. When the program runs, the
profiler records every distinct call path (for example `main -> f1 -> square`),
how many times each path ran, and its inclusive and exclusive time. It also logs
every call and return so you can see which function called which.

The full write-up with the running examples is in `report/SPE_Report.pdf`.

## Folder layout

```
src/profiler.h, src/profiler.c   the profiler (instrumentation code)
tests/doctor_program.c           square/cube/f1/f2 program from the lecture sheet
tests/grades_program.c           reads tests/scores.txt, has early returns
tests/example_program.c          small 5-function demo
tests/scores.txt                 input text file for grades_program
output/<program>/                what each test produced:
    path_profiles.txt            path table, calling context tree, per-function times
    event_log.txt                every CALL/RETURN + caller -> callee counts
    call_trace.txt               indented enter/exit trace
    console_output.txt           what the program printed
report/SPE_Report.pdf            the report
Makefile, build.bat              build + run everything
```

## Build and run

Windows (MinGW-w64 or MSYS2 gcc on PATH): double-click `build.bat`, or run it from
a command prompt.

Linux / macOS:

```
make run
```

Both build the three test programs and run them. The results go into
`output/<program>/`. To build one program by hand:

```
gcc -std=c99 -Wall -O2 -Isrc tests/doctor_program.c src/profiler.c -o doctor_program
```

The program writes its three output files into whatever folder you run it from.

## Instrumenting your own program

```c
#include "profiler.h"

int helper(int x)
{
    FUNC_ENTRY();
    if (x < 0) {
        FUNC_EXIT();          /* every return needs one */
        return 0;
    }
    int r = x * 2;
    FUNC_EXIT();
    return r;
}

int main(void)
{
    profiler_init();          /* first line of main */
    FUNC_ENTRY();
    helper(5);
    FUNC_EXIT();
    profiler_finish();        /* writes the files */
    return 0;
}
```

If you forget a `FUNC_EXIT()` before a return, the profiler prints a warning and
closes that call when the caller exits, so the numbers for the rest of the
program stay correct.

## Limits

Recursive functions and C++ member functions are out of scope, as the
assignment allows. The program must be single-threaded. It handles up to 512
distinct paths and a call depth of 64 (both set in `profiler.h`), and it prints
a warning if either limit is reached. The number of calls is not limited.
