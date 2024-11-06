# Tree Visualization With Support Values

This script uses Toytree and Toyplot to load, root, and visualize a phylogenetic tree (`.tre`) from a local file. It applies specific styling based on node support values, generates a custom visualization, and exports the output to an SVG file.

## Import Libraries

- `html`: Escapes XML-invalid characters.
- `toytree`: Loads and processes the tree structure.
- `re`: Handles regular expressions for label matching.
- `toyplot.pdf`, `toyplot.png`, `toyplot.svg`: Export functions for rendering the tree in different formats.

```python
import html
import toytree
import re
import toyplot.pdf
import toyplot.png
import toyplot.svg
```

## Load Tree Data

- Loads the tree from a specified file (`tree.tre`).
- Finds tips that match a given pattern (`'militaris'` in this example).

```python
tree_url = "tree.tre"
tree = toytree.tree(tree_url)

# Find tips that match 'militaris'
matching_tips = [name for name in tree.get_tip_labels() if 'militaris' in name]
```

## Root the Tree Using the Most Recent Common Ancestor (MRCA)

If matching tips are found, the tree is rooted based on their MRCA.

```python
if matching_tips:
    mrca = tree.get_mrca_node(*matching_tips)
    rooted_tree = tree.root(mrca)
```

## Ladderize the Tree

Ladderizing the tree arranges nodes to create a more visually appealing structure.

```python
ladderized_tree = rooted_tree.ladderize()
```

## Initialize Node Attributes

- `node_labels`: List to hold labels for each node, with empty strings by default.
- `node_sizes`: List of sizes for each node, initialized to zero.
- `node_markers`: Specifies the marker type for each node (e.g., square, triangle).
- `node_colors`: Colors for each node, defaulting to black.

```python
node_labels = ['' for _ in range(ladderized_tree.nnodes)]
node_sizes = [0 for _ in range(ladderized_tree.nnodes)]
node_markers = ['' for _ in range(ladderized_tree.nnodes)]
node_colors = ['black' for _ in range(ladderized_tree.nnodes)]
```

## Traverse Internal Nodes and Apply Support-based Styling

Adds labels, sizes, and markers to internal nodes based on their support values (e.g., squares for nodes with support ≥ 50).

```python
for node in ladderized_tree.treenode.traverse():
    if node.is_leaf():
        continue  # Skip terminal nodes
    support_value = node.support
    if support_value is not None and support_value >= 50:
        node_labels[node.idx] = str(int(support_value))
        node_sizes[node.idx] = 15
        node_markers[node.idx] = "s"
```

## Draw the Tree

Visualizes the tree with custom tip labels, node sizes, markers, and colors.

- `tip_labels_style`: Styles the tip labels with custom font size, color, and position.
- `node_labels_style`: Styles internal node labels based on support values.
- `edge_style`: Sets the line color and width for edges.

```python
canvas, axes, mark = ladderized_tree.draw(
    width=2000,
    height=5000,
    tip_labels_align=False,
    tip_labels_style={
        "fill": "#262626",
        "font-size": "18px",
        "-toyplot-anchor-shift": "15px",
    },
    node_labels=node_labels,
    node_labels_style={
        "fill": "#262626",
        "font-size": "15px",
    },
    node_sizes=None,
    node_markers=node_markers,
    node_colors=node_colors,
    edge_style={
        "stroke": "black", 
        "stroke-width": 1,
    },
)
```

## Export the Visualization

The canvas is saved as an SVG file. Other formats, such as PDF and PNG, can be enabled by uncommenting the corresponding lines.

```python
#toyplot.pdf.render(canvas, "tree_visualization.pdf")
#toyplot.png.render(canvas, "tree_visualization.png")
toyplot.svg.render(canvas, "out/tree_supportvalue.svg")
```

This script enables easy customization of tree visualization by adjusting styling based on node support values and provides export options in multiple formats for publication or sharing.
```
