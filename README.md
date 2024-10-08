# JuPDF

JuPDF is a small Python package to convert `.ipynb` files to single `.pdf` files via the use of Pandoc.

### Supported Features

- All standard markdown features supported in Pandoc conversion, such as lists, tables, etc.
- PNG images within markdown (i.e. `![...](...png)`).
- LaTeX mathematics, including in-line LaTeX.
- `stdout` code-cell output.
- `PNG` code-cell output (e.g. the output of `plot.show()` from `matplotlib`).
- `YAML` metadata for use with the `eisvogel.tex` template.

> Other image types are planned to be supported in future versions. For now, please ensure both markdown images and code output images are `.png`.

### Requirements
- In order to use JuPDF, as well as any dependencies handled by `pip`, you must have Pandoc installed on your system, as the conversion process utilizes the `pandoc` command.

- You must also have a TeX engine installed on your system. For example, on Windows, I use MikTeX.

---

## Basic Usage

### Converting a single `.ipynb` file to `.pdf`

```py
from jupdf.pypdfnb import PYPDFNB
from jupdf.pypdfnb_jobs import single_to_pdf

pypdfnb = PYPDFNB()
pypdfnb.read_ipynb('notebook.ipynb')
single_to_pdf(pypdfnb, 'notebook.pdf')
```

### Converting multiple `.ipynb` files to `.pdf`

```py
from jupdf.pypdfnb import PYPDFNB
from jupdf.pypdfnb_jobs import multiple_to_pdf

pypdfnb_a, pypdfnb_b = PYPDFNB(), PYPDFNB()
pypdfnb_a.read_ipynb('notebook_a.ipynb')
pypdfnb_b.read_ipynb('notebook_b.ipynb')
multiple_to_pdf([pypdfnb_a, pypdfnb_b], 'notebook.pdf')
```

### Setting a `PYPDFNB` instance's `contents` property back to `[]`
```py
from jupdf.pypdfnb import PYPDFNB

pypdfnb_instance = PYPDFNB()
pypdfnb_instance.read_ipynb('notebook.ipynb')

# do something . . .

pypdfnb_instance.empty()
```

---

## `eisvogel.tex` Metadata

Before converting a `PYPDFNB` instance to a `.pdf` file, you can set various `YAML` metadata attributes of the instance that will affect how Pandoc converts the notebook markdown to PDF using the `eisvogel.tex` template.

The code below shows the `YAML` metadata properties available for `PYPDFNB` instances, and their values on initialization.

```py
def __init__(...):

    # other attributes ...

    self.title: Optional[str] = None
    self.author: Optional[str] = None
    self.date: Optional[str] = None
    self.subject: Optional[str] = None
    self.keywords: Optional[list[str]] = None
    self.lang: Optional[str] = None
    self.listings: bool = False
    self.titlepage: bool = False
```

- `title` - setting this will place the string in the top-left of the PDF pages.
- `author` - setting this will place the string in the bottom-left of the PDF pages.
- `date` - setting this will place the string in the top-right of the PDF pages.
- `subject` - *non-visual* - the subject of the PDF document.
- `keywords` - *non-visual* - keywords associated with the PDF document.
- `lang` - *non-visual* - the language code of the document (e.g. `'en'`).
- `listings` - whether or not to use Pandoc listings during conversion.
- `titlepage` - whether or not to insert a title page at the start of the PDF, which will include the `title`, `author` and `date`.

---

## Parsers

JuPDF provides a few different parsing Callables that can be passed to `PYPDFNB` instances. These callables determine how a `.ipynb` is read, therefore determining how a PDF will look following conversion. 

There are currently two distinct types of parsers: cell parsers and code parsers. Cell parsers will determine how cells are placed within the documuent, where as code parsers determine how code cells should be handled within the document.

| Callable | Type | Description |
| --- | --- | --- |
`cell_parser_regular` | cell | Parses cells such that cells are placed in the next available space in a PDF.
`cell_parser_one_cell_per_page` | cell | Parses cells such that every cell ends with a page break.
`cell_parser_one_md_cell_per_page` | cell | Parses cells such that each markdown cell specifically starts with a page break.
`code_parser_regular` | code | Parses code cells such that both the code itself and the code's output are included within the PDF. |
`code_parser_source_only` | code | Parses code cells such that only the code itself is included within the PDF. |
`code_parser_output_only` | code | Parses code cells such that only the code's output is included within the PDF. |

An example of using these parsers is shown below.

```py
# Convert a Jupyter Notebook to PDF, whereby each cell starts on a seperate page, and only the output of
# code cells is included in the PDF.

from jupdf.pypdfnb import PYPDFNB
from jupdf.pypdfnb_parsing import cell_parser_one_cell_per_page, code_parser_output_only
from jupdf.pypdfnb_jobs import single_to_pdf

pypdfnb = PYPDFNB(cell_parser_one_cell_per_page, code_parser_output_only)
pypdfnb.read_ipynb('notebook.ipynb')
single_to_pdf(pypdfnb, 'notebook.pdf')
```

---

## Saving time with Saved Parses

Suppose you have a massive `.ipynb` file that you think you'll need to convert several times. If the file is big enough, then reading and parsing the file may take some time. As such, you likely do not wish to repeat this process again and again. This is where `.pypdfnb` files come into play.

Using the `write_pypdfnb` instance method, you can write the current contents of a `PYPDFNB` instance to a `.pypdfnb` file. Now, whenever you need to convert that massive file, you can use the `read_pypdfnb` instance method instead of `read_ipynb`, which will require no parsing.

```py
from jupdf.pypdfnb import PYPDFNB
from jupdf.pypdfnb_jobs import single_to_pdf

pypdfnb = PYPDFNB()
pypdfnb.open_ipynb('massive_notebook.ipynb')

# writes massive_notebook.pypdfnb to a pypdfnbs directory - this method handles the .pypdfnb extension for you!
pypdfnb.write_pypdfnb('massive_notebook', dir='pypdfnbs')

pypdfnb.empty()


# Later on . . .
pypdfnb.read_pypdfnb('pypdfnbs/massive_notebook.pypdfnb')  # No time spent parsing!
single_to_pdf(pypdfnb, 'massive_notebook.pdf')
pypdfnb.empty()

```