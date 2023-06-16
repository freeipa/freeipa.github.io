# FreeIPA website Sources

The documentation is written in [reStructuredText] format and can be build
with [Sphinx] documentation generator. Alternatively, [MyST], a Markdown flavor, can be
used as well.

[reStructuredText]: https://docutils.sourceforge.io/rst.html
[Sphinx]: https://www.sphinx-doc.org
[MyST]: https://mystmd.org/

## Install Python dependencies

It is recommended to install Python dependencies in virtual environment.

```console
$ python3 -m venv .venv
$ source .venv/bin/activate
(.venv) $ pip3 install -r requirements.txt
```

When finished working inside the virtual environment you can step out of it:

```console
(.venv) $ deactivate
```

## Build it
To build the project you need to be in the virtual environment.
```console
(.venv) $ make html
```

Some changes may require a full project rebuild as documents are usually
linked to each other:
```console
(.venv) $ make clean && make html
```

## View it
```console
$ firefox _build/html/index.html
```
You do not need to be in virtual environment to see the build result.

## Add new content

1. Create a new page in reStructuredText or Markdown formats and include it somewhere in
   [src] folder.
2. Every new page needs to be part of a page table of content (ToC) tree. To
   add a page to the ToC tree, either include it in `toctree` directive in an
   existing document or add it to the top level navigation at the end of the [index]
   document.
3. Build the documentation with `make html` and navigate to your page to check
   that everything is displayed as intended.
4. Open a Pull Request against this repository with your change.

[src]: src
[contents]: src/index.rst
