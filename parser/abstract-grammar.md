---
description: Grammar described linguistically (infix notation).
---

# 🍿 Abstract Grammar

> #### Notice
>
> All lexical notation is whitespace-ignoring.

## Notation

Anything enclosed by: `(` EnclosedValue `)` represents optional syntax.

Anything enclosed by: `<` EnclosedValue `>` represents syntactical elements.

## Declaring a Variable

`let` statements

```
let <Identifier>(: <Type>) = <Expression>
```

`const` statements:

```
const <Identifier>(: <Type>) = <Expression>
```

## Classes

```typescript
class <ClassName> {
    constructor\([<Args>(,)]...\) {
        <Body>
    }
    
    <Functions>...
}
```
