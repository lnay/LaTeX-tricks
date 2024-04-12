# Configuring `latexmk`

`latexmk` is a standard tool in LaTeX distributions which will call the latex compiler (`pdflatex` or `lualatex`) the correct amount of times to have things such as references and citations filled out correctly.

An alternative to using your LaTeX editor to handle the compilation would be to run the following in the command line:
```bash
latexmk -pdf -pvc file.tex
```
This compiles the `file.tex` file to a PDF and opens a preview; but also watches for any changes in relevant files and rebuilds the document when necessary.

## Why should I know about `latexmk`?

Even if you use an editor to handle the compilation and preview of your document, it almost certainly uses `latexmk` under the hood, or can be configured to used `latexmk`.
This means that by adding a `latexmk` configuration file for your project, you could configure the way your document is built in a way that's agnostic to the editor.

This could be a benefit if you...
- Want the flexibility to use different tools on different computers for the same project
- Have multiple people on a project with personal setups
- Want to reduce your dependence on specific tools

And by "configuration", I mean things including:
- Name and location of the output file
- Whether to use `lualatex` or `pdflatex`
- Extra options to pass to the above, such as:
  - whether to ask for interaction when there's an error
  - whether to allow "shell escape" (necessary for some packages)
  - whether to use an existing "format file" (see [preamble precompilation](preamble-precompilation.md))
- What to do when a file is not found
- Locations of extra places for packages (`.sty` files)

## Useful configurations for `latexmk`

To configure `latexmk` for a project, add a file called `latexmkrc` in the same directory. When you call `latexmkrc` in the same directory, this file will be read and saves passing these configurations as command line arguments to `latexmk` (what is typically done by LaTeX editors).

Here are a few things you might consider putting in your `latexmkrc` file:

### Enabling PDF mode
```perl
$pdf_mode = 1;
```

Most likely in all your projects, to build PDFs (as opposed to other targets). This saves calling `latexmk` with the `-pdf` flag.

### Specifying the main TeX file
```perl
@default_files = ('file.tex');
```
By default, `latexmk` will find all `.tex` files in the current directory and try to compile them if you didn't specify any.

### Specifying the command to compile LaTeX

The `$pdflatex` variable can be defined to specify what command to be run in place of `pdflatex`. This can be used to specify to use `lualatex` instead, but also to specify any extra command line arguments to use. 
You may consider to use one of the following lines:
```perl
$pdflatex = 'pdflatex';
$pdflatex = 'pdflatex -interaction=nonstopmode'; # Do not ask user to fix errors interactively
$pdflatex = 'lualatex';
$pdflatex = 'lualatex -shell-escape'; # necessary for some packages
```

### Specify an output location
```perl
$out_dir = '<output_folder>';
# e.g. $out_dir = 'output';
```
Instead of scrolling through a sea of auxiliary files to find the PDF, put the output files (including the PDF) in the location specified.

### Specify a location for the auxiliary files
```perl
$aux_dir = '<folder>';
# e.g. $aux_dir = 'aux_files';
```
Put all the aux files (random files produced by LaTeX) in the location specified. Gives a similar benefit of decluterring. Also simplifies your `.gitignore` file if you use `git`.

### Specify extra locations to look for packages

Maybe you have a custom 