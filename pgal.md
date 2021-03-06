---
title: Guarded Action Language : Parameters
keywords: gal
tags: [gal]
sidebar: home_sidebar
permalink: pgal.html
summary: GAL Parameters.
---

# Parametric modeling

One essential task when studying a system consists in analyzing and comparing variants of a system.

For instance, many classic models can be scaled up (e.g. the number of philosophers in a ring),
you might want to study the how buffer sizes affect your system, you might want to explore 
different configurations of your components (ring, star, chain topologies...).   

Model-checking as performed in ITS-tools is not __parametric__, in the sense that it can only 
check properties for a given value of a parameter. But we do support parametric models, 
where just changing one declaration is enough to produce a specific instance of a model.

To this end, we introduce **GAL parameters**, which  are prefixed by a **$** sign, and 
are __run-time constants__. In a specific model __instance__ they have a single
 value that cannot change as the state of the system evolves.
 
These parameters can thus be regarded as syntactic sugar, helping the user define his system
 more easily, but any parametric GAL model can be degeneralized by substituting constant
  values to parameters in the text.
   
This conversion is currently performed __before__ invoking the model-checker, inspect the models
 produced in the "work/" subfolder after using "Run As->ITS Model-check" on a parametric GAL 
 to see the results of this degeneralization. 

## Parameters with a single value

The first and most basic type of parameters are simply a symbolic alias for a constant.

For instance, you can declare <code>$N=3</code>, then use <code>$N</code> in integer expressions of your specification.

While there is no syntactic constraint, we typically use upper-case names for these constant parameters (e.g. **$N** rather than **$n**).

### Global parameters

Global parameters are declared at the top level of a GAL file, simply introducing the name of the parameter and its value.
The value is just a constant.

Examples :

Suppose you want to configure the size of the system as a parameter N. You can define 
<code>$N=3;</code>, then use $N to define initial values of variables, size of arrays, maximum value in a range type definition...
Updating this **$N** declaration will update all these dependent declarations to produce the model you need.

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/sample_15.gal.html %}
</code></pre></figure>


Another use case is simply as symbolic constants, improving readability. 
In this example, the state of an automaton was encoded onto an integer, using parameters helps read the code. 

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/sample_16.gal.html %}
</code></pre></figure>

### Type parameters

A GAL type declaration can optionally declare one or more _type parameters_. 
A type parameter essentially declares a symbolic name for an integer constant, that can then be used within the various instructions of the GAL system (including initializations, typedef, guards, statements...). 

Parameter names start with a $ sign to avoid any confusion with the variables of the system. 
Parameters are given a value directly after their declaration. 

Note that when instantiating a GAL or composite within the context of composite ITS, parameters can be given a value (different from the value given in the parameter declaration).
Thus type parameters can be used to simulate a constructor for the GAL.

Syntactically, type parameters are given as a parenthesized comma separated list, just after the name of the system. 
Only constant expressions may be used in parameter initializations.

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/sample-param.gal.html %}
</code></pre></figure>

This small example shows a GAL with two **type parameters**, and how to override the values for these parameters at instantiation.
The syntax for Composite type parameters and their instantiation are the same as for GAL.


## Parameters over a __range__ of values

The second type of parameter is a bit different; it takes its values from a predefined finite range.

For instance, you can declare <code>typedef indexes=0..3;</code>, then use a parameter <code>$p : indexes</code> 
that can take any of the values in this range. 

This is especially useful when using arrays, and to define similar behaviors in a compact way.

While there is no syntactic constraint, we typically use lower-case names for these parameters over a range (e.g. **$p** rather than **$P**).

### **typedef** range declaration

A GAL **typedef** declaration allows to define a symbolic name for a given range of integers from __min__ to __max__. 

These bounds are inclusive, with the constraint that __min &le; max__, so that there is at least one element in any range definition.

These type definitions are used when declaring transition parameters, or within a **for** loop. 
They allow to define a set of similar transitions in a compact and readable manner.

Syntactically, a type definition is introduced with the keyword <span class="galElement">typedef</span> followed by a unique name for this type, followed by the actual range specification in the form "= min..max;". 
min and max are integer expressions built from constant parameters and constants only.

