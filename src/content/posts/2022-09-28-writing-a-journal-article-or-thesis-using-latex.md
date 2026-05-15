---
title: 'Writing a journal article or thesis using LaTeX'
date: 2022-09-28
lastmod: 2022-11-15
tags: ['LaTeX', 'thesis']
categories: ['Uncategorised']
---
When I started a PhD one of my supervisors encouraged me to try using LaTeX, and despite it seeming daunting at first, I'm very glad she did! I don't think I can ever go back to Word now... I've written this guide on getting started with LaTeX to help you get over the initial hump, and to provide a thesis template that meets the University of Southampton's guidelines, which can easily be adapted for other applications.

* * *

To get started I suggest playing around with [Overleaf](<https://www.overleaf.com/>). There are some great templates on there that you can, _A._ get started with quickly, and _B._ take inspiration (and code) from to build your own documents. You can use my (relatively) basic template to get started, **[download it here](<https://lawrenceyule.com/wp-content/uploads/2022/11/Basic-template.zip> "Basic template")**.

##### LaTeX and Visual Code Studio

Personally I prefer not to work in the browser, and I found that the LaTeX workshop plugin within Visual Code Studio is a great way to work from the desktop. You need a TeX distribution to get started, of which there are [many choices](<https://tex.stackexchange.com/a/239204/267108>). I suggest using either TeX Live, which is updated yearly, or MiKTeX, which allows you to install packages on-the-fly, saving on download time. You can follow [this guide](<https://mathjiajia.github.io/vscode-and-latex/>) to get setup with TeX Live, or [this guide](<https://mjcb.io/blog/2020/01/23/visual-studio-code-with-latex/>) for MiKTeX. Both are functionally the same.

I also suggest [setting up a Github repository](<https://code.visualstudio.com/docs/sourcecontrol/github>) for you to backup your work too, which will also allow you to push/pull to/from Overleaf for easy collaboration. I use a few different VS Code extensions, including LaTeX Utilities, LaTeX Workshop, TexLab, LaTeX language support, Dictionary Completion, Code Spell Checker, Grammarly, and the One Monokai Theme.

![](/assets/img/wp-uploads/2022/09/rkBoLTOmHK.png)LaTeX in Visual Code Studio

Although the installation guide linked above provides a selection of recipes for your settings.json file, you might find that you need additional options. The most common of which is likely to be switching between the two most common backends for handling referencing, `bibtex` (older, faster, for use with `bibtex` and `natbib`) and `biber` (newer, slower, for use with `biblatex`). Most of the time you'll want to use `[latexmk](<https://ctan.org/pkg/latexmk/>)` to handle compilation, but I have recipes for `pdflatex`, `xelatex`, and `lualatex`, just in case. See an explanation of the differences between these [here](<https://tex.stackexchange.com/a/13601/267108>). 
    
    
        "latex-workshop.latex.recipes":  
            }, 
            {
                "name": "pdflatex",
                "tools": 
            }, 
            {
                "name": "pdflatex -> biber -> pdflatex*2",
                "tools": 
            },      
            {
                "name": "pdflatex -> bibtex -> pdflatex*2",
                "tools": 
            }, 
            {
                "name": "xelatex",
                "tools": 
            }, 
            {
                "name": "lualatex",
                "tools": 
            },
            {
                "name": "xelatex->biber->xelatex",
                "tools": 
            }, 
            {
                "name": "lualatex->biber->lualatex",
                "tools": 
            },
        ],
        "latex-workshop.latex.tools": , 
                "env": {} 
                }, 
            {
                "name": "pdflatex",
                "command": "pdflatex",
                "args": 
            },
            {
                "name": "bibtex",
                "command": "bibtex",
                "args": 
            },
            {
                "name": "biber",
                "command": "biber",
                "args": 
            },
            {
                "name": "xelatex",
                "command": "xelatex",
                "args": 
            },
            {
                "name": "lualatex",
                "command": "lualatex",
                "args": 
            },   
        ],

![](/assets/img/wp-uploads/2022/10/yyHRjlCKKE.png)LaTeX commands in VS Code

