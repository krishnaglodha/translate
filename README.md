# MkDocs Translate

A translate script is provided to facilitate working with pandoc and deepl translation services.

1. To install development version use:

   ```
   pip install git+https://github.com/jodygarnett/translate.git
   ```
   
2. Install script requirements, and check it runs:

   ```
   mkdocs_translate --help
   ```

3. The script is runs from the location of your mkdocs project (with `docs` and `mkdocs.yml` files):

   ```
   cd core-genetwork/docs/manual
   ```

3. The script makes use of ``target`` folder for scratch files:

   ```
   mkdir target
   ```

4. This script requires ***pandoc*** be installed:

   Ubuntu:
   ```bash
   apt-get install pandoc
   ```

   macOS:
   ``` bash
   brew install pandoc
   ```

   References:

   * https://pandoc.org/installing.html

## Format conversion from sphinx-build rst files

1. Copy everything over (so all the images and so on are present)
   
   ```
   cd core-geonetwork/docs
   copy -r manuals/source manual/doc
   ```
   
2. Use ``docs/.gitignore`` to carefully avoid committing:
   
   ```
   *.rst
   conf.py
   ```
   
3. To index references in rst files into `docs/anchors.txt`:

   ```
   cd core-geonetwork/docs/manual
   mkdocs_translate index
   ```

4. To bulk convert all content from ``rst`` to ``md``:
   
   ```
   cd core-geonetwork/docs/manual
   mkdocs_translate rst docs/contributing/doing-a-release.rst
   ```
   
5. Review this content you may find individual files to fix.

   Some formatting is easier to fix in the `rst` files before conversion:
   
   * Indention of nested lists in ``rst`` is often incorrect, resulting in restarted numbering or block quotes.
  
   * Random ``{.title-ref}`` snippets is a general indication to simplify the rst and re-translate.

   * literalinclude (of source code or configuration file) not yet handled, recommend mdkdocs-include plugin.

6. Convert a single file:
   
   ```
   cd core-geonetwork/docs/manual
   mkdocs_translate rst docs/contributing/doing-a-release.rst
   ```

7. Bulk convert files in a folder:
   
   ```
   cd core-geonetwork/docs/manual
   mkdocs_translate rst docs/introduction/**/*.rst
   ```

8. To copy any edited `rst` files back to the origional rst folder:

   ```
   cd core-geonetwork/docs/manual/doc
   find . -name '*.rst' | cpio -pdm  ../../manuals/source
   ```

9. To clean up ``rst`` files from `docs/` folder when finished:

   ```
   find . -type f -regex ".*\.rst" -delete
   rm docs/conf.py
   ```

## Language Translation

Translations are listed alongside english markdown:

* `example.md`
* `example.fr.md`

Using ***pandoc*** to convert to `html`, and then using the [Deepl REST API](http://deepl.com).

4. Provide environmental variable with Deepl authentication key:

   ```
   export DEEPL_AUTH="xxxxxxxx-xxx-...-xxxxx:fx"
   ```

5. Translate a document to french using pandoc and deepl:

   ```
   mkdocs_translate french docs/help/index.md
   ```
   
6. To translate several documents in a folder:

   ```
   mkdocs_translate french docs/overview/*.md
   ```
   
   Deepl charges by the character so bulk translation not advisable.

See ``mkdocs_translate french --help`` for more options.

You are welcome to use  google translate, ChatGPT, or Deepl directly - keeping in mind markdown formatting may be lost.

Please see the writing guide for what mkdocs functionality is supported.

## Local Development

To build and test locally:

1. Clone:

   ```
   git clone https://github.com/jodygarnett/translate.git translate
   ```

2. Install requirements:
   ```
   cd translate
   pip3 install -r mkdocs_translate/requirements.txt
   ```

2. Install locally:
   ```
   pip3 install -e .
   ```

Debugging:

1. Recommend troubleshooting a single file at a time:

   ```rst
   mkdocs_translate rst docs/index.rst
   ```
   
2. Compare the temporary files staged for pandoc conversion:

   ```
   bbedit docs/index.rst docs/index.md target/convert/index.tmp.html target/convert/index/tmp.md
   ```
   
3. To turn on logging during conversion:

   ```bash
   mkdocs_translate --log=DEBUG translate.yml rst
   ```

Pandoc:

1. The pandoc plugin settings are in two constants:

   ```python 
    md_extensions_to =
        'markdown+definition_lists+fenced_divs+backtick_code_blocks+fenced_code_attributes-simple_tables+pipe_tables'
    md_extensions_from =
        'markdown+definition_lists+fenced_divs+backtick_code_blocks+fenced_code_attributes+pipe_tables'
   ```

2. The pandoc extensions are chosen to align with mkdocs use of markdown extensions, or with post-processing:

   | markdown extension   | pandoc extension       | post processing |
   |----------------------|------------------------|-----------------|
   | tables               | pipe_tables            |                 |
   | pymdownx.keys        |                        | post processing |
   | pymdownx.superfences | backtick_code_blocks   | post processing | 
   | admonition           | fenced_divs            | post processing |
   
3. To troubleshoot just the markdown to html conversion:
   
   ```bash
   mkdocs_translate internal_html manual/docs/contributing/style-guide.md
   mkdocs_translate internal_markdown target/contributing/style-guide.html

   diff manual/docs/contributing/style-guide.md target/contributing/style-guide.md
   ``` 

## Configuration

For core-geonetwork (or other projects following maven conventions) no configuration is required.

To override configuration on command line add `-concfig <file.yml>` before the command:

```bash
mkdocs_translate --config translate.yml rst
```

The file `mkdocs_translate/config.yml` file contains some settings (defaults are shown below):

* `deepl_base_url`: "https://api-free.deepl.com"
      
   Customize if you are paying customer.
      
* `project_folder`: "."

* `rst_folder`: "docs"

* `build_folder`: "target"
   
   The use of "target" follows maven convention, python projects may wish to use "build"

* `docs_folder`: "docs"
   
   mkdocs convention.
   
* `anchor_file`: 'anchors.txt'
  
* `upload_folder`: "translate"
  
   Combined with ``build_folder`` to stage html files for translation (example:  `build/translate`)
   
* `convert_folder`: "convert"

   Combined with ``build_folder`` for rst conversion temporary files (example:  `build/convert`).
   Temporary files are required for use by pandoc.
   
* `download_folder`: "translate"
   
   Combined with ``build_folder`` to retrieve translation results (example:  `build/translate`)
   Temporary files are required for use by pandoc.
