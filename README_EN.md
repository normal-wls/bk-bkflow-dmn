# bkflow-dmn: A DMN (Decision Model Notation) library for Python

## Introduction
bkflow-dmn is a Python-based DMN (Decision Model Notation) library that uses FEEL (Friendly Enough Expression Language) as its description language. It can serve as a decision engine to solve decision problems in real business scenarios.

## Quick Start

### 1. Install dependencies

```
$ pip install bkflow-dmn
```

### 2. Construct decision table data
Currently, bkflow-dmn supports single table decisions. It allows describing decision tables using Python dictionaries, as shown in the example below:

``` python
decision_table = {
    "title": "decision table", # Table title, currently has no practical use
    "hit_policy": "Unique", # Hit policy
    "inputs": { # Input table
        "cols": [{"id": "score"}, {"id": "sex"}], # Column descriptions
        "rows": [ # Values corresponding to rows
            ["[0..70]", '"boy"'],
            ["(70..80]", '"boy"'],
            ["(80..90]", '"boy"'],  # For actual scenarios, if you want to [represent multiple column expressions in a single row], use a string type description, like 'score in (80..90] and sex="boy"'
            ["(90..100]", '"boy"'],
            ["[0..60]", '"girl"'],
            ["(60..70]", '"girl"'],
            ["(70..80]", '"girl"'],
            ["(80..100]", '"girl"'],
        ],
    },
    "outputs": { # Output table
        "cols": [{"id": "level"}], # Column descriptions
        "rows": [ # Values corresponding to rows
            ['"bad"'],
            ['"good"'],
            ['"good+"'],
            ['"good++"'],
            ['"bad"'],
            ['"good"'],
            ['"good+"'],
            ['"good++"'],
        ],
    },
}
```

The input table defaults to evaluating the expression in each cell as a whole for truth value determination. It also supports some shorthand notations (only when not using [single row to represent multiple column expressions]):

| Shorthand | Completed expression (assuming column name is x) |
|-----------|--------------------------------------------------|
| ">=10"    | "x>=10"                                          |
| ">10"     | "x>10"                                           |
| "<=10"    | "x<=10"                                          |
| "<10"     | "x<10"                                           |
| "[1..2]"  | "x in [1..2]"                                    |
| "(1..2]"  | "x in (1..2]"                                    |
| '"boy"'   | 'x="boy"'                                        |
| "1"       | "x=1"                                            |
| "1.2"     | "x=1.2"                                          |
| "-1"      | "x=-1"                                           |
| "-2.3"    | "x=-2.3"                                         |
| "true"    | "x=true"                                         |
| "false"   | "x=false"                                        |
| "null"    | "x=null"                                         |

### 3. Call the function to make decisions

``` python
from bkflow_dmn.api import decide_single_table

facts = {"score": 10, "sex": "boy"} # Column data and context for the input table
result = decide_single_table(decision_table, facts, strict_mode=True)  # [{"level": "bad"}]
```

For multi-table decisions, you can currently combine it with the process engine [bkflow-engine](https://github.com/TencentBlueKing/bamboo-engine).

### 4. Supported hit policies
| Hit Policy     | Description                                                                                                                             |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Unique         | Unique hit, in strict mode, only one rule should be hit                                                                                 |
| Any            | Any hit, in strict mode, there must be a hit result and the hit result must be unique                                                   |
| First          | First hit, in strict mode, there must be a rule hit                                                                                     |
| Priority       | Priority hit, using output as sorting priority, in strict mode, there must be a rule hit and the priority sorting result must be unique |
| RuleOrder      | Multiple rule hit policy, output according to the order of rule definition                                                              |
| OutputOrder    | Multiple output hit policy, sort and output according to the output order                                                               |
| Collect        | Multiple output hit policy, output all matching results                                                                                 |
| Collect(Sum)   | Multiple output hit policy, sum the output values                                                                                       |
| Collect(Max)   | Multiple output hit policy, find the maximum of the output values                                                                       |
| Collect(Min)   | Multiple output hit policy, find the minimum of the output values                                                                       |
| Collect(Count) | Multiple output hit policy, count the number of output values                                                                           |
