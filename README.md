---
description: Introduction to the Dawn language and it's structure.
---

# 👋 Introduction

## Welcome

Welcome to the Dawn API GitBook! Here I'd like to introduce you to the Dawn compiler structure, the `dawn` CLI tool, and the Dawn syntax (in a nutshell).

### What is Dawn?

Dawn is a "mapping" language i.e. Dawn is a language that compiles to some other language. It's very possible Dawn will eventually remove the "middle-man" so to speak, as in, Dawn won't need another compiler to sustain it. The primary goal of Dawn, though, is to provide maximum cross-compatibility with two languages on the opposite ends of the spectrum.

{% tabs %}
{% tab title="TypeScript" %}
Dawn compiles all of its code directly to TypeScript.

_Typescript example_

```typescript
//*dawn:ts-4.7.2; --strip-doccomments; --strip-dev
class Fraction {
    n: number;
    d: number;
    constructor(n: number, d: number) {
        this.n = n;
        this.d = d;
    };
    
    toString(): string {
        return `${this.n}/${this.d}`;
    };
    
    eval(): number {
        return n/d;
    }
};

export default class Point {
    x: Fraction;
    y: Fraction;
    constructor(x: Fraction, y: Fraction) {
        this.x = x;
        this.y = y;
    };
};
```
{% endtab %}

{% tab title="C" %}
Dawn compiles all of its code directly to C.

_C example_

```c
//*dawn:c99; --strip-doccomments; --strip-dev
#include "dawnc/dawn_string.h"

typedef struct Fraction {
    n: int;
    d: int;
} Fraction;
//*dawn:c99; poly-transform: Fraction(n,d) becomes declaration
void Fraction_constructor(Fraction* this, int n, int d) {
    this->n = n;
    this->d = d;
};
//*dawn:c99; Fraction.toString()
string Fraction_toString(Fraction* this) {
    return genString("{}/{}", this->n, this->d);
};
//*dawn:c99; Fraction.eval()
int Fraction_eval(Fraction* this) {
    return (float)this->n/this->d; //*dawn:c99; Might result in float
}

//*dawn:c99;export default--DefaultExportPoint
typedef struct DE__Point {
    Fraction x;
    Fraction y;
} DE__Point;

//*dawn:c99; poly-transform: Point(n,d) becomes declaration
void* DE__Point_constructor(DE__Point* this, Fraction x, Fraction y) {
    this->x = x;
    this->y = y;
};
```
{% endtab %}
{% endtabs %}

`dawnc` exposes an API for cross-compatibility between C and TypeScript applications.

```typescript
import { CEngine } from `@dawnts/dawnc`;

// reverses Dawn-to-C abstractions
const { Fraction } = CEngine.execFile('./something.c');

// smart arguments
Fraction.constructor(5, 6);

// You have to stringify it, otherwise it'll return a CVariableInstance class
console.log(Fraction.eval().toString());
```

{% hint style="warning" %}
`dawnts` is still a work in progress. Avoid usage until it is safe.
{% endhint %}

`dawnts` exposes the TS-to-C API.

```c
#include "dawnts/engine.h"
CEngine Module = CreateCEngineFromFile("./something.ts");
RunCEngine(*Module);
// get TS elements
Class Fraction = GetClass(Module, "Fraction");
Class a = Initialize(Fraction, 1, 2);
```

### How does Dawn work?

Dawn's compiler structure is actually pretty simple (Dawn is written in TypeScript). Dawn parses Dawn projects and constructs an AST. Then it parses the AST via a series of _passes_. This allows Dawn's code to be modular. For example:

```
struct MyStruct {
    a: int;
    b: int;
};

impl MyStruct {
    init(a: int, b: int) {
        this.a = a;
        this.b = b;
    };
};
```

Could be parsed into an AST similar to:

```typescript
[
    StructDeclaration {
        name: 'MyStruct',
        values: [
            VarTypeDeclaration {
                name: 'a',
                type: 'int'
            },
            VarTypeDeclaration {
                name: 'a',
                type: 'int'
            }
        ]    
    },
    StructImplementation {
        struct: 'MyStruct',
        functions: [
            'init': {
                special: true,
                type: 'constructor',
                definition: FunctionDeclaration {
                    name: 'init',
                    params: [
                        VarTypeDeclaration {
                            name: 'a',
                            type: 'int'
                        },
                        VarTypeDeclaration {
                            name: 'a',
                            type: 'int'
                        }
                    ],
                    body: {
                        EqualityDeclaration {
                            left: AttrReference {
                                root: SpecialKeyword {
                                    name: 'this'
                                },
                                stem: 'a'
                            },
                            right: VarName {
                                name: 'a'
                            }
                        },
                        EqualityDeclaration {
                            left: AttrReference {
                                root: SpecialKeyword {
                                    name: 'this'
                                },
                                stem: 'b'
                            },
                            right: VarName {
                                name: 'b'
                            }
                        }
                    }
                }
            }
        ]
    }
]
```

By attaching a `primitive` function next to these AST elements, we can perform primitive transformations on these elements. We can then refine these transformations with passes to include _context_.

#### Mathematics

Recursive passes require larger amounts of operations than linear ones.

$$
R(d) = \left ( \sum_{n=1}^{d}2^n \right )+1
$$

### `dawn` CLI tool

`dawn` is a super-useful CLI tool for running the Dawn compiler.

#### Args

```rest
dawn [build | fmt | check] (*.dawn)... --output?:c-[spec] | ts-[ver] /
--post?: m(c-fmt | ts-fmt | babel | ugl | compile)
```

We can also download libraries to use in our project using:

```bash
dawntools get pkg-name --global?>-g
```
