# Selecting a Modern Type System

Sam Lazarus, Northeastern University

**Abstract**

Type systems have always been a fundamental part of programming language
design, however until recently, algebraic type systems were out of reach
for practical languages used in industry. As modern programming
languages mature, many today have algebraic equivalent type systems that
employ Hindley Milner type inference. With complex type systems becoming
more commonplace, many groups are looking for ways to constrain types
beyond what is possible in a standard algebraic type system. This paper
reviews the approaches being considered for refining types further than
what is possible with a standard algebraic type system. Additionally, we
recommend a strategy for deciding which refinement typing system best
suits your needs based on the requirements for your project.

## Introduction

Modern programming languages have many features to check the validity of
a program before it is ever run. These diagnostics, known as static
guarantees, which are generated through the process of static analysis,
allow programmers to find many bugs that may otherwise slip through into
actual programs. Type systems are among the most widely adopted
strategies for static analysis as they allow the programmer to make
guarantees about the inputs and outputs of their functions which can
prove that their programs are structured in reasonable ways. Adding
additional specificity to type systems allows for the development of
more stable, and secure applications as programmers can enforce more
guarantees about their programs before they're even run.

As languages evolve, so to do their types systems. Until recently, the
majority of programming languages had type systems only powerful enough
to represent algebraic data types, or ADTs. Many of these systems are
built using the widely accepted algorithm Hindley Milner type deduction
(HM). New advances in static analysis however have allowed for types to
become even more specific. These types with enhanced specificity are
known as refinement types. In this paper, we review the techniques being
considered for implementing refinement types and compare their
advantages and disadvantages to each other, and existing HM based type
systems.

## Approaches

There are a variety of approaches to allowing refinement types in modern
programming languages. Here, we outline the approaches being explored
today, and highlight some of their strengths and weaknesses.

### Liquid Types:

Liquid types are an extension of standard HM typing which allows the
specification of refinements through the creation of subtypes of
existing types. Liquid is an abbreviation of *Logically qualified data*,
as it takes existing data types, and adds additional *qualifications* to
refine them. Liquid types differentiate themselves from other approaches
by making each of these subtypes abstract. To perform liquid type
analysis, first, the standard HM algorithm is applied to deduce a
concrete typing solution, and only then are liquid subtypes deduced. The
abstraction of refinement types until concrete types are solved allows
for liquid types to be implemented as a pure extension of existing
languages with HM type systems [2]. Such implementations exist for
many widely used programming languages. The most famous of these
implementations is Liquid Haskell, which can be found at
<https://ucsd-progsys.github.io/liquidhaskell-blog/about.html>.
Abstracting constraint solving provides two main advantages: it allows
for more specific refinement types than most other systems, and because
it is a process applied after HM deduction, it can be easily added onto
existing HM type systems. Liquid type systems also have associated
disadvantages. Due to the complex constraint system introduced on top of
HM, they tend to have slow compilation times [6]. Additionally, you
lose the powerful type inference associated with normal HM typing. In
some cases, this means producing cryptic error messages when types fail
to infer and in other cases it means types must be thoroughly specified
throughout your program or compilation is impossible. This adds an
additional burden to converting existing programs for use with liquid
types.

It is worth noting that liquid type systems *are not complete*. There
are cases where a program is valid, but the type system is unable to
prove its type soundness. These cases are rare but do exist and have
often been cited as the strongest argument against liquid type systems.

### Gradual Liquid Type Inference (GuiLT):

GuiLT is a modification of liquid types which borrows ideas from gradual
type systems in its implementation of refinement types. A gradual type
system is one in which types can be assigned during static analysis, or
they can be marked as untyped in which case they will be checked at
runtime [3]. Similarly, in a GuiLT system, some refinement types which
cannot be easily proven can be marked by the type checker as unrefined.
in this case they will not be processed by the system until after all
types have been assigned. The only requirement placed on these unrefined
types is that there exists some refinement type which can replace the
unrefined type which allows the program to type check. GuiLT then has
another constraint solving phase in which it attempts to prove that the
concrete program being processed constitutes an instantiation of a valid
replacement [4].

Because of its ability to defer refinement type deduction, GuiLT allows
for type inference in many places where liquid type systems would
require explicit type information. Additionally, GuiLT is able to
produce more informative error messages than inferred liquid types as in
the case that it fails to solve the constraint system, it can use
candidate concretizations to provide useful suggestions to the user
[4]. GuiLT also has one notable disadvantage: the GuiLT type checking
algorithm is slow even in comparison to already slow liquid type
systems. GuiLT checks if a concretization of a type is valid by
generating all possible concretizations of a type [4]. For simple
types, this is not a complex operation, but the complexity of generating
candidates scales exponentially with the number of unrefined types that
need to be deduced.

### HMF:

HMF is another extension on top of HM which adds first class
polymorphism and System F types. Like Liquid types, this approach allows
for the refinement of existing types through the creation of subtypes,
however it differs in its implementation approach. Rather than being
applied on top of HM, HMF is a modification to the HM algorithm which
deduces polymorphic constraints and solves refinement type systems
during deduction [5]. Because of this, HMF has many of the benefits of
liquid Haskell without some of the downsides. While it still suffers
from lengthy compile times, it manages to maintain the perfect type
inference of standard HM. In HMF type systems, types only need to be
specified when polymorphic types are desired. Since standard HM type
systems do not allow polymorphic types, all programs written for an HM
type system are valid in an HMF type system.

Unlike liquid type systems, making use of a refined type does not
require explicit type signatures, [5]. This advantage is also found in
GuiLT, however HFM type checks significantly faster than the glacially
slow GuiLT. Additionally, the improved type inference means no work
needs to be done to convert an existing HM program to its HMF
equivalent. Despite all of its advantages, HMF is still a (relatively)
new concept, and due to its lack of maturity has only been implemented
in two widely used languages (Haskell and OCaml) while liquid types have
been implemented in a plethora of existing languages.

### Fusion:

Fusion, like HMF, is a modification to the HM algorithm. Unlike all the
other strategies mentioned prior, Fusion doesn't abstract away
refinement type information during initial type checking. Other
algorithms shy away from introducing refinement type information before
the type system is partially solved to avoid the associated exponential
blowup in complexity. Fusion avoids this problem in a different way.
Instead of trying to get rid of the possibility of exponential blowup
completely, it uses scoping rules, and locality of types within modules
to limit the possibility of that exponential blowup to extremely
uncommon cases. What makes Fusion stand out is that in cases where
exponential blowup does not occur, type checking tends to be much faster
than in liquid type systems. This is because inferences can be made
before concrete types are fully deduced which can be used to speed up
the rest of the type checking process. In practice, this led to Cosman
and Jhala's implementation of Fusion Haskell to type check the Liquid
Haskell benchmarks over twice as fast as Liquid Haskell. In some
specific test cases, it even performed upwards of ten times as well as
Liquid Haskell. In addition, unlike every other algorithm considered,
Fusion is complete, meaning it can successfully prove the soundness of
any valid program [6]. Unfortunately, Fusion has no publicly available
implementations.

### Abstract Refinement Types (ART)

ART allows programs to specify abstractions over their type refinements
which adds expressive power, and then ability for unique constraint
solving techniques. It does so without sacrificing any speed over a
standard type system. The power of ART systems is on level with theorem
provers such as Coq, or Agda [7]. This expressive power comes at the
cost of a loss in type inference. Because types can be so specific, in
an ART system, type inference is very rarely possible. This is not
considered much of a weakness though, as ART systems are so powerful,
they're only needed when attempting to prove small theorems in the type
system. In such specific cases, it's an acceptable burden to forego type
inference, however most users would reach for another tool before ART
systems.

### Picking the Right Tool

Among the approaches being developed, by far the most promising and
applicable solution is Fusion. Fusion provides type check times better
than both liquid type systems, and HMF while being a complete type
algebra, a quality which none of the other approaches aside from ART
provide. Unfortunately, there is no publicly available implementation of
Fusion, so knowing it is the clear winner gets us no closer to picking
the best practical approach.

In order to determine the best solution then, the tradeoffs of the other
four solutions needed to be compared. The following table illustrates
the notable differences between the approaches. The table is ordered by
relative type check speed from slowest to fastest.

| **Approach** | **Implementation**                                    | **Complete Algebra** | **Type Inference**                                                    |
|--------------|-------------------------------------------------------|----------------------|-----------------------------------------------------------------------|
| Fusion       | Haskell                                               | Yes                  | Yes                                                                   |
| HMF          | Haskell, OCaml                                        | No                   | Only required on module boundaries                                    |
| ART          | Haskell                                               | Yes                  | HM Equivalent                                                         |
| Liquid Types | Haskell, OCaml, Racket, TypeScript, SML, among others | No                   | Bad error messages on failure, sometimes unable to deduce valid types |
| GuiLT        | Haskell                                               | No                   | HM Equivalent                                                         |

To determine the best solution, we set forth the following criteria: a
good solution must have relatively fast compile times, be complete, and
support as much type inference as possible. This immediately rules out
every one of our possible solutions which indicates that trying to pick
one solution as the best is a futile effort. Instead, we concluded that
there are different categories of use cases, each of which we provide a
different recommended solution for. To find the best solution to your
problem, determine which of the following descriptions your application
meets.

### Safety Critical Applications

Safety critical applications are applications which cannot break after
they are deployed. Examples include microcontrollers in large machinery,
critical medical devices, and stock trading algorithms. When designing
safety critical programs, speed of development is of little concern. As
such, the most important criteria is the specificity of the type system.
For these applications we recommend ART. It's shortcomings mostly come
in the form of inconvenience to the programmer, but its near proof-level
type system allows for guaranteed safety in systems that need additional
proof of their correctness. Since reliability is the top priority for
safety critical applications, ART is the natural choice.

