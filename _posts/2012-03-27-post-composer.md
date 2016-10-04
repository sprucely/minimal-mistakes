---
title: "Composer - a new software paradigm, or just spinning my wheels?"
excerpt: "I have a hobby project that &lt;Aggrandization&gt;could potentially revolutionize how software is  developed and even change how microprocessors are designed.&lt;/Aggrandization&gt;"
modified: 2012-03-27
header:
  teaser: 
tags: 
  - Composer
  - Cosa
  - synchronous-reactive
---

I have a hobby project that &lt;Aggrandization&gt;could potentially revolutionize how software is  developed and even change how microprocessors are designed.&lt;/Aggrandization&gt; Yeah, I know -  yet another attempt at finding the elusive silver bullet that will solve all our software development  woes. I'm trying not to have any illusions about this going anywhere. As it is, the time I am able  to put into it is sporadic at best. But it's fun, so I continue to poke at it.

### What is Composer?

Well, in order to answer that I must associate myself with someone who's blog postings often need to be taken with a grain of salt. If you take yourself too seriously, then he's not for you. [Project COSA](http://www.rebelscience.org/Cosas/COSA.htm) was conceived of by  [Louis Savain](http://www.rebelscience.org/)...

> <a name="Abstract">Abstract</a> COSA is a <a href="http://www.rebelscience.org/Cosas/COSA.htm#Reactive">reactive</a>,          signal-based software construction and execution environment.         A COSA program is inherently concurrent and ideally suited for         fine-grained parallel processing. The         two main goals of Project COSA is to enable the creation of bug-free         software applications and to improve software productivity by several orders of         magnitude. COSA is         based on the premise that unreliability is not an essential         characteristic of complex software systems. The primary reason that computer programs are         unreliable is the age-old practice of using the algorithm as the basis         of software construction. Switch to a synchronous, signal-based model         and the problem will disappear (see <a href="http://www.rebelscience.org/Cosas/Reliability.htm">The         Silver Bullet</a> page for more information on this topic).

The search that led me to COSA was facilitated by my frustration at the complexity of fixing a  small bug in the <a href="http://boo-lang.org/">Boo compiler</a>. I couldn't understand why  compilers go through all the trouble of building up an  <a href="http://en.wikipedia.org/wiki/Abstract_syntax_tree">AST (abstract syntax tree)</a> only  to discard all the information when finished. So I wanted to see if there was any work out there  on persisting and visually manipulating the AST. I found <a href="http://en.wikipedia.org/wiki/Executable_UML"> Executeable UML</a> which did not smell very appetizing to me. I also found <a href="http://coherence-lang.org/"> Coherence</a>, developed by <a href="http://alarmingdevelopment.org/">Jonathan Edwards</a>, which had some  interesting concepts. But when I came across COSA, it gripped me. Here was something different enough to  keep my attention (it throws algorithmic programming out the door), yet completely feasible. I won't try  to explain any further what COSA is, because Louis' postings do a much better job.

### Is it vaporware?

I guess you can call it that. After a couple spikes (aka false starts) implementing a bare-bones operating  system hosting a COSA virtual machine (and I did actually get the OS to execute a very simple COSA program),  I realized that the lack of a component graph editor would make it difficult to create all but the most simple tests.  So my plan is for Composer to be an application for creating and managing COSA components. At first I chose WPF due to familiarity and the availability of some graphing libraries. But, seeing the writing on the wall for the future of  WPF, I'm taking a hard long look at HTML5. There are some decent open-source javascript libraries for managing SVG diagrams (<a href="http://www.jointjs.com/">jointjs</a>) and building MVC applications  (<a href="http://emberjs.com/">emberjs</a>). However, I'm a little put off by the impasse over the Web SQL standard. IndexedDB seems way too limiting in how I could manage local data, but maybe it's just that I'm a little entrenched in the SQL mindset. 

Ultimately I envision a suite of software components. I'm sticking to a musical theme, because it seems  to fit nicely. Here is a list in order of complexity/ambition...

- Composer - for developing/verifying COSA components

  Execution would be very slow because structure of component graph data is optimized for loading the top-level view of a component hierarchy during development, not for efficient loading of individual cells according to fired signals.

- Conductor - a virtual machine for executing a COSA program loaded from a file

  There are many issues to be resolved in deciding how a COSA program is represented in a file. I can see starting out with a simple xml representation, but ultimately for performance, a binary format would need to be created.

  Because current processors are optimized for the algorithmic model, execution of a COSA program would not be very efficient. However, things can be done to optimize execution. It might be possible to pass execution to GPUs which could better handle the MIMD nature of a COSA program. Ultimately, to realize the full potential of the COSA model, processors would need to be modified to natively handle cell execution/signaling, perhaps even automatically splitting the processing of cells across many cores.

- ConcertOS - an operating system that provides low-level services in the form of specialized COSA components     

  I don't have anything else to say about this at the moment.

---
edit 4/6/2012:
I just came across a nice javascript library called <a href="http://neyric.github.com/wireit/index.html">WireIT</a> that basically handles most aspects of visually managing a graph and includes an editor that supports graph serialization through JSON. <a href="http://neyric.github.io/wireit/docs/index.html#examples">Here's some examples</a>.
