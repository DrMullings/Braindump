---
layout: post
title:  "Obfuscation and code optimization"
date:   2021-09-29 15:45:00 +0100
categories: cheatsheets
---

# Code optimization

All of the methods can be used iteratively

| Name | Desccription |
| propagation of constants | variables that do not change their value at runtime are being made constant |
| dead code elimination | code that is never used will not be translated |
| inlining | code of subroutines is put directly at the place of execution |

Loop optimizations:

| unswitching | case distinction is moved out of the loop and the loop is being put in each branch of the case distinction |
| loop unrolling | if the execution of a loop is known at compile time, the content of the loop is multiplied and the executions of the loop are reduced |
| loop inversion | converting head controlled to foot controlled loops |

Control flow optimization

| branch to branch elimination | multiple nested case distinctions are seperated in single cases and go-tos |
| sub expression  elimination | expressions that appear multiple times are only evaluated once |
| branchless code | conditional jumps are transformed to equivalent code, mostly using sbb |
| idioms | abusing commands for other than the intended purpose |
| frame pointer omission | if the offset of parameters and variables is defined at compile time their addressing can be done with esp, this frees ebp for usage as general purpose register |
| hot and cold parts | if it is possible to determine which parts of the code are more likely to be executed at compile time those (hot) parts stay in the cache |

# Obfuscation


| string obfuscation | strings are being encrypted to avoid giving away information about the code |
| junk code insertion | useless instrustions are inserted, unlike dead code junk code is being executed |
| code permutation | basically idioms used with different intention |

Opaque predicates 

conditions where the evaluations alway return the same value, the other branches increase the complexity of the code

| bogus control flow / dead code | the else branch of an "always true" condition is never executed, but increses the  complexity |
| fake loop | loops that are never executed increase the complexity of the code |
| random predicates  | branches with the same logic are randomly executed |

| number disguising | numbers can be disguised in calculations or lookup table |
| merging | two subroutines that are not logically related are merged in one function |
| splitting | the counter part to merging:  subroutines are split in multiple smaller subroutines |
| control flow flattening (chenxify) | the control flow is flattened to increase complexity (e.g. with switch case) |
| import hiding | libraries are loaded dynamically, best used with string obfuscation |
| aliasing | code is hidden in alias functions that seem never to be executed |
| structured exception handling | tbd |

Prevention of disassembly

| polymorphy | tbd  |
| metamorphy | tbd  |
| self modifying code | tbd  |
| unaligned branches | tbd  |