The next thing to consider before doing any writing is referencing. I recommend using [JabRef ](<https://www.jabref.org> "JabRef ")as a referencing manager, as it will allow you to easily generate a **.bib** file that can be read by LaTeX. You can also use Mendeley desktop (soon to be phased out for an online only version) or Zotero, but personally I find JabRef to be the best for my workflow. I also suggest creating a new library for each different topic that you're working on, as this helps to keep things organised.

![](/assets/img/wp-uploads/2022/09/m6kAXbz6N1.png)JabRef

Now you're ready to start writing something, but where do you start? Consider that during your PhD you are likely to write a number of conference papers or journal articles, in many cases the publisher will provide a template for you to follow, which means they're handing the presentation element, not you. For your thesis you have a little more control, but your university is also likely to have a template or at least some guidelines on how they'd like it presented. Keep this in mind before spending too much time getting things just right. Most of the time you can trust LaTeX to position things in an aesthetically pleasing way anyway.

I have created a class file that contains all the necessary packages and macros to produce a thesis that matches the University of Southampton's guidelines, but it can be easily adapted for other uses. I have also provided templates for the various sections (abstract, declaration of authorship, etc.), and some examples of equation/figure/table environments. [**Download it here**](<https://lawrenceyule.com/wp-content/uploads/2022/09/UoS-style-doc.zip>). 

It's worth noting that there's already a slightly more official UoS template available [here](<https://git.soton.ac.uk/uos-latex-group/uos-latex-template-instructions>) ([also available on Overleaf](<https://www.overleaf.com/latex/templates/southampton-phd-thesis/rwkmxqhzktrx>)), which has far better documentation than mine. We've taken slightly different routes to achieve essentially the same thing, although there are some subtle differences.

One last thing before I get into it, setup a seperate test document that you can use to play around with new packages you add, or the generation of tables etc. Sometimes LaTeX will throw you an error that's really hard to diagnose in a big document, so try things out in a simplified environment first. 

##### Packages

There are a number of packages which I think are essential for any style of document, so I'll briefly explain them here. A great beauty of LaTeX is that someone has almost always figured out a way around a problem you're having, so get used to doing a lot of Googling!

  * `[siunitx](<https://ctan.org/pkg/siunitx?lang=en>)`
    * For typesetting units this is basically essential, and very easy to use.
  * [`gensymb`](<https://ctan.org/pkg/gensymb>)
    * Simple commands for common symbols such as `\degree`, `\celsius`, `\perthousand`, `\micro` and `\ohm`.
  * [`geometry`](<https://ctan.org/pkg/geometry>)
    * For setting page sizes and margins.
  * [`tabu`](<https://ctan.org/pkg/tabu>), [`tabulary`](<https://ctan.org/pkg/tabulary>), [`tabularx`](<https://ctan.org/pkg/tabularx>), and [`booktabs`](<https://ctan.org/pkg/booktabs>)
    * For creating nice tables. You may not need to use tabu, tabulary, and tabularx (I mostly use tabulary) but it's worth reading about the benefits of all of them. 
  * [`adjustbox`](<https://ctan.org/pkg/adjustbox>)
    * For resizing tables and figures.
  * [`microtype`](<https://ctan.org/pkg/microtype?lang=en>)
    * Fixes a lot of kerning issues and protrusions.
  * [`acro`](<https://ctan.org/pkg/acro>)
    * To create lists of acronyms/abbreviations that you can refer to throughout the document. Remember to declare them each time you add a new one. Your library can be used across multiple documents easily. 
  * [`cleveref`](<https://ctan.org/pkg/cleveref>)
    * Refer to tables/figures/sections that you've labelled. Automatically adds the prefix (e.g. "Chapter") which can be globally altered (e.g. always caps, abbreviated).
  * [`fancyhdr`](<https://ctan.org/pkg/fancyhdr>)
    * Controls the layout of your headers and footers.
  * [`hyperref`](<https://ctan.org/pkg/hyperref>)
    * Controls links and bookmarks within the document. Should be loaded last in most cases.
  * `[caption](<https://ctan.org/pkg/caption>)` and [`subcaption`](<https://ctan.org/pkg/subcaption>)
    * For setting the style of captions for figures and sub-captions for sub-figures.
  * [`titlesec`](<https://ctan.org/pkg/titles>)
    * Control the style of titles.
  * [`setspace`](<https://ctan.org/pkg/setspace?lang=en>)
    * Set the line spacing for the document. I use `\onehalfspacing`.
  * [`xcolor`](<https://ctan.org/pkg/xcolor?lang=en>)
    * To set custom colours for links etc. Called on by `hyperref`.
  * `[xurl](<https://ctan.org/pkg/xurl?lang=en>)` and `[url](<https://ctan.org/pkg/url>)`
    * `xurl` allows URLs to break at alphanumerical characters, which stops them from extending into the margins, and `url` with the `\urlstyle{same}` command uses the main font for URLs in the bibliography rather than a monospaced font.

All of these packages (and more) are already included in my class file with comments to show where they are used. 

##### document structure

For a thesis that's likely to cover multiple chapters and topics it makes sense to split up your work into multiple .tex files. This will make it faster to compile while you're working on it (you can just include one .tex file at a time), and it makes it easy to copy a whole chapter into a new document for a conference/journal article. For figures use a seperate folder, to keep them separate from the selection of auxiliary files that LaTeX will generate. Start with a 'main' .tex file that contains your preamble, package commands, macros, etc. and then includes the various chapters as `\input{}` files.

Before you get into the bulk of the document you'll want to have your title page, abstract, contents, list of figures/tables, and definitions/abbreviations. You will need to set up two different page styles (using the `fancyhdr` package), one without headers/footers (plain) and one with (fancy):
    
    
    %set main page style
    \fancypagestyle{fancy}{
      \fancyhf{}
      \fancyhead{\textrm\thepage}
      \fancyhead{\nouppercase{\textit{\rightmark}}}
      \fancyhead{\nouppercase{\textit{\leftmark}}}
      \fancyhead{\textrm\thepage}
      \chead{}\lfoot{}\rfoot{}\cfoot{}
      \renewcommand{\headrulewidth}{0.5pt}
    }
    %define simple page style with page number only
    \fancypagestyle{plain}{
      \fancyhf{}
      \fancyhead{\thepage}
      \fancyhead{\thepage}
      \renewcommand{\headrulewidth}{0pt}
    }

You can switch page style using the `\pagestyle{plain}` command. You'll also want to control page numbering between roman and arabic, which you can do with the `\pagenumbering{roman}` command.

If you are using a two-sided style then you need to ensure that every chapter starts on a right-hand page, with at least one blank page between chapters. I'm sure there is a way to do this automatically, but for now I use the `\cleardoublepage` command and (sometimes multiple) `\afterpage{\blankpage}` commands at the end of each chapter.

You might find that LaTeX really likes to hyphenate words when it doesn't need to. Try a combination of these commands to surpress it:
    
    
    \pretolerance=5000
    \tolerance=9000
    \emergencystretch=0pt
    \hyphenpenalty=10000
    \exhyphenpenalty=100
    \righthyphenmin=4
    \lefthyphenmin=4
    \doublehyphendemerits=10000       % No consecutive line hyphens.
    \brokenpenalty=10000              % No broken words across columns/pages.
    \widowpenalty=9999                % Almost no widows at bottom of page.
    \clubpenalty=9999                 % Almost no orphans at top of page.
    \interfootnotelinepenalty=9999    % Almost never break footnotes.

##### Referencing

There are a few different bibliography systems available in LaTeX, and the intricacies of them can make things very confusing. For a detailed explanation of the differences between packages, see [this stack exchange post](<https://tex.stackexchange.com/questions/25701/bibtex-vs-biber-and-biblatex-vs-natbib>). The two most common systems are `bibtex` and `biblatex`. `Biblatex` is the more modern one, and what I suggest you use for a thesis, but unfortunately it isn't always accepted by journals because they have their own `bibtex` style. This isn't really an issue as they both read .bib files in the same way, and they both use the same `\cite{}` command, but it's something to be aware of. For `biblatex` you use:
    
    
    \usepackage{biblatex}
    \addbibresource{refs.bib}
    %%%%%%%%%%%%%%%%%%%%%%%%%
    \printbibliography
    

And for `bibtex` you use:
    
    
    \bibliographystyle{your_style_here}
    %%%%%%%%%%%%%%%%%%%%%%%%%%
    \bibliography{refs}

A great list of different styles is available [here](<http://debibify.dorian-depriester.fr>). It's also a good idea to use the [notoccite package](<https://ctan.org/pkg/notoccite> "notoccite package") which fixes the ordering of citations if they're used in section titles or captions. 

##### typeface

For a lot of people (me included) one of the draws of LaTeX is the typeface. Computer Modern has a certain look that says "this is LaTeX", which is both a positive and a negative. I've since come to love the [Libertinus font family](<https://github.com/alerque/libertinus>), which I think provides a cleaner look while also having a certain flair (just look at that Q!). Being freely available also means it's easy to use for figures, to keep things consistent. To use the font in LaTeX just add the package to your preamble: `\usepackage{libertinus}`.

##### Figures

This is a slight tangent from the writing of the document, but you'll want to pay special attention to figures, only using vector format graphics where possible. LaTeX likes to work with .eps files, which you can export to with MATLAB, R, Python, etc. You can then tweak these figures in Adobe Illustrator/Inkscape, or annotate a photo for example, and save to an .eps. In my case I went through multiple iterations of figure style, changing colour palettes, line thicknesses, font sizes, etc. before I was happy, which was only possible because I worked with .eps files from the start. I created the large majority of my figures with MATLAB, where the `export_fig` package came in very useful. Here's some code to control the axes colours, line width, plot size, font, and output format of your figures: 
    
    
    set(groot,{'DefaultAxesXColor','DefaultAxesYColor','DefaultAxesZColor'},{'k','k','k'})
    set(gca,'LineWidth',1.2)
    
    x0=10;
    y0=10;
    width=1200;
    height=1000;
    set(gcf,'position',)
    set(gca,'fontname','Libertinus Sans','fontsize', 20)
    set(gcf, 'Color', 'w');
    
    export_fig figure.eps -painters -nocrop

To include a figure in your document use the following code:
    
    
    \begin{figure}
        \centering
        \includegraphics{./figures/figure.eps}
        \caption{This is the caption}\label{fig:x}
    \end{figure}

Notice that there are two inputs to the `\caption` command, the first one `[]` controls what will appear in your list of figures, while the second `{}` controls what appears as the caption. This is useful if you don't want a citation or a copyright declaration to appear in your list of figures.

You may find that a figure ends up somewhere you're not expecting it to be, which is normally because of the `` command. Change it to `` (you _really_ want it there), or force it onto a new page with ``. Also check that it isn't too big for the page, and consider scaling it down.

##### Tables

I suggest using an [online LaTeX table generator](<https://www.tablesgenerator.com/>) (in booktabs style) to generate the code for a table from Excel data. This allows you to play around with the look and layout of the table before you try including it in your document. If you have particularly wide tables you can shrink them down to fit within the margins by placing the `tabular` environment within an `adjustbox` environment:
    
    
    \begin{table}
        \centering
        \begin{adjustbox}{width=1\textwidth,center=\textwidth}
        \begin{tabular}{ll}
            \toprule
            \textbf{Heading 1} & \textbf{Heading 2}  
            \midrule
            \bottomrule
        \end{tabular}
    \end{adjustbox}
    \caption{}\label{table:table1}
    \end{table}

##### equations

I like to include the units for my equations right-aligned within the equation environment. You can do this by including the follow in your preamble:
    
    
    \makeatletter
    \providecommand\add@text{}
    \newcommand\tagaddtext{%
      \gdef\add@text{#1\gdef\add@text{}}}% 
    \renewcommand\tagform@{%
      \maketag@@@{\llap{\add@text\quad}(\ignorespaces#1\unskip\@@italiccorr)}%
    }
    \makeatother

Then in the text:
    
    
    \begin{equation} \label{velocitycalc}
        v = \frac{d}{t_F}
        \tagaddtext{}
    \end{equation} 

![](/assets/img/wp-uploads/2022/09/dX9mLED9B3.png)

##### Common commands

There are a number of commands that you'll find yourself using regularly, once you understand what they do. Here are a list of a few of them:

  * `\noindent` \- Stops a sentence from being indented. 
  * `\cref{}` \- From the cleveref package, this allows you to refer back to anything that you've set a label for using `\label{}`. Try to preface your labels with fig:, tab:, chapter:, etc. to help you keep track of what is being labelled.
  * `\cite{}` \- To cite something from your .bib file. You should get an autocomplete prompt to help you find what you want. 
  * `\chapter{}`, `\section{}`, `\subsection{}`, `\subsubsection{}` \- fairly self explanatory, you can control how deep the numbering goes with `\setcounter{secnumdepth}{4}` in the preamble.
  * `\unit` and `\num `\- From the `siunitx` package, this is how you'll write units. E.g. `\unit{\meter\per\second}` = m s-1, and numbers e.g. `\num{0.3e45}` = 0.3 × 1045.
  * `\clearpage` and `\cleardoublepage` \- For when you want a page to end and for any floats (figures etc.) to be on the next page. The double page variant is only relevant to two-sided documents. 
  * `\vspace*{\fill}` \- Use this to create a space above a float when it's at the top of a page, as `\vfill` doesn't work in that case. 
  * `\afterpage{\blankpage}` \- To add a blank page using the `afterpage` package.
  * `~` \- A non-breaking space. Place these between numbers and units to stop them from being split apart at the end of lines.

##### shrinking pdfs

You might find that the pdf generated by LaTeX is quite large, which makes it difficult to share via email etc. This is generally because you have fonts embedded in your figures multiple times. To reduce the file size you can use [pdfsizeopt](<https://github.com/pts/pdfsizeopt>), or you can use a Ghostscript command at the command line. I use:
    
    
    gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.5 -dNOPAUSE -dQUIET -dBATCH -dPrinted=false -sOutputFile=output.pdf input.pdf
    

That's just about everything I think you need to get started. Any questions or suggestions are very welcome!
