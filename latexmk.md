# Configuring `latexmk`

`latexmk` is a standard tool in LaTeX distributions which will call the latex compiler (`pdflatex` or `lualatex`) the correct amount of times to have things such as references and citations filled out correctly.

An alternative to using your LaTeX editor to handle the compilation would be to run the following in the command line:
```bash
latexmk -pdf -pvc file.tex
```
This compiles the `file.tex` file to a PDF and opens a preview; but also watches for any changes in relevant files and rebuilds the document when necessary.

## Why should I know about `latexmk`?

Even if you use an editor to handle the compilation and preview of your document, there's a good chance that it already uses `latexmk` under the hood, or can be configured to do so.
This means that by adding a `latexmk` configuration file for your project, you could configure the way your document is built in a way that's agnostic to the editor.

This could be a benefit if you...
- want the flexibility to use different tools on different computers for the same project
- have multiple people on a project with different editors
- want to reduce your dependence on specific tools

And by "configuration", I mean things including:
- name and location of the output file ([here](#specify-an-output-location))
- whether to use `lualatex` or `pdflatex` ([here](#specifying-the-command-to-compile-latex))
- extra options to pass to the above, such as:
  - whether to ask for interaction when there's an error
  - whether to allow "shell escape" (necessary for some packages)
  - whether to use an existing "format file" (see [preamble precompilation](https://lukideangeometry.xyz/blog/preamble-compilation))
- locations of extra places for packages or tex files ([here](#specify-extra-locations-to-look-for-files))
- what to do when a file is not found (with the `-use-make` flag, not discussed here)

## Useful configurations for `latexmk`

To configure `latexmk` for a project, add a file called `latexmkrc` in the same directory.
When you call `latexmkrc` in the same directory this file will be read,
saving you having to pass these configurations as command line arguments to `latexmk`
(what is typically done by LaTeX editors).
> Configurations which are project agnostic could go in a global `latexmkrc` file instead
(location specified [here](https://man.archlinux.org/man/latexmk.1#CONFIGURATION/INITIALIZATION_(RC)_FILES))

Full documentation for `latexmk` can be found [here](https://man.archlinux.org/man/latexmk.1),
but to get started here are a few things you might consider putting in your `latexmkrc` file:

### Enabling PDF mode
Most likely relevant good in all your projects, to build PDFs (as opposed to other targets).
This saves calling `latexmk` with the `-pdf` flag, and should really be the default setting.
```perl
$pdf_mode = 1;
```

### Specifying the main TeX file
```perl
@default_files = ('file.tex');
```
By default, `latexmk` will find all `.tex` files in the current directory and try to compile them if you didn't specify any.
If there are extra `.tex` files for sections or chapters, attempting to compile them individually will result in errors
anyway, so you might as well make sure that `latexmk` knows which is the main file.

### Specifying the command to compile LaTeX

The `$pdflatex` variable can be defined to specify what command to be run in place of `pdflatex`. This can be used to specify to use `lualatex` instead, but also to specify any extra command line arguments to use.
You may consider using one of the following examples as a starting point:
```perl
$pdflatex = 'pdflatex %O %S';
$pdflatex = 'pdflatex -interaction=nonstopmode %O %S'; # Do not ask to fix errors interactively
# (^ if a package was not installed, I'm not going happen to have the sty file somewhere and type out its path! Just cancel the run and let me install the package before trying again!!)
$pdflatex = 'lualatex %O %S';
$pdflatex = 'lualatex -shell-escape %O %S'; # necessary for some packages which run shell commands
```
> The `%O` and `%S` are placeholders for extra options and the file to compile, respectively.

### Specify an output location
```perl
$out_dir = '<OUTPUT_FOLDER>';
# e.g. $out_dir = 'output';
```
Instead of scrolling through a sea of auxiliary files to find the tex file or PDF,
put the output files (including the PDF) in the location specified.

### Specify a location for the auxiliary files
```perl
$aux_dir = '<folder>';
# e.g. $aux_dir = 'aux_files';
```
Put all the auxiliary files (random files produced by LaTeX) in the location specified.
Gives a similar benefit of decluttering.
Also simplifies your `.gitignore` file if you use `git`.

### Base name of output files
LaTeX compilers can be provided with a "jobname" which is used to name the output files, including the PDF.
```perl
$jobname = "PAPER_TITLE";
```

### Specify extra locations to look for files

Maybe you want to use a tex file or package that's in a different folder, you can make you LaTeX compile search
this folder by adding it to the `TEXINPUTS` environment variable.
This could be done in the command line, in you shell init script (such as `.bashrc`),
or you could enforce this in the `latexmkrc` file like so:
```perl
ensure_path("TEXINPUTS", "PATH_TO_OTHER_STY_OR_TEX_FILES");
```
However you may want to consider alternatives:
- put personal preambles or packages in you TEXMF tree
- specify the relative path of sub-parts of your document directly (e.g. `\input{../other_folder/other_file.tex}` or `\import{../other_folder/}{other_file.tex}`)

### Allow LaTeX compiler to run even more times

By default, `latexmk` will run the LaTeX compiler until the references and citations are filled out correctly,
but a maximum of 5 times.
If for whatever reason, you need to run the compiler more times, you can specify this in the `latexmkrc` file:
```perl
$max_repeat = 6;
```