### Existing Haskell / OCaml Applications, New applications

For languages where it is possible to work in Haskell or OCaml (either
existing applications in those languages, or new applications where a
language can be chosen) we recommend HMF. This proved to be the most
difficult category to produce a reasonable recommendation for as it
involved weighing completely disparate qualities of each system against
each other. Ultimately, this judgement is a subjective one, but we
believe it will serve most cases well.

For non safety critical applications, the specificity of the type system
becomes a secondary concern to the ability to productively develop
software in a language employing the chosen type system. In order to
productively develop software, one needs to not be burdened by their
tools, either by having to wait too long, or write down too many things
which should be immediately clear to the programmer writing them. Based
on these qualifications, the two best candidates for this recommendation
stood out to be GuiLT and HMF as they both meet have reasonable type
inference systems. GuiLT provides by far the best type inference model
of the two, but it does so at the cost of type check performance which
may slow down development cycles. That said, wrangling with the type
checker due to poor inference also incurs a development slowdown. We
make our recommendation of HMF, then, based on two factors. First,
GuiLT's type check speed is by a large margin the slowest of any
considered strategy. Second, while GuiLT has a better type inference
model, HMF only requires that refinement types be specified at module
boundaries. In practice, this isn't much of a barrier to productivity.
Additionally, documentation of module type constraints is often
considered good practice, so it may have been written even if it wasn't
a requirement. Finally, because the types inside a module cannot pollute
the type system of modules that depend on it, it is trivial to add new
modules with refinement types to an existing code base without them.
This makes gradual migration of existing code to employ HMF, and writing
only critical parts of your codebase with refinement types possible. If
a restriction preventing migration of existing code to HMF existed,
GuiLT would likely have been recommended instead due to how important
this is to the development of both a codebase, and a language ecosystem.

### Applications in Languages with Liquid Type Support

For projects in languages other than Haskell and OCaml which have
support for liquid typing, we strongly recommend employing liquid type
refinements where possible. The ability to further strengthen static
guarantee is extremely powerful as a programmer and leads to more
robust, secure, and stable applications.

## Conclusion

In this paper, we reviewed the various existing implementations of
refinement types, and each of their advantages and disadvantages. We
provided a guide for those wanting to employ refinement types so they
can determine what the best refinement typing system is for them. We
additionally determined that GuiLT is not a suitable typing system for
most practical applications despite being quite interesting
theoretically. Of note among the existing implementations examined is
Fusion, which while it isn't currently available to the public will
likely become an important tool for programmers in the future as it
becomes more accessible. Given its allowance for specific types, speed,
and impressive type inference, it is likely to become prominent among
programming language enthusiasts, and possibly even popularly used
languages.

## Acknowledgements

First, I would like to thank Professor Tom Akbari for helping shape me
as a writer. His thoughtful critique on my past portfolio has helped
push me to craft literary works with the same thought and attention I
give to my work in Computer Science. I feel I've crafted some of my best
essays under his guidance. I also wish to thank Erick Dang, and Bendetto
Vittum for their discussion regarding this work, and their critique of
my goals. Their advice to widen the scope of this discussion lead to the
review in its present state. Additionally, I'd like to thank Dr. Olin
Shivers for his mentorship in the world of compilers and programming
languages. His teachings gave me the technical expertise necessary to
provide insightful commentary on the state of the field. I'd also like
to thank the entire Swift programming language community, particularly
the core team, for teaching me everything I know about modern type
systems and constraint solving. I'd also like to thank Michal Lucas, and
Daniel Rassaby for their consultation. Finally, I'd like to thank
Sridevi Dayanandan for her gracious review of this paper, and her
support in the rest of my life.

## References

1.  T. Freeman, F. Pfenning, "Refinement Types for ML," Carnegie Mellon
    University, Pittsburgh PA, Tech. Report, June, 1991

2.  P. M. Rondon, M. Kawaguchi, R Jhala, "Liquid Types," University of
    California, San Diego, Tech. Report, June, 2008

3.  Y. Miyazaki, T. Sekiyama, A. Igarashi: "Dynamic Type Inference for
    Gradual Hindley-Milney Typing" arXiv: 1810.12619v2 [cs.PL], Nov,
    2018

4.  N. Vazou, E. Tanter, D. V. Horn: "Gradual Liquid Type Inference"
    arXiv: 1807.02132v1 [cs.PL], July, 2018

5.  D. Leijen, "HMF: Simple type inference for first-class
    polymorphism,", Proceedings of the 13^th^ ACM SIGPLAN international
    conference on Functional programming, Sept., 2008. \[Online
    serial\]. Available:
    https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-2007-118.pdf.
    [Accessed April. 5, 2019]

6.  B. Cosman, R. Jhala: "Local Refinement Typing" arXiv: 1706.08007v1
    [cs.PL], June, 2017

7.  N. Vazou, P. M. Rondon, R. Jhala, "Abstract Refinement Types,"
    University of California, San Diego, Google, Tech. Report, March,
    2016l
