# Guards

[![NPM version](https://badgen.net/npm/v/@hxjs/guards)](https://www.npmjs.com/package/@hxjs/guards)

Composable type predicates for TypeScript.

## Install

```shell
$ npm i -D @hxjs/guards
```

## En garde

The package provides a set of ready-to-use type predicates (aka type guards) for primitive types, and a few macros for
composing new ones.

```typescript
import g from '@hxjs/guards'

type Players = Array<{
  name:  string
  score: number
}>

const isPlayers = g.arrayOf(g.struct({
  name:  g.string,
  score: g.number
}))

async function getPlayers() {
  const players = await (await fetch('/players').json())
  if (isPlayers(players)) {
    // TypeScript will treat players as type Players in this block.
    players.forEach(({name, score}) => console.log(`Player ${name} has a score of ${score}.`))
  }
}
```

Check [the code](src/guards.ts) for a complete list of packaged predicates for simple types.

### Literal types

```typescript
type Status    = 'open' | 'closed' | null
const isStatus = g('open', 'closed', null)
```

### Unions

All included and generated predicate functions are decorated with `and`, `or`, and `optional` modifiers.

```typescript
type NameOrNames    = string | string[]
const isNameOrNames = g.string.or(g.arrayOf(g.string))

type HasName  = { name: NameOrNames }
type HasScore = { score: number }
type Player   = HasName & HasScore

const isHasName  = g.struct({name: isNameOrNames})
const isHasScore = g.struct({name: g.number})
const isPlayer   = isHasName.and(isHasScore)

type Winner    = Player | undefined
const isWinner = isPlayer.optional
```

### Tuples

```typescript
type Pair    = [string, number]
const isPair = g.tupleOf(g.string, g.number)
```

### Arrays

```typescript
type Names    = string[]
const isNames = g.arrayOf(g.string)
```

There's also `array`, for brevity:

```typescript
type Sequences   = Array<unknown[]>
const isSequence = g.arrayOf(g.array)
```

### Objects

An object is defined by this package as an `object` with `Object` as its constructor.

```typescript
g.object({}) // => true
g.object([]) // => false
```

Like `arrayOf`, `objectOf` checks object value:

```typescript
type ScoresByName    = Record<string, number>
const isScoresByName = g.objectOf(g.number)
```

### Structs

A struct checks each key of an object, usually defined by an `interface` (see [main example](#en-garde)).

#### Required keys

By default, all keys in a struct are required. You can make only some keys required by including the `required` option.

```typescript
interface Player {
  name:   string
  score?: number
}

const isPlayers = g.struct({
  name:  g.string,
  score: g.number
}, {
  required: ['name']
})
```

> This is different to the `optional` modifier, which unions a type with `| undefined`.

#### Additional keys

By default, additional keys are allowed in a struct with any type. You can limit additional keys to a specific type by
including the `additional` option. To forbid additional keys, use the `never` type.

```typescript
const isPlayers = g.struct({
  name:  g.string,
  score: g.number
}, {
  required:   ['name'],
  additional: g.never
})
```

### Hinting

Though not necessary, hinting a complex predicate's type can make it easier in your editor to define inner predicates.

```typescript
interface Player {
  name:  string
  score: number
}

const isPlayers = g.struct<Player>({
  name:  g.string, // IDEs will hint as you type "name", and warn if the given predicate does not assert a string.
  score: g.number
})
```

When you guard a value with this predicate, your IDE will tell you it has the `Player` type, rather than the composed
`{name: string, score: number}` type that it infers from the predicate.
