# Phylogenetic Tree SVG Formatter

This script processes SVG files to format genus and species names in italics and highlights any "type" keywords in bold. It uses **MyCoPortal** to verify the validity of genus names.

## Requirements

- **Internet connection** for genus verification with MyCoPortal.
- **Libraries:** `requests`, `xml.etree.ElementTree`, and `re`.

```python
import re
import xml.etree.ElementTree as ET
import os
import requests
```

## Code Explanation

### Define SVG Namespace

This ensures XML elements are handled with the correct namespace.
```python
# Define SVG namespace
SVG_NS = "http://www.w3.org/2000/svg"
ET.register_namespace('', SVG_NS)
```

### Genus Validation Function

Checks MyCoPortal for the validity of genus names. Returns `True` if the genus is found.
```python
def check_genus(genus_name):
    url = f"https://www.mycoportal.org/fdex/query.php?qText={genus_name}&qField=taxon"
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return genus_name.lower() in response.text.lower()
        else:
            print(f"Error accessing MyCoPortal: Status {response.status_code}")
            return False
    except Exception as e:
        print(f"Exception occurred: {e}")
        return False
```

### Define Patterns for Genus/Species and 'Type' Words

Using regular expressions, this defines patterns to find:
- Genus/species and associated qualifiers.
- Words containing "type" for bold formatting.
```python
type_pattern = re.compile(r'\b(\w*type\w*)\b', re.IGNORECASE)
genus_pattern = re.compile(
    r'(?P<genus>\b[A-Z][a-z]{3,})\s+'
    r'(?:(?P<qualifier>sp|cf|aff)\.?)?\s*'
    r'(?P<species>[A-Za-z-]+)?\s*'
    r'(?P<voucher>[A-Z][\w]*\s*\d+.*)?'
)
```

### Italicize Genus and Species Names in SVG

This main function iterates through all text elements in an SVG, applies italics to genus and species names, and bolds any "type" keywords.
```python
def italicize_genus_species(svg_file, output_file):
    tree = ET.parse(svg_file)
    root = tree.getroot()
    genus_cache = {}

    for text_elem in root.findall(f'.//{{{SVG_NS}}}text'):
        text = ''.join(text_elem.itertext()).strip()
        adjusted_text = text.replace('_', ' ')

        x_attr = text_elem.get('x')
        y_attr = text_elem.get('y')

        text_elem.clear()
        if x_attr:
            text_elem.set('x', x_attr)
        if y_attr:
            text_elem.set('y', y_attr)

        last_index = 0
        process_text_elements(text_elem, adjusted_text, genus_cache)

    tree.write(output_file, encoding='utf-8', xml_declaration=True)
```

### Process Genus and Species Names

This helper function italicizes genus and species names if verified. It also manages qualifiers like "sp" and adds any voucher as plain text.
```python
def process_text_elements(text_elem, text, genus_cache):
    matches = list(genus_pattern.finditer(text))
    last_index = 0

    for match in matches:
        genus, qualifier, species, voucher = (
            match.group('genus'), match.group('qualifier'),
            match.group('species'), match.group('voucher')
        )

        if genus not in genus_cache:
            genus_cache[genus] = check_genus(genus)

        if match.start() > last_index:
            plain_text = text[last_index:match.start()]
            process_type_words(text_elem, plain_text)

        if genus_cache[genus]:
            italic_genus = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan', attrib={'style': 'font-style:italic'})
            italic_genus.text = genus + ' '
        else:
            plain_genus = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan')
            plain_genus.text = genus + ' '

        if qualifier:
            qualifier_tspan = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan')
            qualifier_tspan.text = f"{qualifier}. "
            if qualifier.lower() == 'sp':
                species = None

        if species:
            italic_species = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan', attrib={'style': 'font-style:italic'})
            italic_species.text = species + ' '

        if voucher:
            voucher_tspan = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan')
            voucher_tspan.text = voucher + ' '

        last_index = match.end()

    if last_index < len(text):
        process_type_words(text_elem, text[last_index:])
```

### Process 'Type' Words

This function finds words containing "type" and applies bold formatting.
```python
def process_type_words(text_elem, text):
    matches = list(type_pattern.finditer(text))
    last_index = 0

    for match in matches:
        if match.start() > last_index:
            tspan_before = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan')
            tspan_before.text = text[last_index:match.start()]

        bold_tspan = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan', attrib={'style': 'font-weight:bold'})
        bold_tspan.text = match.group(1)

        last_index = match.end()

    if last_index < len(text):
        tspan_after = ET.SubElement(text_elem, f'{{{SVG_NS}}}tspan')
        tspan_after.text = text[last_index:]
```

### Process All SVG Files in a Folder

This function iterates through a folder, applies the formatting function to each SVG file, and saves an output file with the suffix `_output`.
```python
def process_svg_folder(input_folder):
    for file_name in os.listdir(input_folder):
        if file_name.endswith('.svg'):
            input_file = os.path.join(input_folder, file_name)
            base_name = os.path.splitext(file_name)[0]
            output_file = os.path.join(input_folder, f"{base_name}_output.svg")

            print(f"Processing file: {input_file}")
            italicize_genus_species(input_file, output_file)
            print(f"Output saved to: {output_file}")
```

### Example Usage

Specify the input folder containing SVG files and process each file.
```python
svg_input_folder = 'out/'  # Adjust path as needed
process_svg_folder(svg_input_folder)
```
