Program Analysis and Transformation Survey and Links
====================================================

Table of Contents:

* [Names](#names)
* [Intermediate Representation Forms/Types](#intermediate-representation-formstypes)
* [SSA Form](#ssa-form)
  * [Classification of SSA types](#classification-of-ssa-types)
  * [History](#history)
  * [Construction Algorithms](#construction-algorithms)
  * [Deconstruction Algorithms](#deconstruction-algorithms)
* [Control Flow Analysis](#control-flow-analysis)
* [Alias Analysis](#alias-analysis)
* [Register Allocation](#register-allocation)
* [Projects](#projects)

Names
=====

As a dedication:

We study Program Analysis because it's objective and complex phenomena of
nature devoid of subjectivities of the mankind. But then, we can't separate
it from the work of great human minds who laid the paths in this area, whose
steps we now follow.

These are people who contributed to the Program Analysis field of study
(with all the apology to many more who are not listed here). The emphasis
here is on well-knownness and public availability of their works:

* Gregory Chaitin
* Jeffrey Ullman
* Alfred Aho
* [Keith Cooper](http://keith.rice.edu/)
  * thesis: 1983 "Interprocedural Data Flow Analysis in a Programming Environment"
* [Andrew Appel](https://www.cs.princeton.edu/~appel/)
  * thesis: 1985 "Compile-time Evaluation and Code Generation in Semantics-Directed Compilers"
  * book 1998: "Modern Compiler Implementation in ML/Java/C"
  * 2000: [Optimal Register Coalescing Challenge](https://www.cs.princeton.edu/~appel/coalesce/)
* [Preston Briggs](https://genealogy.math.ndsu.nodak.edu/id.php?id=94538)
  * thesis: 1992 "Register Allocation via Graph Coloring"
* [Clifford Click](http://cliffc.org/blog/sample-page/) [@cliffclick](https://github.com/cliffclick)
  * thesis: 1995 "Combining Analyses, Combining Optimizations"
* [Sebastian Hack](http://compilers.cs.uni-saarland.de/people/hack/)
  * thesis: 2006 "Register Allocation for Programs in SSA Form"
* [Matthias Braun](https://pp.ipd.kit.edu/personhp/matthias_braun.php) [@MatzeB](https://github.com/MatzeB)
  * thesis: 2006 "Heuristisches Auslagern in einem SSA-basierten Registerzuteiler" in German, "Heuristic spilling in an SSA-based register allocator"
* [Sebastian Buchwald](https://pp.ipd.kit.edu/personhp/sebastian_buchwald.php)
  * thesis: 2008 "Befehlsauswahl auf expliziten Abhangigkeitsgraphen" in German, "Instruction selection on explicit dependency graphs"
* [Florent Bouchez](http://florent.bouchez.free.fr/)
  * thesis: 2009 "A Study of Spilling and Coalescing in Register Allocation as Two Separate Phases"
* [Benoit Boissinot](https://bboissin.appspot.com/)
  * thesis: 2010 "Towards an SSA based compiler back-end: some interesting properties of SSA and its extensions"
* Quentin Colombet
  * thesis 2012: "Decoupled (SSA-based) Register Allocators: from Theory to Practice, Coping with Just-In-Time Compilation and Embedded Processors Constraints"

Intermediate Representation Forms/Types
=======================================

* Imperative
* Functional
  * Static Single-Assignment (SSA) - As argued by Appel, SSA is a functional representation.
  * (Lambda-)Functional
  * Continuation-passing Style (CPS)

SSA Form
========

Put simple, in an SSA program, each variable is (statically, syntactically)
assigned only once.

Wikipedia: https://en.wikipedia.org/wiki/Static_single_assignment_form

General reference: "SSA Book" aka "Static Single Assignment Book" aka
"SSA-based Compiler Design" is open, collaborative effort of many SSA
researchers to write a definitive reference for all things SSA.
* Direct download (new versions are published):
  http://ssabook.gforge.inria.fr/latest/book-full.pdf
* Download directory:
  http://ssabook.gforge.inria.fr/latest/
* GForge project: https://gforge.inria.fr/projects/ssabook/
* Subversion repository:
  https://gforge.inria.fr/scm/browser.php?group_id=1950

Classification of SSA types
---------------------------

* Axis 1: Minimality. There're 2 poles: fully minimal vs fully maximal SSA form.
  Between those, there's continuum of intermediate cases.
  * Fully maximal
    * Defined e.g. by Appel:
      > A really crude approach is to split every variable at every basic-block
      > boundary, and put φ-functions for every variable in every block.

      Maximal form is the most intuitive form for construction, gives the simplest
      algorithms for both phi insertion and variable renaming phases of the
      construction.
  * Optimized maximal
    * An obvious optimization of avoiding placing phi functions in blocks with
      a single predecessor, as they never needed there. While cuts the number
      of phi functions, makes renaming algorithm a bit more complex: while for
      maximal form renaming could process blocks in arbitrary order (because
      each of program's variables has a local definition in every basic block),
      optimized maximal form requires processing predecessor first for each
      such single-predecessor block.
  * Minimal for reducible CFGs
    * Some algorithms (e.g. optimized for simplicity) naturally produce minimal
      form only for reducible CFGs. Applied to non-reducible CFGs, they may
      generate extra Phi functions. There're usually extensions to such
      algorithms to generate minimal form even for non-reducible CFGs too (but
      such extensions may add noticeable complexity to otherwise "simple"
      algorithm). Examplem of such an algorithm ins 2013 Braun et al.
  * Fully minimal
    * This is usually what's sought for SSA form, where there're no superflous
      phi functions, based only on graph properties of the CFG (with consulting
      semantics of the underlying program).
* Axis 2: Prunedness. As argued (implied) by 2013 Braun et al., prunedness is
  a separate trait from minimality. E.g., their algorithm constructs not fully
  minimal, yet pruned form. Between pruned and non-pruned forms, there're
  intermediate types again.
  * Pruned
    * Minimal form can still have dead phi functions, i.e. phi functions which
      reference variables which are not actually used in the rest of the program.
      Note that such references are problematic, as they artificially extend live
      ranges of referenced variables. Likewise, it defines new variables which
      aren't really live. The pruned SSA form is devoid of the dead phi functions.
      Two obvious way to achieve this: a) perform live variable analysis prior to
      SSA construction and use it to avoid placing dead phi functions; b) run
      dead code elimination (DCE) pass after the construction (which requires
      live variable analysis first, this time on SSA form of the program already).
      Due to these additional passes, pruned SSA construction is more expensive
      than just the minimal form. Note that if we intend to run DCE pass on the
      program anyway, which is often happens, we don't really need to be concerned
      to *construct* pruned form, as we will get it after the DCE pass "for free".
      Except of course that minimal and especially maximal form require more
      space to store and more time to go thru it during DCE.
  * Semi-pruned
    * Sometimes called "Briggs-Minimal" form. A compromise between fully
      pruned and minimal form. From Wikipedia:
      > Semi-pruned SSA form[6] is an attempt to reduce the number of Φ
      functions without incurring the relatively high cost of computing
      live variable information. It is based on the following observation:
      if a variable is never live upon entry into a basic block, it never
      needs a Φ function. During SSA construction, Φ functions for any
      "block-local" variables are omitted.
  * Not pruned
* Axis 2: Conventional vs Transformed SSA
  * Conventional
    * Allows for easy deconstruction algorithm (literally, just drop
      SSA variables subscripts and remove Phi functions). Usually,
      after construction, SSA is in conventional form (if during
      construction, additional optimizations were not performed).
  * Transformed
    * Some optimizations applied to an SSA program make simple deconstruction
      algorithm outlined above not possible (not producing correct
      results). This is known as "transformed SSA". There're algorithms
      to convert transformed SSA into conventional form.
* Axis 3: Strict vs non-strict SSA
  * Non-strict SSA allows some variables to be undefined on some paths
    (just like conventional imperative programs).
  * Strict form requires each use to be dominated by definition. This
    in turn means that every variable must be explicitly initialized.
    Non-strict program can be trivially converted into strict form, by
    initializing variables with special values, like "undef" for truly
    undefined values, "param" for function paramters, etc. Most of SSA
    algorithms requires/assume strict SSA form, so non-strict is not
    further considered.


Discussion: There's one and true SSA type - the maximal one. It has a
straightforward, easy to understand construction algorithm which does
not dependent on any other special algorithms. Running a generic
DCE algorithm on it will remove any redundancies of the maximal form
(oftentimes, together with other dead code). All other types are
optimizations of the maximal form, allowing to generate less Phi
functions, so less are removed later. Optimizations are useful, but
the usual warning about premature optimization applies.

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

Based on excerpts from "Simple Generation of Static Single-Assignment Form",
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

  > In this paper we present a new, simple method for converting to SSA
  form, which produces correct solutions for nonreducible control-flow
  graphs, and produces minimal solutions for reducible ones.

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

* 2013 M. Braun, S. Buchwald, S. Hack, R. Leißa, C. Mallon, and
  A. Zwinkau. Simple and efficient construction of static single assignment
  form. In R. Jhala and K. Bosschere, editors, Compiler Construction,
  volume 7791 of Lecture Notes in Computer Science, pp.
  102–122. Springer, 2013. doi: 10.1007/978-3-642-37051-9_6.

  > Braun, et al present a simple SSA construction algorithm, which allows
  direct translation from an abstract syntax tree or bytecode into an
  SSA-based intermediate representation. The algorithm requires no prior
  analysis and ensures that even during construction the intermediate representation
  is in SSA form. This allows the application of SSA-based optimizations
  during construction. After completion, the intermediate representation
  is in minimal and pruned SSA form. In spite of its simplicity,
  the runtime of the algorithm is on par with Cytron et al.’s algorithm.

  * 2016 Verified Construction of Static Single Assignment Form, Sebastian Buchwald,
    Denis Lohner, Sebastian Ullrich

Deconstruction Algorithms
-------------------------

---
Epigraph (due to [Boissinot](https://bboissin.appspot.com/static/upload/bboissin-outssa-cgo09-slides.pdf), slide 20):

*Naively, a k-input Phi-function at entrance to a node X can
be replaced by k ordinary assignments, one at the end of
each control flow predecessor of X. This is always correct...*

-- *Cytron, Ferrante, Rosen, Wegman, Zadeck (1991)
Efficiently computing static single assignment form and the control
dependence graph.*

> Cytron et al. (1991): Copies in predecessor basic blocks.

Incorrect!
* Bad understanding of parallel copies
* Bad understanding of critical edges and interference

> Briggs et al. (1998)

Both problems identified. General correctness unclear.

> Sreedhar et al. (1999)

Correct but:
* handling of complex branching instructions unclear
* interplay with coalescing unclear
* "virtualization" hard to implement

Many SSA optimizations turned off in gcc and Jikes.

---

TBD. Some papers in the "Construction Algorithms" section also include
information/algorithms on deconstruction.


Control Flow Analysis
=====================

According to 1997 Muchnick:

* Analysis on "raw" graphs, using dominators, the iterative dataflow algorithms
* Interval Analysis, which then allows to use adhoc optimized dataflow analysis.
  Variants in the order of advanceness:
  * The simplest form is T1-T2 reduction
  * Maximal intervals analysis
  * Minimal intervals analysis
  * Structural analysis


Alias Analysis
==============

* 1994 [Program Analysis and Specialization for the C Programming Language](http://www.cs.cornell.edu/courses/cs711/2005fa/papers/andersen-thesis94.pdf),
  Lars Andersen


Register Allocation
===================

Wikipedia: https://en.wikipedia.org/wiki/Register_allocation

Terms:

* Decoupled allocator - In classic register allocation algorithms, variables
  assignment to registers and spilling of non-assignable variables are
  tightly-coupled, interleaving phases of the single algorithm. In a decoupled
  allocator, these phases are well separated, with spilling algorithm first
  selecting and rewriting spilling variables, and assignment algorithm then
  dealing with the remaining variables. Most of decoupled register allocators
  are SSA-based, though recent developments also include decoupled allocators
  for standard imperative programs.
* Chordal graph - A type of graph, having a property that it can be colored
  in polynomial time (whereas generic graphs require NP time for coloring).
  Interference graphs of SSA programs are chordal. (Note that arbitrary
  pre-coloring and/or register aliasing support for chordal graphs, as
  required for real-world register allocation, may push complexity back into
  NP territory).

Conventional Register Allocation
--------------------------------

TBD

Register Allocation on SSA Form
-------------------------------

* [University of Saarland page on SSA register allocation](http://compilers.cs.uni-saarland.de/projects/ssara/)
  gives a good overview of Register Allocation area, and how SSA form makes
  some matters easier.


Projects
========

Academic projects
-----------------

* [SUIF1](https://suif.stanford.edu/suif/suif1/index.html) - 1994, Stanford University
  * "The SUIF (Stanford University Intermediate Format) 1.x compiler,
    developed by the Stanford Compiler Group, is a free infrastructure
    designed to support collaborative research in optimizing and
    parallelizing compilers."
* [SUIF2](https://suif.stanford.edu/suif/suif2/index.html) - 1999, Stanford University
  * "A new version of the SUIF compiler system, a free infrastructure
    designed to support collaborative research in optimizing and
    parallelizing compilers. It is currently in the beta test stage
    of development."
* Machine SUIF aka machsuif - "Fork" of SUIF1/SUIF2, Harvard University
  * https://web.archive.org/web/20090630022924/http://www.eecs.harvard.edu/machsuif/software/software.html (via archive.org)
  * http://www.cs.cmu.edu/afs/cs.cmu.edu/academic/class/15745-s02/public/doc/overview.html
* [NCI (National Compiler Infrastructure)](https://suif.stanford.edu/suif/NCI/) ([archive.org](https://web.archive.org/web/20090901021758/http://www.cs.virginia.edu/nci/)) - 1998-200x? Collaborative project among US universities
  * "the National Compiler Infrastructure project has two components:"
  * SUIF
  * [Zephyr](https://web.archive.org/web/20090310184351/http://www.cs.virginia.edu/zephyr/)
    * Zephyr ASDL now lives at http://asdl.sourceforge.net/
      * Zephyr ASDL description used in Python: https://github.com/python/cpython/blob/master/Parser/Python.asdl
      * Oilshell blog post dedicated to Zephyr ASDL: https://www.oilshell.org/blog/2016/12/11.html

---
This list is compiled and maintained by Paul Sokolovsky, and released under
[Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).
