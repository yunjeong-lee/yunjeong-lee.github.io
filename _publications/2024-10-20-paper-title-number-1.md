---
title: "DSLs in Racket: You Want It How, Now?"
collection: publications
category: conferences
permalink: /publication/2009-10-01-paper-title-number-1
excerpt: 'This paper presents a taxonomy of language design intents and analysis of Racket-embedded domain specific languages.'
date: 2024-10-20
venue: 'Software Language Engineering'
#slidesurl: 'http://academicpages.github.io/files/slides1.pdf'
paperurl: 'https://dl.acm.org/doi/abs/10.1145/3687997.3695645'
#citation: 'Yunjeong Lee, Kiran Gopinathan, Ziyi Yang, Matthew Flatt, and Ilya Sergey. (2024). &quot;DSLs in Racket.&quot; <i>SLE</i>.'
---

Domain-Specific Languages (DSLs) are a popular way to simplify and streamline programmatic solutions of commonly occurring yet specialized tasks. While the design of frameworks for implementing DSLs has been a popular topic of study in the research community, significantly less attention has been given to studying how those frameworks end up being used by practitioners and assessing utility of their features for building DSLs "in the wild". In this paper, we conduct such a study focusing on a particular framework for DSL construction: the Racket programming language. We provide (a) a novel taxonomy of language design intents enabled by Racket-embedded DSLs, and (b) a classification of ways to utilize Racket's mechanisms that make the implementation of those intents possible. We substantiate our taxonomy with an analysis of 30 popular Racket-based DSLs, discussing how they make use of the available mechanisms and accordingly achieve their design intents. The taxonomy serves as a reusable measure that can help language designers to systematically develop, compare, and analyze DSLs in Racket as well as other frameworks.