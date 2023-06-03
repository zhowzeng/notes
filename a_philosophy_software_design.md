# A Philosophy of Software Design

* 作者：John Ousterhout
* 中文翻譯：<https://go7hic.github.io/A-Philosophy-of-Software-Design/#/>

## 總結

### Summary of Design Principles

1. Complexity is incremental: you have to sweat the small stuff (see p. 11).
1. Working code isn’t enough (see p. 14).
1. Make continual small investments to improve system design (see p. 15).
1. Modules should be deep (see p. 22)
1. Interfaces should be designed to make the most common usage as simple as possible (see p. 27).
1. It’s more important for a module to have a simple interface than a simple implementation (see pp. 55, 71).
1. General-purpose modules are deeper (see p. 39).
1. Separate general-purpose and special-purpose code (see p. 62).
1. Different layers should have different abstractions (see p. 45).
1. Pull complexity downward (see p. 55).
1. Define errors (and special cases) out of existence (see p. 79).
1. Design it twice (see p. 91).
1. Comments should describe things that are not obvious from the code (see p. 101).
1. Software should be designed for ease of reading, not ease of writing (see p. 149).
1. The increments of software development should be abstractions, not features (see p. 154).

### Summary of Red Flags

1. Shallow Module: the interface for a class or method isn’t much simpler than its implementation (see pp. 25, 110).
1. Information Leakage: a design decision is reflected in multiple modules (see p. 31).
1. Temporal Decomposition: the code structure is based on the order in which operations are executed, not on information hiding (see p. 32).
1. Overexposure: An API forces callers to be aware of rarely used features in order to use commonly used features (see p. 36).
1. Pass-Through Method: a method does almost nothing except pass its arguments to another method with a similar signature (see p. 46).
1. Repetition: a nontrivial piece of code is repeated over and over (see p. 62).
1. Special-General Mixture: special-purpose code is not cleanly separated from general purpose code (see p. 65).
1. Conjoined Methods: two methods have so many dependencies that its hard to understand the implementation of one without understanding the implementation of the other (see p. 72).
1. Comment Repeats Code: all of the information in a comment is immediately obvious from the code next to the comment (see p. 104).
1. Implementation Documentation Contaminates Interface: an interface comment describes implementation details not needed by users of the thing being documented (see p. 114).
1. Vague Name: the name of a variable or method is so imprecise that it doesn’t convey much useful information (see p. 123).
1. Hard to Pick Name: it is difficult to come up with a precise and intuitive name for an entity (see p. 125).
1. Hard to Describe: in order to be complete, the documentation for a variable or method must be long. (see p. 131).
1. Nonobvious Code: the behavior or meaning of a piece of code cannot be understood easily. (see p. 148).
