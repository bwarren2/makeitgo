---
title: "Python ASTs"
date: 2020-09-15T23:23:21-04:00
draft: false
summary: "Notes from AST/CST transforming of Python"
---

So I am working on a project with some overly long imports. I have stuff like

```python
from module.file import (
thisclass,
thatclass,
functionA,
...
functionAA,
functionAB,
...
)
```

and that list might be 40 items long; way too much. I have been exploring how to refactor it away, and these are my notes.

I would want that import to become

```python
import module.file as module_file
```

and all the usages would change to

```python
module_file.functionA
```

. I investigated lots of tools; `black`, `isort`, `rope`, PyCharm, and nothing was giving me the level of procedural control I wanted. I was able to experiment with the Python `ast` module:

```python
import ast
import astor

MAX_IMPORT_FROM = 5
to_rewrite = {}


class AnalysisNodeVisitor(ast.NodeVisitor):
    def visit_ImportFrom(self, node):
        names = [x.name for x in node.names]
        if len(names) > MAX_IMPORT_FROM and "services" in node.module:
            for name in names:
                to_rewrite[name] = node.module
        ast.NodeVisitor.generic_visit(self, node)
        return node

    def generic_visit(self, node):
        for field, old_value in ast.iter_fields(node):
            if isinstance(old_value, list):
                new_values = []
                for value in old_value:
                    if isinstance(value, ast.AST):
                        value = self.visit(value)
                        if value is None:
                            continue
                        elif not isinstance(value, ast.AST):
                            if hasattr(value, "id") and value.id in to_rewrite:
                                value.id = "YEET." + value.id
                            new_values.extend(value)
                            continue
                    new_values.append(value)
                old_value[:] = new_values
            elif isinstance(old_value, ast.AST):
                new_node = self.visit(old_value)
                if hasattr(new_node, "id") and new_node.id in to_rewrite:
                    new_node.id = "YEET." + new_node.id
                if new_node is None:
                    delattr(node, field)
                else:
                    setattr(node, field, new_node)
        return node


with open("path/to/file.py", "r") as f:
    data = f.read()
    p = ast.parse(data, "file.py", mode="exec")
    v = AnalysisNodeVisitor()
    v.visit(p)
    print(to_rewrite)
```

but that too waas inadequate; ASTs drop comments and whitespace. PyBowler looked like it might do something good, but in some hand testing it produced invalid output and I abandoned it. The error was useful though: complaining about an invalid CST (note: not AST) turned me onto Concrete Syntax Trees, and eventually libCST. Much better documented than Bowler or undebt, I am optimistic I can make something of it; it even supports the visitor pattern `ast` (and my sample) use.
