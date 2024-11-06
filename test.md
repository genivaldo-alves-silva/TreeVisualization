# Tree Visualization Script

This script uses **Toytree** and **Toyplot** to load, root, and visualize a phylogenetic tree from a `.tre` file. Nodes are customized based on their support values, and the resulting tree can be saved in multiple formats.

## Requirements

Make sure to install the following packages:
```bash
pip install toytree toyplot
```

## Script Breakdown

### Import Libraries
We begin by importing the necessary libraries:
```python
import toytree
import re
import toyplot.pdf
import toyplot.png
import toyplot.svg
```

### Load and Root the Tree
Specify the tree file path and load it using `toytree`. Then, root the tree using the **Most Recent Common Ancestor (MRCA)** of nodes labeled as "outgroup":
```python
# Load the phylogenetic tree from the file
tree_url = "tree.tre"
tree = toytree.tree(tree_url)

# Find tips that match 'outgroup'
matching_tips = [name for name in tree.get_tip_labels() if 'outgroup' in name]

# Root the tree using the MRCA of the matched tips
if matching_tips:
    mrca = tree.get_mrca_node(*matching_tips)
    rooted_tree = tree.root(mrca)
```

### Ladderize the Tree
Ladderizing the tree organizes branches in a way that improves visual clarity.
```python
# Ladderize the tree (arrange branches for clarity)
ladderized_tree = rooted_tree.ladderize()
```

### Customize Node Appearance
Initialize lists to control each node’s size, shape, and color. For each internal node, the **support value** is checked, and appearance attributes are assigned accordingly.
```python
# Initialize lists for node customization
node_sizes = [0 for _ in range(ladderized_tree.nnodes)]  # Default size 0
node_markers = ['' for _ in range(ladderized_tree.nnodes)]  # Default empty markers
node_colors = ['black' for _ in range(ladderized_tree.nnodes)]  # Default black color

# Customize nodes based on support values
for node in ladderized_tree.treenode.traverse():
    if node.is_leaf():
        continue  # Skip terminal nodes
    support_value = node.support  # Get support value

    # Set node attributes based on support thresholds
    if support_value is not None:
        if support_value >= 99:
            node_sizes[node.idx] = 12
            node_markers[node.idx] = 'o'  # Circle marker
            node_colors[node.idx] = "black"
        elif 70 <= support_value < 99:
            node_sizes[node.idx] = 12
            node_markers[node.idx] = 's'  # Square marker
            node_colors[node.idx] = "gray"
        elif 60 <= support_value < 70:
            node_sizes[node.idx] = 12
            node_markers[node.idx] = 'd'  # Diamond marker
            node_colors[node.idx] = "white"
        elif 50 <= support_value < 60:
            node_sizes[node.idx] = 12
            node_markers[node.idx] = 'v'  # Triangle marker
            node_colors[node.idx] = "white"
```

### Draw the Tree
Using `ladderized_tree.draw()`, the tree is rendered with customized tip labels, node sizes, markers, and colors.
```python
canvas, axes, mark = ladderized_tree.draw(
    width=1200,
    height=1200,
    tip_labels_align=False,
    tip_labels_style={
        "fill": "#262626",          # Text color
        "font-size": "18px",        # Font size in pixels
        "-toyplot-anchor-shift": "15px",  # Label position adjustment
    },
    node_labels=None,          # Hide support values
    node_sizes=node_sizes,     # Node sizes
    node_markers=node_markers, # Node markers
    node_colors=node_colors,   # Node colors
    edge_style={
        "stroke": "black", 
        "stroke-width": 1,
    },
)
```

### Save the Tree to Files
Uncomment any of the following lines to save the visualization in your desired format:
```python
# Save the canvas to different formats
# toyplot.pdf.render(canvas, "tree_visualization.pdf")  # Save as PDF
# toyplot.png.render(canvas, "tree_visualization.png")  # Save as PNG
toyplot.svg.render(canvas, "out/tree_markers.svg")    # Save as SVG
```
