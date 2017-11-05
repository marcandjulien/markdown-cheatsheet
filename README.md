# OCL cheatsheet
January 2017 - Montreal<br/>
Simple memo for OCL


# Introduction
> Object Constraint Language (OCL)
> Developed by IBM in 1995 <br/>
> Standardised by OMG in 1997 <br/>
> OCL 2.0 defined in UML 2.0 <br/>
> Allows to add extra information about a model (Like UML diagram) <br/>


# OCL basics
### OCL Constraints
- **inv** invariant: constraint must be right at all time
```
context Employee 
    inv: self.age > 18
```

- **init**: specify an initial value for an attribute
```
context Employee::wage
	init: age = 18
```

- **derive**: specify the calculation rule for a derived attribute
```
context Employee::wage
	derive : wage = self.age * 50
```

### Data types
> All types inherit from OclAny (Like the class Object in Java) <br/>
> OclVoid can inherit from any type. Its unique instance is null.

- Primitive types (From UML): Boolean / Integer / Real / String
- Collection types: Collection / Set / Bag / Sequence
- User defined types

##### Example for enum type
> Let's have Gender enum with values "male" and "female"

```
context Customer inv:
	self.gender = Gender::male
        implies self.title = 'Mr.'
```


# Collections
> Use `->` to apply operations on collections. Example: `self.participant->size() <= 25`

### Collections types
- **Set**: mathematics ensemble without duplicates
```
Set{1,2,5,88}
Set{'apple',''orange','strawberry'}
```

- **OrderedSet**: ordered 'set'

- **Bag** is a multi-ensemble (Can have duplicates)
```
Bag{1,3,45,2,3}
Bag{1,2,3,3,45}
//These 2 Bags are identical
```

- **Sequence**: is an ordered Bag
```
Sequence{1,3,45,2,3}
Sequence{1..10}
```

### Operations on collections
| Function | Description |
| --- | --- |
| size() | |
| isEmpty() | |
| notEmpty() | |
| count(Object) | The number of apparitions of an object in the collection |
| includes(Object) | True if the object is an element of this collection |
| includesAll(Collection) | True if collection is a sub-ensemble of current collection |
| any(bool expr) | Randomly choose an object that satisfy this expression |
| sum() | Calculate the sum of all elements for a collection of numbers |
| sortedBy(expr) | Create an OrderedSet(Sequence) with all original elements from the Set(Bag) according to the expression |
| = | True if all elements from both collections are same. Bag: nb occurrences for each element must be same. Ordered: order must be same. |
| union(Collection) |  |
| intersection(Collection) |  |
| symmetricDifference(Collection) | Only for Set |
| including(Object) | Add object in collection. Set: add if not already in collection. Sequence: equivalents to append(Object) |
| append(Object), prepend(Object) | Add element at the end / beginning of the collection |
| excluding(Object) | Remove every occurrence of Object |
| first(), last() | Return first / last element (Only for ordered collections) |
| at(int) | Index start at 1 |

### Examples of collection use
```
context Company inv:
    if self.budget < 50000
    then self.employees->size() < 31
    else true 
    endif

```


# Iteration on collections
> iterate(exp) apply expression on each element and put it into a Bag (Or sequence)
```
context Collection(T)::sum() : T
post: result = self->iterate( element : T;total : T = 0 total + element)
```

### Operations with iterator
| Function | Description |
| --- | --- |
| exists(bool expr) | True if expression true for at least one element |
| one(bool expr) | True if expression true for exactly one element |
| forAll(bool expr) | True if expression true for all elements |
| select(bool expr) | Choose all elements that match expression |
| reject(bool expr) | Exclude all elements that match expression |
| collect(expr) | Calculate expression for each element and put it on a Bag (Or Sequence) |
| isUnique(expr) | True if the expression result is unique for each element |

### Examples with iterator
```
context Company inv:
    self.employees->select(age < 18)->isEmpty()

context Company inv:
    self.employees->reject(age>=18)->isEmpty()
    
context Company inv:
    self.employees->collect(wage)->sum() < self.budget
    
context Company inv:
    self.employees->forAll(age > 18)

context Company inv:
    self.employees->exists(e|e.wage=20000)

```


# OCL Data Type checking
### List functions
| Function | Description |
| --- | --- |
| oclType() | Return object type |
| oclIsKindOf(OclType) | True if object type is OclType or sub class |
| oclIsTypeOf(OclType) | Like oclIsKindOf but no sub class |
| oclAsType(OclType) | Cast |
| allInstances() | Return ensemble of all class instances (To avoid because resources expensive) |

### Data Type checking Examples
```
context Employee inv:
    Employee.allInstances()->forAll(p1, p2 | p1 <> p2 implies p1.name <> p2.name)

```


# Other useful things

- **closure** recursive operation
```
parent.closure(children)
```
> Return a collection including all children, children of children etc...

- **let**: create sub-expression used more than once in the same constraint
```
context Person inv:
    let income = self.job.salary->sum() in
    if isUnemployed then
        income < 100
    else
        income >= 100
    endif
```

- **def**: variables or operation to use in several constraints
```
context Person
    def: income : Integer = self.job.salary->sum()
    inv: if isUnemployed then
        income < 100
    else
        income >= 100
    endif
    inv: taxes = income * 0.25
```

- **body**: specify request result
```
context Employee::getWage() : Integer
    body: self.wage
```

- **pre / post**: pre / post condition that must be true before / after operation execution
```
context Employee::raiseWage(newWage:Integer)
    pre: newWage > self.wage
    post: wage = newWage
    
context Person::getCurrentSpouse() : Person
    pre: self.isMarried = true
    body: self.marriages->select(m | m.ended = false ).spouse

context Person::isSenior() : Boolean
	pre: age >= 0
	post: age = age@pre
	body: age >= 65
    //Age is unchanged by function
```

- **@pre** refers to property value before operation


# OCL examples
```
context Customer::discount: Integer derive:
    if not self.premium then
        if self.rental.car.carGroup->
            select(c|c.category=‘high’)->size() >= 5
        then 15
        else 0 endif
    else 30 endif
```
> 30% discount for premium customer. <br/>
> 15% discount for non premium customer with at least 5 rented autos (With 'high' category) <br/>
> Otherwise, no discount

```
context Job inv:
    self.employee.age>18 implies 
        self.employer.isIncorporated = true
```
>If at least one employee is 18 or more, then company must be incorporated


# Resources
- [Eugene Syriani](http://www-ens.iro.umontreal.ca/~syriani/main.html)
- [IFT3911 lesson by Eugene Syriani - Slides about Constraints](http://www-ens.iro.umontreal.ca/~syriani/teaching.html)
