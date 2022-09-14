---
layout: post
title: ACM LaTeX Template
---

# ACM LaTeX Template

## Resources
- [ACM](https://www.acm.org/)
- [acmart](https://github.com/borisveytsman/acmart)
- [Capitalize My Title](https://capitalizemytitle.com/)
- [CCS Concepts](https://dl.acm.org/ccs#)

## Introduction
The Association for Computing Machinery (ACM) is an umbrella organization for academic computer
science. It is the world's largest educational organization with around 100,000 members. ACM
sponsors many conferences including
[EC: Economics and Computation](https://dl.acm.org/conference/ec),
[ACM CCS Workshop on Decentralized Finance and Security](https://defi.security/) and
[The Web Conference (WWW)](https://www2022.thewebconf.org/).
ACM also publishes over 50 journals including
[Distributed Ledger Technologies: Research and Practice (DLT)](https://dl.acm.org/journal/dlt) and
[ACM Transactions on Economics and Computation](https://dl.acm.org/journal/teac).

[SIGecon](http://www.sigecom.org/) is an ACM special interest group that focuses on the
intersection of economics and computer science.

## LaTeX Template
Manuscripts submitted to any of the ACM journals or conferences must be formatted using the ACM
[LaTeX template](https://www.acm.org/publications/proceedings-template). The template alternates
the document such that it can more easily be organized in the "Digital Library" by metadata.
There is an extensive
[guide](https://www.acm.org/binaries/content/assets/publications/consolidated-tex-template/acmart.pdf)
for the LaTeX typesetting class for the ACM publications.

There is an
[approved list of accepted packages](https://www.acm.org/publications/taps/accepted-latex-packages)
to ensure full compatibility.

ACM has partnered with and has a version of their template on
[Overleaf](https://www.overleaf.com/gallery/tagged/acm-official#.WOuOk2e1taQ).

## Technicals
The acmart package is included in the most common LaTeX distributions such as TeXLive or MikTeX.

There are a number of documentclass-options such as reviewer mode, anonymous authors and double
versus single column format. A potentially useful option is "nonacm" which will use the class
typesetting options for a non-ACM document, which will not include the conference/journal
header and footers or permission statements.

## Reference formatting
- [ACM BibTeX Formatting](https://www.acm.org/publications/authors/bibtex-formatting)
- [Citation Style and Reference Formats](https://www.acm.org/publications/authors/reference-formatting)

The ACM template unfortunately uses natbib which is not compatible with BibLaTeX only with BibTeX.
However there is a solution to turn this off in the documentclass options by `natbib=false`.

## Add CCS Concepts to tex file
The ACM Computing Classification System helps readers find and search for your work. CCS Concepts
are added to your document by including the code from the CCS page in your tex file.
