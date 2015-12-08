Number (with) Unit
==================

[![build status](https://api.travis-ci.org/jprichardson/number-unit.svg)](http://travis-ci.org/jprichardson/number-unit)

A [heavily tested](https://github.com/jprichardson/number-unit/tree/master/tests) JavaScript component to handle arbitrary precision numbers with units.


Why?
----

A number without context (unit) is meaningless. Even more so in the realm of
computers. Computers rely on input, usually from humans, to be precise. Humans are prone to error.
This library helps to remove that error.

Consider these two famous incidents...

1. **Mars Surveyor '98 Orbiter**

From https://en.wikipedia.org/wiki/Mars_Climate_Orbiter:

> However, on September 23, 1999, communication with the spacecraft was lost as the spacecraft went into orbital insertion, due to ground-based computer software which produced output in non-SI units of pound-seconds (lbf s) instead of the metric units of newton-seconds (N s) specified in the contract between NASA and Lockheed. The spacecraft encountered Mars on a trajectory that brought it too close to the planet, causing it to pass through the upper atmosphere and disintegrate.

2. **Air Canada Flight 143**

From https://en.wikipedia.org/wiki/Gimli_Glider:

> ...ran out of fuel at an altitude of 12,500 metres (41,000 ft) MSL, about halfway through its flight originating in Montreal to Edmonton. The crew were able to glide the aircraft safely to an emergency landing at Gimli Industrial Park Airport, a former Royal Canadian Air Force base in Gimli, Manitoba.[1]

> The subsequent investigation revealed a combination of company failures and a chain of human errors that defeated built-in safeguards. Fuel loading was miscalculated because of a misunderstanding of the recently adopted metric system which replaced the imperial system.

---

If you're writing software that **handles people's money**, you can't afford to be wrong. **That's why
this library was built.**


Install
-------

    npm i --save number-unit


Usage
-----

**Quick example:**

```js
// import { UnitType } from 'number-unit' // if using ES6 (ES2015)
var UnitType = require('number-unit').UnitType

// create a UnitType first
var bitcoin = UnitType.create('bitcoin', { satoshis: 1, bits: 1e2, BTC: 1e8 }, 'bits')

// now create a NumberUnit
var amount1 = bitcoin.BTC(1.53)
var amount2 = bitcoin.bits('1530000') // notice, can accept strings as well

console.log(amount1.toString()) // => 1.53 BTC
console.log(amount2.toString()) // => 1530000 bits

// compare numerical values
console.log(amount1.equals(amount2)) // => true
```



### Important Concepts to Know

1. You must use `UnitType.create()` to create a `UnitType()` to start working with
NumberUnits.

2. UnitTypes can be a type of other UnitTypes. For example, you may create
a `UnitType` named `distance`, and then want to create another named `distanceSI`
representing your desire to model [SI / Metric Units](https://en.wikipedia.org/wiki/International_System_of_Units).
Now you may want to create another named `distanceUS`, modeling
[United States customary units](https://en.wikipedia.org/wiki/United_States_customary_units). Since both
have `distance` has a parent type, you can convert between the two. This is the value
of parent types. As it wouldn't make sense to convert from distance to currency or something
lie that. See for some examples: https://github.com/jprichardson/number-unit


### UnitType

#### UnitType.create()

Method signature: `UnitType.create(label, [parentUnitType], [definitions], [defaultUnit])`

Creates an instance of `UnitType` and returns it.

- `label`: The unit type label.
- `parentUnitType`: The parent unit type. Useful for converting between UnitType that have the same parent.
- `definitions`: Actual conversions.
- `defaultUnit`: Default unit. Used when `defaultUnit` is called.

```js
var UnitType = require('number-unit').UnitType
var bitcoin = UnitType.create('bitcoin', { satoshis: 1, bits: 1e2, BTC: 1e8 }, 'bits')
```


#### UnitType.prototype.parse()

Method signature: `parse(string)`

Method that parses the input string and returns an instance of `NumberUnit` with
a number value extracted from the string and a `unitName` from the string.

```js
var amount = bitcoin.parse('1.53 BTC')
console.log(amount.toNumber()) // => 1.53
console.log(amount.unitName) // => BTC
```


#### UnitType.prototype.ZERO

Property that creates and returns an instance of `NumberUnit` with a number value of `0` and a
the unit being the default unit.

```js
var zero = bitcoin.ZERO
console.log(amount.toNumber()) // => 0
console.log(amount.unitName) // => bits
```



### NumberUnit

Create instances of `NumberUnit` with the UnitType factory methods.

**Note:** All methods on `NumberUnit` instances return new instances of `NumberUnit`,
that is, NumberUnits are immutable.

```js
var amount = bitcoin.BTC(3.5)
console.log(amount instanceof NumberUnit) // => true
```


#### NumberUnit.prototype.abs()

**Signature:** `abs()`

**Parameters:** (none)

**Returns:** a new instance of `NumberUnit` with the absolute value of the number.

**Example:**

```js
var amount = bitcoin.BTC(-3.5)
console.log(amount.abs().toString()) // => 3.5 BTC
```


#### NumberUnit.prototype.add()

**Signature:** `add(number)`

**Parameters:**
- `number`: Can either by of type `number` or `NumberUnit`. If `strict` mode is set
it can only be of type `NumberUnit`.

**Returns:** a new instance of `NumberUnit` that represents the sum of the two numbers.

**Example:**

```js
var amount = bitcoin.BTC(-3.5)
var sum = amount.add(bitcoin.bits('1000000'))
console.log(sum.toString()) // => -2.5 BTC
```


#### NumberUnit.prototype.clone()

**Signature:** `clone()`

**Parameters:** (none)

**Returns:** New instance of `NumberUnit` that is a clone.

**Example:**

```js
var amount = bitcoin.BTC(1.3)
var amount2 = amount.clone()
console.log(amount2.toString()) // => 1.3 BTC
```


#### NumberUnit.prototype.equals()

**Signature:** `equals(numberUnit)`

**Parameters:**
- `numberUnit`: Another of instance of a `NumberUnit` to check equality with. Units
do not matter. e.g. `1000 m = 1 km`.

**Returns:** A `boolean`, `true` if equals.

**Example:**

```js
var distance1 = distanceSI.km(1)
var distance2 = distanceSI.m(1000)

console.log(distance1.equals(distance2)) // => true
```


#### NumberUnit.prototype.gt()

Check if one is greater than the other.

**Signature:** `gt(numberUnit)`
**Parameters:**
- `numberUnit`: Another of instance of a `NumberUnit`.
**Returns:** A `boolean`, `true` if it's greater than.
**Example:**

```js
var distance1 = distanceSI.km(1.1)
var distance2 = distanceSI.m(1000)

console.log(distance1.gt(distance2)) // => true
```


#### NumberUnit.prototype.negate()

Negate the number.

**Signature:** `negate()`
**Parameters:** (none)
**Returns:** New instance of `NumberUnit` with the number negated.
**Example:**

```js
var distance1 = distanceSI.km(1)
var distance2 = distance1.negate()

console.log(distance2.toString()) // => '-1 km'
```


#### NumberUnit.prototype.subtract()

**Signature:** `subtract(numberUnit)`
**Parameters:**
- `numberUnit`: Another of instance of a `NumberUnit`.
**Returns:** An instance of NumberUnit that represents the subtraction of the two.
**Example:**

```js
var distance1 = distanceSI.km(3)
var distance2 = distanceSI.m(1000)
var distance3 = distance1.subtract(distance2)

console.log(distance3.toString()) // => '2 km'
```




### Unit





### ConversionUnit