Here is an example of a system with some range definitions:

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/paramtype.gal.html %}
</code></pre></figure>


### Transition parameters

Transitions can optionally define one or more parameters that allow a more concise and readable representation of a complex transition relation. 

Transition parameters iterate over a given range of integers, defined using a **typedef**. 

Semantically, each parameter can be replaced in the transition effects by each of the possible values in its range.
Thus a parametric transition compactly represents many alternative transitions, that differ by the substitution value used for their parameters.

Transition parameters have the transition body as scope, and thus do not name clash with parameters of other transitions.

Syntactically, transition parameters are declared in a parenthesized comma separated list just after the transition name, with a syntax reminiscent of arguments for a function or method. 

Each parameter is defined by giving its type followed by the parameter name which must start with a $ sign and cannot shadow another parameter already in scope.

In this small example, we use two parameters.
In such a case, the cross product of the ranges of the parameters is explored. 

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/param.gal.html %}
</code></pre></figure>

This example is equivalent to this version that does not use parameter definitions. :

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/param.inst.gal.html %}
</code></pre></figure>

Transition parameters can be seen as a way of expressing a (parametric) set of **alternative** behaviors.
If the transition label does not use parameters, any one of these variants can be non-deterministically chosen for firing.
This makes this mechanism a dual of the **for** loop, that offers (parametric) **sequential composition** of behaviors.

### For loop action

To ease modeling, GAL provide a constrained For loop iterative control structure. 
This mechanism is close to macro expansion, the loop is unrolled before analysis is performed.

A for loop defines a local parameter which has as scope the body of the loop.
Since the domain of the parameter is is both known and finite, the loop can be simply unrolled.

The syntax is reminiscent of Java foreach loop, <code>for ($forparam : paramType) { body; }</code>. The body is an arbitrary sequence of statements.

This example shows a use of a for loop to set values in an array.

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/for.gal.html %}
</code></pre></figure>

It is strictly equivalent to this version.

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/for.inst.gal.html %}
</code></pre></figure>

### Parameters in Labels

Parameters can be used in labels, helping to define a finite set of labels sharing some characteristics.
Labels can define a set of __arguments__ which must have a fixed value at runtime (i.e. the set of labels does not evolve with the state).

The values for these arguments can be given or computed using the transition parameters when declaring a label.
Similarly, when calling a label all arguments must have a fixed value.

This example shows a (lossy) buffer containing a data.
For external synchronization the buffer exports a set of labels, one pair __send(d),receive(d)__,for each data __d__ it can contain.

We also declare a bounded counter, which can be incremented by calling it's __inc(d)__ label.
The value passed to the label controls how much the counter is incremented. 

But if the value passed to this buggy counter is __inc(1)__, an alternative available is to **not** increment the counter.
This __inc2__ transition shows how to define label parameters even if the transition itself is not parametric.

The overall composite just assembles these pieces into a scenario, where a counter is used to
 both count and limit the number of sent messages in the buffer. 
 
Two more counters are used to compute respectively how many messages were received, and the sum of their contents. 

This scenario is not particularly meaningful, but it is chosen to exhibit all the syntax related
to both declaring parameters on labels and calling such labels. 
 
<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/sample_17.gal.html %}
</code></pre></figure>

### Parameters in Composite

Within composite types, there are no variables, only nested instances.
Parameters can however be used to define parametric synchronizations, to iterate on arrays of instances using a for loop, 
and to define complex synchronization rules in combining **if-then-else** (where the condition is over parameters) 
and **abort** statements.

These control structures **for,if-then-else** and **abort** statements are the [same as in GAL](galbasics.html#if-then-else-action), 
within the limits of not having variables in scope.

This means all their condition reduces to __true__ or __false__ after parameter instantiation,
 hence this is again syntactic sugar degeneralized away prior to model-checking. 
 Look in the "work/" subfolder created after you execute "Run As-> ITS model-check" to see the target model. 

This example shows some use of these features to model some classic synchronization patterns :
* __start__ : A __choose one from a set__ with event that gives the token to a single __random__ participant
* __passToken__ : A __circular synchronization__ using a modulo to chain participants in a logical ring
* __reset__ : A __broadcast__ synchronization, that all the participants receive
* __passFast__ : A more complex synchronization, featuring a condition and an abort. The condition modeled here is completely arbitrary,
 it gives fast channels from participant at index __0__ to all other participants and allows some participants to skip their immediate successor. 
 The **abort** avoids creating a self loop with no effect on all states, otherwise __passFast__ instantiated for any __($i,$j)__ that do not meet
 the condition would produce an empty synchronization. 

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/sample_18.gal.html %}
</code></pre></figure>



