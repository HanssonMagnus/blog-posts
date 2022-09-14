---
layout: post
title: How to Cite Software
---

# How to Cite Software

## Resources
- [BibLaTeX](https://ctan.org/pkg/biblatex?lang=en)
- [BibLaTeX docs](https://ftp.acc.umu.se/mirror/CTAN/macros/latex/contrib/biblatex/doc/biblatex.pdf)
- [BibTeX](https://ctan.org/pkg/bibtex?lang=en)
- [Github Citations](https://github.blog/2021-08-19-enhanced-support-citations-github/)
- [Journal of Open Source Software](https://joss.theoj.org/about)
- [Software Sustainability Institute: Manifesto](https://www.software.ac.uk/about/manifesto)
- [Software Sustainability Institute: How to cite and describe software](https://www.software.ac.uk/how-cite-software)
- [Wiki: BibTeX](https://en.wikipedia.org/wiki/BibTeX)

## Introduction
This blog post summarizes how to cite software used in research papers and how developers can set
recommendations for how the software should be cited.

I recommend reading
[Software Sustainability Institute's blog post](https://www.software.ac.uk/how-cite-software)
and [Smith et al (2016) "Software citation principles"](https://peerj.com/articles/cs-86/)
about citing software, they were very helpful.

The tradition and importance of citing other's work is noticeable in academia, at least when it
comes to journal articles. Vital open source software used to conduct the research is
necessarily not cited. However, fields have come further than others, a friend told me that the field
of mathematics has started to [classify papers by software](https://zbmath.org/software/).
Outside of
academia it's not obvious that people cite the work that they used in their projects. E.g., if a
company uses open source software to build a product it's not given that it credits the
developers of that software.

The importance of citation include:
- To give credit to the inventor.
- To tell the author how their work is used.
- To ensure reproducibility.
- To ensure credibility.
- To ensure independence and intellectual honesty.
- To draw further attention to the work that helped the project/research.

(Note that some proprietary software licenses require that the software is cited.)

## LaTeX
When using `LaTeX` it's possible to automatically generate references from a `.bib` file.
There are two main ways to generate references: BibTeX and BibLaTeX.
[BibLaTeX](https://github.com/plk/biblatex) is a modern reimplementation of the LaTeX reference
system with modern features and is in general recommended over BibTeX.

In BibTex the
`.bib` database supports the following types: Article, book, booklet, conference, inbook,
incollection, inproceedings, manual, masterthesis, misc, proceedings, techreport and unpublished.
However, in BibLaTeX this list is extended.

## How to cite software?
Since more and more research requires computer science tools it's vital to cite these tools.
This is, however, not always as
easy as it sounds. Should I cite every `library` I use? How should I cite a project with only a
`Github` page? Although not straight forward, if the work was of significant importance to your
project it should be cited in one way or another.
A [guide line](https://www.software.ac.uk/how-cite-software) is that
it depends on the focus of the research: "Did the software play an important role?" and "Did the
software bring anything novel to the research?"

The R project has an entry in [their FAQ](https://cran.r-project.org/doc/FAQ/R-FAQ.html#Citing-R)
on how to cite `R`,

```
To cite R in publications,
@Manual{,
  title        = {R: A Language and Environment for Statistical
                  Computing},
  author       = {{R Core Team}},
  organization = {R Foundation for Statistical Computing},
  address      = {Vienna, Austria},
  year         = YEAR,
  url          = {https://www.R-project.org}
}
where `YEAR` is the release year of the version of `R` used and can determined as `R.version$year`.
```

This `BibTeX` entry can also be found by the command
[citation()](https://github.com/wch/r-source/blob/trunk/src/library/utils/R/citation.R)
in `R`,
```
To cite R in publications use:

  R Core Team (2018). R: A language and environment for statistical
  computing. R Foundation for Statistical Computing, Vienna, Austria.
  URL https://www.R-project.org/.

A BibTeX entry for LaTeX users is

  @Manual{,
    title = {R: A Language and Environment for Statistical Computing},
    author = {{R Core Team}},
    organization = {R Foundation for Statistical Computing},
    address = {Vienna, Austria},
    year = {2018},
    url = {https://www.R-project.org/},
  }

We have invested a lot of time and effort in creating R, please cite it
when using it for data analysis. See also ‘citation("pkgname")’ for
citing R packages.
```

To get only the `BibTeX` output in `R`,
```
cite <- citation()
toBibtex(cite)
```

Similarly, recommended citation for packages in `R` can be found by `citation('package_name')`,
e.g., `citation('ggplot2')` outputs the file
[CITATION](https://github.com/tidyverse/ggplot2/blob/main/inst/CITATION),

```
 @Book{,
    author = {Hadley Wickham},
    title = {ggplot2: Elegant Graphics for Data Analysis},
    publisher = {Springer-Verlag New York},
    year = {2016},
    isbn = {978-3-319-24277-4},
    url = {https://ggplot2.tidyverse.org},
  }
```

Citation files for `R` packages are usually found under `inst/CITATION`. Github will link to these
citation files under the tab "Cite this repository" but it will not try to parse the file. Github
only parses `CITATION.cff` files.

Note that [BibLaTeX](https://ftp.acc.umu.se/mirror/CTAN/macros/latex/contrib/biblatex/doc/biblatex.pdf)
has an additional `type` called `software`: "The standard styles will treat this entry
type as an alias for @misc."

Furthermore, there is no standard specifying if the research paper itself should contain all the
information about the software needed to reproduce the results. This is circumstantial and depends on
where the research is published and if the code used to conduct the research is available. If the
paper itself should contain all information necessary to reproduce the results, specific
information such as, which version or git commit may be needed. Although,
[Smith et al (2016)](https://peerj.com/articles/cs-86/) do not recommend citing commits, since they
are not permanent (projects can migrate), which again highlights potential problems with citing
software.

## What can developers do?
By setting a recommended way of citing the software, developers encourage citation of their
work and makes it easy for people to cite it.

The
[Github docs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-citation-files)
have advice for how to add a `CITATION.cff` file to the repository to help users cite the software.
`CITATION.cff` (Citation File Format) is a human and machine readable text file, read more about it
[here](https://citation-file-format.github.io/). If a developer puts a `CITATION.cff` file in the
root of the default branch of a repo it's automatically linked in Github and information is rendered on the
repo's page. A `BibTeX` snippet is also made available that users can copy.


Following the `R` structure the SSI
[blog post](https://www.software.ac.uk/blog/2016-10-06-encouraging-citation-software-introducing-citation-files)
recommends developers to put a text file in the root directory called `CITATION` together with
information on how to cite the software.

If there are white papers `.bib` files (BibTeX for most compatibility) can be created.
The [Journal of Open Source Software](https://joss.theoj.org/about) (JOSS) is built so that
"after you've done the hard work of writing great
software, it shouldn't take weeks and months to write a paper about your work." This is also a good
way for white papers to get published.

## How to cite software with a white paper?
There are a few advantages and disadvantages of citing papers associated with software. The
advantages include the general advantages with citing a paper. The disadvantages (if only method of
citation), however, include:
The software may change and the used version of the software is not cited. This can be mitigated if the
code for the research project is public and include sufficiently specified requirement files or the
software is cited in addition to the paper.

Different journals have different style guides for how to cite software. These recommendations
ranges from not citing (having a footnote or in-text citation instead) to guidelines how the
citation should be done.

There is no category for white paper in the `BibTeX` standard. However one of the following types
would be suitable (fields in parenthesis):

- misc (author, title howpublished, month, year note): For use when nothing else fits.
- unpublished (author, title, note, month, year): A document having an author and title, but not
formally published.

Example using `misc`:
```
@misc{nakamoto2008,
title = {Bitcoin: A Peer-to-Peer Electronic Cash System},
howpublished = {White paper},
year = {2008},
url = {https://bitcoin.org/bitcoin.pdf},
author = {Satoshi Nakamoto}
}
```

However, when using BibLaTeX there is a suitable entry:

- report: A technical report, research report, or white paper published by a university or some
other institution. Use the type field to specify the type of report. The sponsoring
institution goes in the institution field.

With the fields:
- required: author, title, type, institution and year/date.
- optional: subtitle, titleaddon, language, number, version, note, location, month, isrn, eid,
chapter, pages, pagetotal, addendum, pubstate, doi, eprint, eprintclass, eprinttype, url, urldate

Note however that `intitution` is a required field of the `type` and if no such is suitable then
the `misc` `type` might still be the best option, as in the case with the Bitcoin white paper.

Note: The [blog post](https://www.software.ac.uk/how-cite-software) by the Software Sustainability
Institute emphasize that, if DOIs
([digital object identifier](https://en.wikipedia.org/wiki/Digital_object_identifier) exist, use
them. The post states that "the advantage of DOIs is that they separate the description of an output from
its location, allowing the output to move over time" and also mildly advice against using URLs.

## Side note: What are these "colored" papers?
[Grey literature](https://en.wikipedia.org/wiki/Grey_literature) is research produced outside of
academia and include the following (see links for more elaborate explanation):

- [Green paper](https://en.wikipedia.org/wiki/Green_paper): Report by government.
- [White paper](https://en.wikipedia.org/wiki/White_paper): Explains a complex issue.
- [Yellow paper](https://en.wikipedia.org/wiki/White_paper#Variants): Preprint.

I have also seen other "colors", e.g., the
[Ethereum Beige Paper](https://github.com/chronaeon/beigepaper/), explaining the Ethereum
[Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) with simpler syntax.

## Conclusion
We can all agree that it's of significant importance to cite work that is contributing to our
research. However, there is no standard way to cite software. Which method to use boils down to
what the request is from the author, where you are publishing and how you fulfill your research
ethic obligations. Thus, when citing software first look to what the recommendation is. If there
is no recommendation, then from my understanding there are the following advisable ways to cite
software:

- Citing as type `manual` (BibTeX) or `software` (BibLaTeX).
- Citing a white paper as type `misc` (BibTeX) or `report` (BibLaTeX).
- Citing the software in-text.
- Citing the software in a footnote.

Multiple citation methods can be used simultaneously, e.g., both a white paper and a
software could be cited.
