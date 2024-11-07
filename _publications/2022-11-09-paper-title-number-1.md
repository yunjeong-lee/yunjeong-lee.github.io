---
title: "Improving IDE code inspections with tree automata"
collection: publications
category: conferences
permalink: /publications/2024/10/paper-title-number-2
excerpt: 'This paper is about improving IDE code inspections by suppressing bogus warnings through tree automata.'
date: 2022-11-09
venue: 'Foundations of Software Engineering ESEC/FSE'
paperurl: 'https://dl.acm.org/doi/abs/10.1145/3540250.3559081'
#citation: 'Yunjeong Lee, Kiran Gopinathan, Ziyi Yang, Matthew Flatt, and Ilya Sergey. (2024). &quot;Tree Automata&quot; <i>Software Language Engineering (SLE '24)</i>.'
---

Integrated development environments (IDEs) are equipped with code inspections to warn developers of malformed or incorrect code by analyzing the code's data flow. However, the data flow analysis performed by the IDE code inspections may issue warnings in code where warnings are irrelevant. Existing methods to prevent any bogus warnings are either too conservative --- suppressing the same type of warnings in the entire codebase --- or are not sustainable as they require adding a set of heuristics to the data flow analysis. We propose a programming-by-example (PBE) framework that synthesizes tree automata (TAs) to suppress bogus warnings in all the code containing the same patterns as the user-provided examples. Experiments with a prototype of the framework demonstrate that several real-world patterns heuristically suppressed in production IDE can be mechanically translated to a TA in a systematic way. Furthermore, we briefly discuss how TA-based solution can be useful in other IDE features such as code refactoring.