## Parameter Instantiation

This section gives an overview of the algorithms used when instantiating parameters.
Most end-users do not need to be fully aware of the mechanics, so feel free to skip this section, 
provided mostly for documentation.

The main points relevant for the end-user are :
* During parameter instantiation, we also do a number of simplifications, that may remove variables (if they are constant), may remove transitions (if they are labelled and not called), 
may remove statements (if they have no tangible effect) and may remove type declarations (if they are not used in the __main__ instance)
* Some identical behaviors can be fused if they are redundant
* Transitions and synchronizations with several parameters may be translated to a more compact form by decomposing the transition

### Parameter Instantiation

Parameter instantiation consists in applying the following steps, in this order:

1.  All type level parameters (symbolic constants) are replaced by their value in all expressions.
2.  "For" loops are unrolled, i.e. the statement is replaced by n occurrences of the body statement, where the loop variable is replaced by its value in each of these occurrences. The unrolling of loops is done first on nested statements.
3.  Transition parameters are then instantiated. Each transition that has parameters can produce several transitions. Suppose transition has parameters $p1,..$pk in domains D1,..DK (see typedef keyword). 
We first instantiate $p1 giving |D1| transitions, in which $p1 is substituted by one of the values in D1\. More precisely, any occurrence of $p1 is replaced by its numeric value, in the guard, label, and in the transition body (where the parameter $p1 may occur in calls to a label). Each occurrence of the transition has a new name allowing traceability. On the fly, the guard of the newly created transitions is simplified, and no transition is produced if it happens to be false. This can skip construction of many transitions in some examples. Because we do this iteratively and test after each parameter instantiation if the transition is false, the number of "skipped transitions" reported in the verbose trace can thus be smaller than the difference between the Cartesian product of domain sizes and the number of transitions actually produced.

We then apply simplifications, currently we do the following rather trivial simplifications:

