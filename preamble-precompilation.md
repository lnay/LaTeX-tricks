# Preamble Precompilation

If you have ever looked at the output for `pdflatex` when compiling a document, you may notice a wall of text listing out file paths as it's searching for packages that you loaded in the preamble.

This part of the compilation can be non-negligeable (time-wise), and is slower the more packages you have installed (even if not used in the document).
Furthermore, this step in the process is repeated on subsequent runs of `pdflatex`, which may have to run 2-3 times to get things such as references and citations correct. Most of the time when editing a document, you do not change the preamble and would benefit from a slightly faster preview response.

A trick to optimise this process is to process the preamble once, or at least the parts of the preamble which can be processed once before the rest of the document.

*Limitations*: Your mileage may vary with `lualatex`, this works best with `pdflatex`. Also some packages such as `sagetex` do not work properly in a precompiled preamble.

## Step 1: Move preamble to separate file

Suppose you are working in a single TeX file `main.tex`, cut all the contents before the `\begin{document}` and paste it into a separate file, say `preamble.tex`.
Leaving the main LaTeX file with only the document environment and it's contents:

```latex
% main.tex with preamble removed
\begin{document}
\section{Introduction}
%...
\end{document}
```
And all the rest in the other file:
```latex
% new preamble.tex file
\documentclass[...]{...}
\usepackage{...}
%[...]

\newcommand{...}{...}
%[...]

\title{...}
\author{...}
\date{...}
```

### Optional (splitting up more)

A preamble contains multiple different things, including the document class, document details, custom commands, packages...
Some of these could be split up into further files to have a `preamble.tex` file looking like this:

```latex
% preamble.tex
\documentclass[...]{...}

\input{packages.tex}
\input{newcommands.tex}
\input{theoremstyles.tex}

\title{...}
\author{...}
\date{...}
```

As well as the minor benefit in organisation, this can also help with some quality of life improvements. For example, in VSCode, you can hover over sections of inline-math to get a pop-up with a render of that section. This does not involve compiling the whole document properly, so in particular, does not look at your preamble, and will not correctly render any custom commands defined. However, in the settings for the LaTeX plugin, you can point it to a file containing the custom commands used in the document.

Another benefit to splitting the project into several files is to allow more control when using one project as part of the content for another document. There are tools out there which allow you to import a whole other tex file and sorting out the multiple preambles. However I have run into issues when trying this with larger, more complicated projects and resort to more strategic and targeted importing of parts of a subproject's content and preamble into the larger project.

## Step 2: Compile the new file to a "format file"

Run the following command from the terminal in the same directory:
```bash
pdflatex -ini -jobname="preamble" "&pdflatex preamble.tex\dump"
```
This will create a new file called `preamble.fmt`.

### Optional (Makefile)

If you have `make` installed (a standard tool which you may have without even knowing), then we can tell it this "rule" for creating the `preamble.fmt` file by creating a new file named `Makefile` in the same directory:

```Makefile
# Makefile
preamble.fmt: preamble.tex
    pdflatex -ini -jobname="preamble" "&pdflatex preamble.tex\dump"
# Note that this indent^ must be a tab character (not spaces)

# optional shorthand:
preamble: preamble.fmt

```
So now running `make preamble.fmt` (or `make preamble` if using the last lines) will create the format file as required. A neat part of using `make` is that it will not bother recreating `preamble.fmt` if it is already newer than it's dependency `preamble.tex`.


## Step 3: Compile the main document

Now that `preamble.fmt` is created, you can compile the main document referencing this "format file":

```bash
pdflatex -fmt preamble <other options> main.tex
```

*Troubleshooting*: if it complains about a package, it's possible that it cannot be in the precompiled preamble. In this case, try moving that `\usepackage{...}` line back to the main tex file.

If any changes to the preamble are made, make sure to rerun step 2.

### Optional (latexmk)

To avoid remembering this command, stick it in your `latexmkrc` file for the project (in the same directory, or the one where you call `latexmk`):
```perl
# latexmkrc
@default_files = ('main.tex');
$pdf_mode = 1;
$pdflatex = 'pdflatex -fmt preamble <other options>';
# ...
```

Now, when opening your project, you can run in the terminal (assuming you followed the previous optional section):
```bash
make preamble
latexmk -pvc
```
to then have an updating preview of your document as you edit it. And then if you do make a change to the preamble, run `make preamble` in a second terminal, and it will trigger `latexmk -pvc` to rebuild the document.
