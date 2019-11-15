# Coding assignment - Food database

In the context of computer vision, detection tasks with open class domains require more flexibility than flat class distinction in simple vision tasks (dog/cat classification). 



## Context

At Foodvisor, expanding activities in a new regions often means that extra object classes have to be recognized. Below are some reasons why the end target class structure may not be defined at a given moment $t$:

- **Data availability**: not enough existing images of the corresponding item at $t$
- **Task evaluation changes**: user expectations about detection granularity changes

We'll define two concepts:

- **coverage**: ratio of observed classes that are correctly categorized in ground truths, parent category included
- **granularity**: ratio of observed items that have distinct labels in the class structure

Consider the following examples for an image classification problem:

1. **Coverage change**: given a certain labeling budget, your team first decides to define the class structure as follows: fruits, meat, fish, which satisfies users coverage expectations at $t$. Now at $t' > t$, users expects vegetables to be recognized.
2. **Granularity change**: given a certain labeling budget, your team first decides to define the class structure as follows: vegetables, fruits, meat, fish, which satisfies users granularity expectations at $t$. Now at $t' > t$, users expects beef and pork to be distinguished.

Hence the new for a flexible class structure to continuously adapt to changes in user expectations, while maintaining a reasonable labeling budget.



## Assignment

In this assignment, you will define a data structure that handles both coverage and granularity changes using a directed graph structure. Say $O$ is the ensemble of all existing objects, and that, as a first approach we define $A,B,C$ as child nodes of $O$ to be labeled.

-  Consider the case of coverage extension: add $D$ as child node of $O$
- Consider the case of granularity extension: add $E, F$ as child nodes of $A$

Taking a snapshot of your class structure at $t$, you'll want to maximize both coverage and granularity of your existing labeled data. 

- In first case above, if classes are one-hot-encoded, the encoding vector length needs to be extended.
- In second case above, data labeled as $A$ will have to be staged for next labeling task to be either $E$ or $F$, meaning that the information of those being child nodes of $A$, need to persist in your data structure.

### Submission

Your submission is expected to be a GitHub (or similar) repository, implemented in Python3.

Your repository should include:

- instructions to install requirements to run your code
- credits to the different repositories or resources that you used for your implementation



**Task**

Define a `Database` class (in the database.py file) able to continuously maximize coverage and granularity of existing labeled data. A reference abstract category (named `core` in the following examples) will need to be created in your constructor (cf. ensemble $O$ mentioned in the previous section)

**Design requirements**

`Database` class will need to have the following methods implemented:

- `add_nodes`: takes a list of tuples as input and edit the graph
- `add_extract`: takes a dict as input and stored information appropriately
- `get_extract_status`: returns the status of each image considering graph modifications that occurred after the extract was added. 

Feel free to create any additional classes or data structures you deem necessary.

**Inputs**

The `add_nodes` method takes a list of tuples as input. Each tuple has two elements: the first being the ID of a new node, and the second the ID of the parent node. If the parent node is `None`, this operation should be conducted first and will define the root node. Each non-empty graph will start with a tuple whose second member is `None` (it is the expected name of your core abstract category that needs to be created when the graph is instantiated).



Once created, nodes can only be added to the graph. But depending on the parent node, it will have different effect on our class structure:

- coverage extension

```python
[("A", "core"), ("B", "core"), ("C", "core")]
```

- granularity extension

```python
[("A1", "A"), ("A2", "A"), ("C1", "C")]
```

- mixed operation

```python
[("D", "core"), ("D1", "D"), ("D2", "D")]
```



The `add_extract` method takes a dict as input where the keys are image names, and values are list of class/node IDs (string).

```python
{"img001": ["A"], "img002": ["C1"]}
```



Lastly, the `get_extract_status` method will return the status of each data sample from the extract, which can either be:

- `invalid`: some labels could not be matched against database
- `valid`: label is matched and no operation is required
- `granularity_staged`: label is matched but some labels have new child nodes in the database
- `coverage_staged`: label is matched but direct parent node has a new child node since last update (priority again granularity_staged)

```python
{"img001": "granularity_staged", "img002": "valid"}
```

### Evaluation

Here is an input example

```python
from database import Database

# Initial graph
build = [("core", None), ("A", "core"), ("B", "core"), ("C", "core"), ("C1", "C")]
# Extract
extract = {"img001": ["A"], "img002": ["C1"]}
# Graph edits
edits = [("A1", "A"), ("A2", "A")]

# Get status (this is only an example, test your code as you please as long as it works)
status = {}
if len(build) > 0:
    # Build graph
    db = Database(build[0][0])
    if len(build) > 1:
    	db.add_nodes(build[1:])
    # Add extract
    db.add_extract(extract)
    # Graph edits
    db.add_nodes(edits)
    # Update status
    status = db.get_extract_status()
print(status)
```

should return

```python
{"img001": "granularity_staged", "img002": "valid"}
```

**explanation**

"A" has new child nodes since labeled data was provided thus "granularity_staged" for img001



Here is an another example

```python
from database import Database

# Initial graph
[("core", None), ("A", "core"), ("B", "core"), ("C", "core"), ("C1", "C")]
# Extract
{"img001": ["A", "B"], "img002": ["A", "C1"], "img003": ["B", "E"]}
# Graph edits
[("A1", "A"), ("A2", "A"), ("C2", "C")]

# Get status (this is only an example, test your code as you please as long as it works)
status = {}
if len(build) > 0:
    # Build graph
    db = Database(build[0][0])
    if len(build) > 1:
    	db.add_nodes(build[1:])
    # Add extract
    db.add_extract(extract)
    # Graph edits
    db.add_nodes(edits)
    # Update status
    status = db.get_extract_status()
print(status)
```

and the expected result of `status` method

should return:

```python
{"img001": "granularity_staged", "img002": "coverage_staged", "img003": "invalid"}
```

**explanation**

img001 has "A" as label, which got extended granularity

img002 has "C1" as label, its parent node "C" has a new node "C2" since then. It also has "A" label but coverage staging has priority over granularity staging

img003 has an unmatched label "E"



Your implementation should yield the same `get_extract_status` output as expected. 

Not meeting this success condition does not necessarily mean that your application won't be considered further, but we expect you to produce code of your own. <u>Plagiarism will not be considered lightly.</u>



### Data

Please check this [release](https://github.com/Foodvisor/coding-assignment/releases/tag/v0.1.0) attachments for the data you can use to test your code:

- graph_build.json: graph nodes initial build
- img_extract.json: image template
- graph_edits.json: node additions to perform
- expected_status.json: the expected output of the `get_extract_status` method



Best of luck!