1.  Arithmetic and Boolean simplifications, on all constant expressions. Thus we replace (2 + 2) by (4), we also apply basic absorbing (0 * x -> 0, false && x -> false, true &#124;&#124; x -> true) and neutral element (1 * x -> x, false &#124;&#124; x -> x, true && x-> x) simplifications.
2.  We replace calls to non existing labels by an abort statement. These can occur as a result of other transition simplifications.
3.  We replace any sequence of statements containing an abort by a single abort statement. We destroy any transition with abort as body.
4.  Transitions with a false guard are destroyed.
5.  __if (c) { s1 } else { s2 }__ is replaced by __s1__ if __c__ can be reduced to true or to __s2__ if __c__ can be reduced to false.
6.  Constant variables are identified and simplified away. To do this, we first track all write accesses to variables or arrays. Initially, all variables and array cells are supposed constant. We then scan all write access (assignments) in the specification. If a variable is assigned to, it isn't a constant. A write access to a single cell of an array (tab[3]) means this cell is not constant. A write access to an array using an index expression (tab[i]) discards the whole array. Variables that are actually constant can then be replaced by their intial values in the whole specification. Simple constant variables and arrays that are entirely constant are then removed from the state signature, i.e. they are totally discarded. Because of the fact some variables disappear, this can lead to some confusion from the user, please look closely at the traces to understand what was simplified away.
7.  If we can find within a sequence of statements increments and/or decrements of a variable, and that they are commutative with statements that separate them, we fuse their effects. If the total effect is to leave the variable unchanged, the idle statement x=x+0 is removed. This can occur for instance with test-arc behavior in Petri nets.
8.  If two transitions that bear a label called only once and are identical in effects up to renaming of self, label and parameters, they are fused and only one of them is kept. This makes the specification slightly more compact, but is redundant with simplifications that occur in the model-checking engine. This reduction is called "fusion of isomorphic effects".

### Parameter Separation

Parameter separation consists in rewriting conjunction of choices (as expressed by transitions with several parameters) to sequence of choices where possible.

De-generalizing parametric transitions that bear a large number of parameters can produce very large GAL specifications. 
In general, degeneralization produces as many transitions as the size of the cartesian product of the domains of the formal parameters. 
But in many case, the full combinatorial unfolding can be avoided, allowing a compact transition representation while preserving semantics.

A transition has two independent parameters if it contains no statement that uses both parameters. 
Such a transition can be split into two labelled sub-transitions, one for each parameter, the semantics
 being preserved by calling the labels of these two sub-transitions. Statements are moved to the
  appropriate sub-transitions depending on the parameter they rely on. 
  
We thus obtain three transitions: the modified original one, that calls the sub-transitions and has no more parameter, 
and the sub-transitions, each of which has a single parameter. 
If the domain sizes of both parameters are called __r1__ and __r2__,
 we obtain after the instantiation of the parameters __r1 + r2 + 1__ transitions instead of __r1 * r_2__.
 Thanks to the sequence and call statements, the semantics is preserved.

More generally, the detection of the parameters to be separated in a given transition is done by building an hypergraph,
 whose nodes are parameters and where an hyperedge is added for each statement, connecting together all the parameters used in it. 
 We then consider the connectivity graph between parameters induced by this hypergraph. 
 
 If a parameter __$p__ has no neighbor, it can be separated:
  a new transition with parameter __$p__ and a new label __"lp"__ is built, whose body is the sequence of statements that depend on __$p__.

These statements are replaced in the original transition by a call the label __"lp"__. 

Similarly, a parameter __$p1__ that has a single neighbor __$p2__ can also be separated. 
We similarly build a transition with the statements depending upon __$p1__ alone or on both __$p1__ and __$p2__. 
This transition bears both parameters __$p1__ and __$p2__, and is labelled with a new label __"lp2"__. 
Parameter __$p1__ can then be removed from the original transition, and the relevant statements be
 replaced by a call to label __"lp2"__. 
 The procedure can be iterated on the simplified original transition, where __$p2__ may now be separable too. 
 Instantiation will also instantiate __$p2__ in __"lp2"__, so as to ensure consistency for the values of __$p2__ across the separated transitions.
 

This procedure has low complexity, depending on the size of the parametric GAL. 
It is applied before instantiation of the parameters. 
Relying on the distributivity between sequential and parallel composition, it helps producing transition relation
 expressed as sequences of parallel compositions, yielding much more compact representations
  than an expanded parallel composition of sequences (such as used by LTSmin). 
  
  This factored representation would not be efficient without the ability of symbolic operations to deal with parallel composition natively. 
  One may see this transform as a manner of delaying the computation of the combinatorial number of instantiations until
   symbolic evaluation time, where the symbolic data structures help with the combinatorics.

This example transition of a colored Petri net taken from [this VendingMachine example](http://mcc.lip6.fr/pdf/DrinkVendingMachine-form.pdf) of the 
[Model checking contest at Petri nets](http://mcc.lip6.fr) shows how this reduction works in practice.

This is an extract of the [full model](galfiles/drink-vending-2-col.gal). Before separation, we have many independent parameters. In fact here all parameters are independent, since no statement simultaneously uses two parameters. Furthermore parameters $o1 $o2 and $o3 play a very symmetric role.

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/vendingsimple.gal.html %}
</code></pre></figure>

After separation (and fusion of isomorphic effects on $o1 $o2 and $o3) we obtain the following model :

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/vendingsimple.sep.gal.html %}
</code></pre></figure>

This model when instantiated is still compact as shown here. Compare to what a [plain instantiation ](galfiles/vendingsimple.inst.gal) obtains;
 obviously when applicable this approach helps to scale as it can avoid polynomial (with degree the number of independent parameters) blowups in specification size.

<figure class="highlight"><pre><code class="language-c" data-lang="c">
{% include_relative galfiles/vendingsimple.flat.gal.html %}
</code></pre></figure>

