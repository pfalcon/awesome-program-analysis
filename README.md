
Intermediate Representation Forms/Types
=======================================

* Imperative
* Functional
  * Static Single-Assignment (SSA) - As argued by Appel, SSA is a functional representation.
  * (Lambda-)Functional
  * Continuation-passing Style (CPS)

SSA Form
========

Forms:
* Maximal
  * Defined e.g. by Appel:
    > A really crude approach is to split every variable at every basic-
    > block boundary, and put φ-functions for every variable in every block.
* Minimal
  * This is usually what's sought for SSA form, where there're no superflous
* Semi-pruned
* Pruned
  * Minimal form with dead Phi functions removed (i.e. Phi functions which
    reference which reference variables which will no longer be used in the
    rest of program; they just define dead variables likewise). Variable
    liveness is required for construction, so is considered expensive to
    construct.

History
-------

* 1969

According to Aycock/Horspool:

> The genesis of SSA form was in the 1960s with the work of Shapiro and
Saint [23,19]. Their conversion algorithm was based upon finding equivalence
classes of variables by walking the control-flow graph.

R. M. Shapiro and H. Saint. The Representation of Algorithms. Rome Air
Development Center TR-69-313, Volume II, September 1969.

> Given the possibility of concurrent operation, we might also wish to question the automatic one-one mapping of variable names to equipment locations. Two uses of the same variable name might be entirely unrelated in terms of data dependency and thus potentially concurrent if mapped to different equipment locations.

Continues on the p.31 of the paper (p.39 of the PDF) under the title:

> VI. Variable-Names and Data Dependency Relations

* 1988

Then, following Wikipedia, "SSA was proposed by Barry K. Rosen, Mark N. Wegman, and F. Kenneth Zadeck in 1988."
Barry Rosen; Mark N. Wegman; F. Kenneth Zadeck (1988). "Global value numbers and redundant computations"

Construction Algorithms
-----------------------

Based on exceprts from "Simple Generation of Static Single-Assignment Form", 
Aycock/Horspool

### For Reducible CFGs (i.e. special case)

* 1986 R. Cytron, A. Lowry, K. Zadeck. Code Motion of Control Structures in High-Level
  Languages. Proceedings of the Thirteenth Annual ACM Symposium on Principles
  of Programming Languages, 1986, pp. 70–85.

  > Cytron, Lowry, and Zadeck [11] predate the use of φ-functions, and employ
  a heuristic placement policy based on the interval structure of the control-flow
  graph, similar to that of Rosen, Wegman, and Zadeck [22]. The latter work is
  interesting because they look for the same patterns as our algorithm does during
  our minimization phase. However, they do so after generating SSA form, and
  then only to correct ‘second order effects’ created during redundancy elimination.

* 1994 Single-Pass Generation of Static Single-Assignment Form for Structured Languages, Brandis and Mössenböck

  > Brandis and Mössenböck [5] generate SSA form in one pass for structured control-
  flow graphs, a subset of reducible control-flow graphs, by delicate placement of
  φ-functions. They describe how to extend their method to reducible control-flow
  graphs, but require the dominator tree to do so.

* 2000 Simple Generation of Static Single-Assignment Form, 2000, John Aycock and Nigel Horspool

  > In this paper we present a new, simple method
  for converting to SSA form, which produces correct solutions for nonre-
  ducible control-flow graphs, and produces minimal solutions for reducible
  ones.

  * 2016 Verified Construction of Static Single Assignment Form, Sebastian Buchwald,
    Denis Lohner, Sebastian Ullrich

### For Non-Reducible CFGs (i.e. general case)

* 1991 R. Cytron, J. Ferrante, B. K. Rosen, M. N. Wegman, and F. K. Zadeck. Efficiently
  Computing Static Single-Assignment Form and the Control Dependence Graph.
  ACM TOPLAS 13, 4 (October 1991), pp. 451–490.

  > "Canonical" SSA construction algorithm.

  * 1995 R. K. Cytron and J. Ferrante. Efficiently Computing φ-Nodes On-The-Fly. ACM
    TOPLAS 17, 3 (May 1995), pp. 487–506.

    > Cytron and Ferrante [9] later refined their method so that it runs in almost-linear time.

* 1994 R. Johnson, D. Pearson, and K. Pingali. The Program Structure Tree: Computing
  Control Regions in Linear Time. ACM PLDI ’94, pp. 171–185.

  > Johnson, Pearson, and Pingali [16] demonstrate conversion to SSA form as
  an application of their “program structure tree,” a decomposition of the control-
  flow graph into single-entry, single-exit regions. They claim that using this graph
  representation allows them to avoid areas in the control-flow graph that do not
  contribute to a solution.

* 1995 V. C. Sreedhar and G. R. Gao. A Linear Time Algorithm for Placing φ-Nodes.
  Proceedings of the Twenty-Second Annual ACM Symposium on Principles of
  Programming Languages, 1995, pp. 62–73.

  > Sreedhar and Gao [24] devised a linear-time algorithm for φ-function
  placement using DJ-graphs, a data structure which combines the dominator tree
  with information about where data flow in the program merges.
