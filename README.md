# ASketch: A Sketching Framework for Alloy

`ASketch` is a command line tool built on top of
[Alloy5.0](https://github.com/AlloyTools/org.alloytools.alloy).  Given
a partial Alloy model, a candidate fragment generator, and a set of
AUnit tests that capture the desired model properties, `ASketch` is
able to fill holes with the corresponding candidate fragments and
makes the completed model satisfy all AUnit tests.  Internally,
`ASketch` automatically encodes the sketching problem into a meta
Alloy model and invokes the Alloy analyzer to solve for solutions.

# Requirements:

* Operating Systems
  - Linux (64 bit)
  - Mac OS (64 bit)

* Dependencies
  - Java 8: Must be installed and accessible from `PATH`.
  - Bash 4.4: Must be installed and accessible from `PATH`.
  - Maven >3.5.2: Must be installed and accessible from `PATH`.
  - Alloy 5.0: Must be in the classpath.  `ASketch` comes with
    Alloy5.0 under `libs/alloy.jar`.

# Installation:

## Clone ASketch repo

To run `ASketch`, use `git` to clone the repository.

```Shell
git clone git@github.com:kaiyuanw/ASketch.git
```

## Build ASketch

To build `ASketch`, Java 8 and Maven 3.5.2 or above must be installed.
Then, you can run `./asketch.sh --build` in Bash 4.4 to build
`ASketch`.

# Quick Start:

## Sketch Partial Models

To sketch a partial Alloy model, run
```Shell
./asketch.sh --run -m <arg> -f <arg> -t <arg> [-s <arg>] [-n <arg>]
```
or use the full argument name
```Shell
./asketch.sh --run --model-path <arg> --fragment-path <arg> --test-path <arg> [--scope <arg>] [--sol-num <arg>]
```
 * `-m,--model-path`: This argument is required.  Pass the partial
   Alloy model with holes that you want to sketch as the argument.
 * `-f,--fragment-path`: This argument is required.  Pass the
   generator file which provides the candidate fragments to consider
   for each hole as the argument.
 * `-t,--test-path`: This argument is required.  Pass the test file
   which contains AUnit tests that capture the desired properties of
   the expected model as the argument.
 * `-s,--scope`: This argument is optional.  Pass the Alloy scope for
   solving the generated meta Alloy model.  The scope is typically
   larger than or equal to the minimum scope that is necessary to make
   all AUnit tests satisfiable.  If the argument is not specified, a
   default value of 3 is used.
 * `-n,--sol-num`: This argument is optional.  Pass the number of
   unique solutions to the sketching problem you want `ASketch` to
   report.  A solution is unique if at least one value of the hole
   reported by `ASketch` is different from any other solution.  If the
   argument is not specified, a default value of 1 is used.

For each run, the command reports: (1) the interpreted candidate
fragments for each hole provided by the generator; (2) the rank,
associated identifier, type and the size of the candidate space for
each hole; (3) the size of the entire search space computed by
multiplying the sizes of individual search spaces of all holes; and
(4) a set of solutions if any as well as the solving time for each
solution.

The generated meta Alloy model will be stored under the project hidden
directory at `${project_dir}/.hidden/solve.als` in case if you want to
extend `ASketch` and debug the tool.

## Included Examples

We provide 10 example subjects and the command to run them.  Each
subject comes with a partial Alloy model, a generator and a set of
AUnit tests.  The `experiments/models` directory contains all example
models.  The `experiments/fragments` directory contains all
generators.  The `experiments/tests` directory contains all AUnit
tests.  The example models are listed below:

 * `arr` models an array.
 * `bt` models a binary tree.
 * `cd` models a Java class diagram.
 * `contains` checks whether a list contains an element.
 * `ctree` models a two colored undirected tree.
 * `deadlock` models a process deadlock.
 * `dll` models a doubly-linked list.
 * `grade` models how teaching assistants grade assignments.
 * `remove` models removing an element from a list.
 * `sll`: Models a singly-linked list.

To sketch a given example model, run ```Shell ./asketch.sh
--run-example ${model} ``` where `${model}` can be one of `[arr, bt,
cd, contains, ctree, deadlock, dll, grade, remove, sll]`.  By default,
`ASketch` reads the model, the generator and the AUnit tests from
`experiments/models/${model}.als`,
`experiments/fragments/${model}.txt` and
`experiments/tests/${model}.als`, respectively.  The solution is
reported to the standard output.  The scope used varies for different
models and the default number of solutions to report is 1.  For more
details, take a look at `models.sh`.

To sketch all 10 example models, run
```Shell
./mualloy.sh --run-all
```

# Mutation Operators

`MuAlloy` supports the following mutation operators.

| Operator |                              Description                               |
|----------|------------------------------------------------------------------------|
|   MOR    | Multiplicity Operator Replacement.  E.g. `some sig S` => `lone sig S`  |
|   QOR    | Quantifier Operator Replacement.  E.g. `all s: S` => `some s: S`       |
|   UOR    | Unary Operator Replacement.  E.g. `some S` => `no S`                   |
|   BOR    | Binary Operator Replacement.  E.g. `a + b` => `a - b`                  |
|   LOR    | Formula List Operator Replacement.  E.g. `a && b` => `a \|\| b`        |
|   UOI    | Unary Operator Insertion.  E.g. `a.b` => `a.*b`                        |
|   UOD    | Unary Operator Deletion.  E.g. `a.^b` => `a.b`                         |
|   BOE    | Binary Operator Exchange.  E.g. `a - b` => `b - a`                     |
|   IEOE   | Imply-Else Operator Exchange.  E.g. `a => b else c` => `a => c else b` |

The full mutations are shown below:
 * MOR: `${x} sig S {...}` to `${y} sig S {...}` where `${x}`, `${y}`
   ∈ {`ε`, `lone`, `one`, `some`}.  `ε` means empty string and it
   represents `set` multiplicity in Alloy signature declaration by
   default.
 * QOR: `${x} s: S | ...` to `${y} s: S | ...` where `${x}`, `${y}` ∈
   {`all`, `no`, `lone`, `one`, `some`}.
 * UOR: `s: ${x} S` to `s: ${y} S` where `${x}`, `${y}` ∈ {`set`,
   `lone`, `one`, `some`}; `${x} S` to `${y} S` where `${x}`, `${y}` ∈
   {`no`, `lone`, `one`, `some`}; `a.${x}b` to `a.${y}b` where `${x}`,
   `${y}` ∈ {`*`, `^`}.
 * BOR: `a ${x} b` to `a ${y} b` where `${x}`, `${y}` ∈ {`+`, `&`,
   `-`} or {`=`, `!=`, `in`, `!in`} or {`=>`, `<=>`}.
 * LOR: `a ${x} b ${x} ...` to `a ${y} b ${y} ...` where `${x}`,
   `${y}` ∈ {`&&`, `||`}.
 * UOI: `a` to `${x}a` where `${x}` ∈ {`*`, `^`, `~`}.
 * UOD: `${x}a` to `a` where `${x}` ∈ {`*`, `^`, `~`} or {`!`}.
 * BOE: `a ${x} b` to `b ${x} a` where `${x}` ∈ {`-`} or {`in`, `!in`,
   `=>`}.
 * IEOE: `a => b else c` to `a => c else b`.

When the above mutation operators involve both `${x}` and `${y}`, they
cannot be the same relational operator.

# Background

## Alloy Model

We show an [acyclic singly linked
list](experiments/models/singlyLinkedList.als) Alloy model below:
```Alloy
module SinglyLinkedList
sig List {
  header: lone Node
}
sig Node {
  link: lone Node
}
pred Acyclic (l: List) {
  no l.header or some n: l.header.*link | no n.link 
}
run Acyclic
```

The model declares a set of `List` and `Node` atoms.  Each `List` atom
has zero or one `header` of type `Node`.  Each `Node` atom has zero or
one following `Node` along `link`.  `header` and `link` are partial
functions.  The predicate `Acyclic` restricts its parameter `l` to be
acyclic.  The body of the `Acyclic` predicate states that `l` is
acyclic if (1) it does not have an `header` or (2) there exists some
`Node` reachable from `l`'s `header` following zero or more `link`,
such that the `Node` does not have a subsequent node along the `link`.

## Alloy Instance

Below is an satisfiable Alloy instance for the above list model if we
run the `Acyclic` predicate:

![List Instance](../documentation/documentation/images/ListInstance.png)

The instance states that there are two `List` atoms (`List0` and
`List1`) and two `Node` atoms (`Node0` and `Node1`).  `List0`'s header
is `Node1` and `List1`'s header is `Node0`.  `Node1`'s next node is
`Node0`.  Assuming `List0` is implicitly passed as the argument of
`Acyclic` predicate, we can see that `List0` is indeed acyclic as
there is no loop in the list.

## AUnit Test

An `AUnit` test is a pair of a model valuation and a run command.  For
example, the above Alloy instance can be written as an `AUnit` test as
below:
```Alloy
pred test {
  some disj List0, List1: List {
    some disj Node0, Node1: Node {
      List = List0 + List1
      header = List0->Node1 + List1->Node0
      Node = Node0 + Node1
      link = Node1->Node0
      Acyclic[List0]
    }
  }
}
run test
```
The test declares 2 disjoint `List` atoms (`List0` and `List1`) and 2
disjoint `Node` atoms (`Node0` and `Node1`).  It restricts the entire
`List` set to be {`List0`, `List1`} and `Node` set to be {`Node0`,
`Node1`}.  The predicate also states that the `header` maps `List0` to
`Node1` and `List1` to `Node0`, and the `link` maps `Node1` to
`Node0`.  If you run the `test` predicate, you will obtain the
isomorphic Alloy instance shown [above](#alloy-instance).

## Killing Mutant

One of the mutant `MuAlloy` generates from the list example model
using [QOR](#mutation-operators) is shown below:
```Alloy
module SinglyLinkedList
sig List {
  header: lone Node
}
sig Node {
  link: lone Node
}
pred Acyclic (l: List) {
  no l.header or all n: l.header.*link | no n.link 
}
run Acyclic
```
`MuAlloy` mutates `some n: ...` to `all n: ...`, which restricts every
`Node` in `l` without a subsequent `Node` along the `link`.  This
overconstrains `l` so it can be empty or only have one `Node`.  The
above `AUnit` test will be unsatisfiable for `List0` as `Node1` has a
subsequent node `Node0`.  Since the `test` is satisfiable for the
original model but is unsatisfiable for the mutant, it kills the
mutant.  Similarly, if an `AUnit` test is unsatisfiable for the
original model but is satisfiable for the mutant, it also kills the
mutant.

# Limitation

`MuAlloy` currently does not support mutating AST nodes declared
inside signature facts because the equivalence checking is not
supported for now.  The workaround is to move signature fact to a
stand-alone fact declaration.  We may support mutating signature facts
in a future release.

# Publications
* "Towards a Test Automation Framework for Alloy."
    Allison Sullivan, Razieh Nokhbeh Zaeem, Sarfraz Khurshid, and Darko Marinov, SPIN 2014
* "MuAlloy : An automated mutation system for Alloy."
    Kaiyuan Wang, Master Thesis 2015
* "Automated Test Generation and Mutation Testing for Alloy."
    Allison Sullivan, Kaiyuan Wang, Razieh Nokhbeh Zaeem, and Sarfraz Khurshid, ICST 2017
* "MuAlloy: A Mutation Testing Framework for Alloy."
    Kaiyuan Wang, Allison Sullivan, and Sarfraz Khurshid, ICSE 2018

# License

MIT License, see `LICENSE` for more information.