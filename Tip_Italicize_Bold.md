## Italicize Genus and Bold "Type" Words in SVG Files (No Internet Connection Needed)

This script processes SVG files in a specified folder, italicizing genus/species names and bolding any words containing "type". It does not require an internet connection and is ideal for offline usage.

## Import Required Libraries

```python
import re
import xml.etree.ElementTree as ET
import os
```

## Define Constants

- `SVG_NS`: Defines the SVG namespace for XML parsing.
- `ET.register_namespace`: Ensures correct namespace handling within SVG files.

```python
SVG_NS = "http://www.w3.org/2000/svg"
ET.register_namespace('', SVG_NS)
```

## Define Regular Expressions

1. **Genus/Species Pattern**: Matches genus names (e.g., *Genus species*) but excludes words containing "type".
2. **Type Pattern**: Matches any word containing "type", case-insensitively.

```python
# Regex for genus/species names (excluding words with "type")
pattern = re.compile(
    r'\b(?!\w*type\b)' 
    r'(?P<genus>[A-Z][a-z]{3,})\s+'
    r'(?:(?P<qualifier>sp|cf|aff)\.?)?\s*'
    r'(?P<species>[\w-]+)?\s*'
    r'(?P<voucher>(?:[A-Z][\w]*\s*\d+.*)?)'
)

# Regex to detect any word with "type" (case-insensitive)
type_pattern = re.compile(r'\b(\w*type\w*)\b', re.IGNORECASE)
```

## Main Function: `italicize_genus_and_bold_types`

Parses each SVG file and applies styles to matching elements:

1. **Italicize Genus and Species**: Checks for genus and species names and italicizes them.
2. **Bold Type Words**: Words containing "type" are bolded.

```python
def italicize_genus_and_bold_types(svg_file, output_file):
    tree = ET.parse(svg_file)
    root = tree.getroot()

    for text_elem in root.findall(f'.//{{{SVG_NS}}}text'):
        text = ''.join(text_elem.itertext()).strip()
        adjusted_text = text.replace('_', ' ')

        matches = list(pattern.finditer(adjusted_text))
        if not matches:
            process_type_words(text_elem, adjusted_text)
            continue

        x_attr = text_elem.get('x')
        y_attr = text_elem.get('y')

        text_elem.clear()
        if x_attr: text_elem.set('x', x_attr)
        if y_attr: text_elem.set('y', y_attr)

        last_index = 0

        for match in matches:
            if match.start() > last_index:
                process_type_words(text_elem, adjusted_text[last_index:match.start()])

            genus = match.group('genus')
            qualifier = match.group('qualifier')
            species = match.group('species')
            voucher = match.group('voucher')

            add_tspan(text_elem, genus + ' ', italic=True)

            if qualifier:
                add_tspan(text_elem, f"{qualifier}. ")
                if qualifier.lower() == 'sp':
                    species = None

            if species:
                add_tspan(text_elem, species + ' ', italic=True)

            if voucher:
                add_tspan(text_elem, voucher + ' ')

            last_index = match.end()

        if last_index < len(adjusted_text):
            process_type_words(text_elem, adjusted_text[last_index:])

    tree.write(output_file, encoding='utf-8', xml_declaration=True)
```

## Helper Function: `process_type_words`

Adds bold styling to words containing "type".

```python
def process_type_words(text_elem, text):
    last_pos = 0

    for match in type_pattern.finditer(text):
        if match.start() > last_pos:
            add_tspan(text_elem, text[last_pos:match.start()])

        add_tspan(text_elem, match.group(1), bold=True)
        last_pos = match.end()

    if last_pos < len(text):
        add_tspan(text_elem, text[last_pos:])
```

## Helper Function: `add_tspan`

Creates `<tspan>` elements with optional italic and bold styles.

```python
def add_tspan(text_elem, content, italic=False, bold=False):
    style = ""
    if italic:
        style += "font-style:italic; "
    if bold:
        style += "font-weight:bold; "

    tspan = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan', attrib={'style': style.strip()})
    tspan.text = content
```

## Processing Multiple SVG Files

The `process_svg_folder` function iterates through a specified folder, applying the `italicize_genus_and_bold_types` function to each SVG file found.

```python
def process_svg_folder(input_folder):
    for file_name in os.listdir(input_folder):
        if file_name.endswith('.svg'):
            input_file = os.path.join(input_folder, file_name)
            output_file = os.path.join(input_folder, f"{os.path.splitext(file_name)[0]}_output.svg")

            print(f"Processing: {input_file}")
            italicize_genus_and_bold_types(input_file, output_file)
            print(f"Saved: {output_file}")
```

## Example Usage

Specify the input folder containing SVG files. Adjust the folder path as needed.

```python
svg_input_folder = 'out/'  # Adjust as needed
process_svg_folder(svg_input_folder)
```
```
