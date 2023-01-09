---
layout: post
title: Meta-tags in LaTeX documents (Web indexing for academic papers)
---

# Meta-tags in LaTeX documents (Web indexing for academic papers)

## Resources
- [Google Scholar: Inclusion Guidelines](https://scholar.google.com/intl/en/scholar/inclusion.html#indexing)
- [PDF Metadata in LaTeX Documents](https://www.karlrupp.net/2016/01/pdf-metadata-in-latex-documents/)

## Introduction
When compiling a LaTeX document you can fill out metadata in your code such that it gets embedded
in the pdf file. This makes Google Scholar and web indexing software able to easier index
your paper and categorize it. This helps promote your paper and make it more accessible.

## Some basics of Google Scholar
You need to follow these guidelines for Google Scholar to correctly index your paper:

- The full text of your paper is in a PDF file that ends with ".pdf",
- The title of the paper appears in a large font on top of the first page,
- The authors of the paper are listed right below the title on a separate line, and
- There's a bibliography section titled, e.g., "References" or "Bibliography" at the end.

Google usually index the paper within a few weeks.

Further,
Google Scholar indexes papers that are uploaded to websites "connected" to Google Scholar, such as
arXiv and SSRN. However,
if you have uploaded a paper to your own website and want Google Scholar to recognize it you need to include a link to a single
HTML page listing your articles (e.g., www.example.edu/~professor/publications.html) on your
Google Scholar publications page.

## Configuring meta-tags
The PDF format includes a field for metadata, such as author and title.
Compiling your paper to export bibliographic data in HTML `"<meta>"` tags, helps Google to
correctly index your paper and its metadata. This is useful for internet searches and proper
indexing. In LaTeX we can include metadata by using the `hyperref` package,

```
\usepackage{hyperref}
\hypersetup{
pdftitle={Your PDF title},
pdfsubject={Your PDF subject},
pdfauthor={Your PDF author},
pdfkeywords={Your, PDF, keywords}
}
```

## Display metadata
The terminal program `pdfinfo` displays the metadata of a pdf file. Here is the output of one of my
papers without any meta-tags,

```
Title:
Subject:
Keywords:
Author:
Creator:         LaTeX with hyperref
Producer:        pdfTeX-1.40.24
CreationDate:    Mon Jan  9 10:27:30 2023 CET
ModDate:         Mon Jan  9 10:27:30 2023 CET
Custom Metadata: yes
Metadata Stream: no
Tagged:          no
UserProperties:  no
Suspects:        no
Form:            none
JavaScript:      no
Pages:           54
Encrypted:       no
Page size:       595.276 x 841.89 pts (A4)
Page rot:        0
File size:       801878 bytes
Optimized:       no
PDF version:     1.5
```

Neither SSRN nor arXiv add meta-tags to the pdf file by default and you need to do this yourself.
Here is the same file with meta-tags included,

```
Title:           Arbitrage in Crypto Markets: An Analysis of Primary Ethereum Blockchain Data
Subject:         Finance working paper
Keywords:        Arbitrage, Price efficiency, Decentralized exchanges, Ethereum, Blockchain,
Decentralized finance, Automated market maker, Cryptocurrency
Author:          Magnus Hansson
Creator:         LaTeX with hyperref
Producer:        pdfTeX-1.40.24
CreationDate:    Mon Jan  9 11:40:21 2023 CET
ModDate:         Mon Jan  9 11:40:21 2023 CET
Custom Metadata: yes
Metadata Stream: no
Tagged:          no
UserProperties:  no
Suspects:        no
Form:            none
JavaScript:      no
Pages:           54
Encrypted:       no
Page size:       595.276 x 841.89 pts (A4)
Page rot:        0
File size:       803224 bytes
Optimized:       no
PDF version:     1.5
```
