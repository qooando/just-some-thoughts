# Dev Philosophy

## Main philosophies

### Principle of least surprise

From architecture to a single microservices or a single function, you SHOULD apply the principle of least surprise.
The code SHOULD behave as the most of the users think it behaves. From a developing point of view the users are you and
other developers, least surprise means lesser burdens adding a feature, debugging and fixing the code.

See also: [Principle of least astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)

### Twelve-Factor App

Especially for microservices, twelve factor app principles are useful to build interconnected but decoupled components.

See also: [12factor](https://12factor.net/) 

### Boring Software Manifesto 
Best projects doesn't require to solve seven crisis simultaneously. It's ok to use known and common techs to solve old 
and new problems. And the best thing is focusing energies on most interesting and valuable parts. The actual manifest is 
a bit more verbose, but the core is here. The best code, the most reliable code is the boring code, it should be OK to 
write it the first time and forget it forever, and spend energies on valuable parts.

See also: [the boring software manifesto](https://noelrappin.com/blog/2016/05/the-boring-software-manifesto/)

## Rules of thumb

This is a non-exhaustive list of rules of thumb is useful to follow during developing.

### Minimize cognitive load
The burden to understand old code is tech-debt.

Too much cognitive load is tech-debt.

How many time you need to understand the code and fix a bug? 

The gold standard of rules is to reduce at the bare minimum the cognitive load required to understand the code and how it works in every aspect. Most of the following rules are
just specialization of this one, proposed in order to minimize the overall cognitive load. 

### Rule of seven 
The mind of an average human is capable to take in account up to seven "things" at a time, this means we must take in account this
number when we develop the code. More things we need to know or understand while writing a new feature or debugging
something, more the cognitive load is high and we lose efficacy.

How many things we need to take in account?

* every nested function call
* input parameters
* every branch
* output format

PLEASE design and write the code to be simple and to have the minimum required *jumps* and objects.

### Declare your intents

Before write the actual code, you SHOULD declare IN ADVANCE your intents about what the function should do.

```python
class UserRankingService:
  def evluateUserRankWithAlgorithm(user: UserInfo, rankAlgorithm: str):
    """
    this function must evaluate the user rank, given the algorithm
    1. get the intrinsic user measure
    2. check all connections of the user with other users and their weights
    3. check how many times it is cited
    4. apply the ranking formula
    5. return the result
    """
    ...
```

### Maintainability above efficiency

Most of the time efficiency doesn't mean better code, especially if the code has no strict time constraints.

Obviously favor efficiency, but if it means writing obscure black magic code and too much lines to comment how it works,
maybe something is wrong.

### Purgatory of code replication

Code replication may save you future burdens when you need to customize the code only for a specific case.

Furthermore code replication may be useful to maintain a function simple
and linear and reduce the nested function calls. This is usually the case with delegation pattern, sometimes it is
better to replicate same code in every delegate for sake of readability.

Code replication may save us from
uncontrolled proliferation of utility classes and methods with small and difficult to track differences.

Please don't generalize everything just to avoid a small code replication.

### Code verbosity

BE VERBOSE with class names, function names, variable names SHOULD be self-explanatory.

> Use short or abbreviated name ONLY IF THE CONTEXT IS OBVIOUS AND EXTREMELY LIMITED
> (e.g. a for loop with only 4 lines of code)

### Fluent notation for building, processing, and filtering

If available, favor fluent notation to create new objects, process and filter lists and maps.

```java 
item = MyItem.builder()
    .name("Item1") 
    .description("This item is annotated with lombok @Builder") 
    .build(); 
itemDto = MyItemDto.builder() 
    .name(item.getName()) 
    .description(item.getDescription())
    .build();
```

e.g. filter and processing

```java 
ranks.stream()
    .map(x -> { return x + 1; } 
    .filter(x -> x > 5) 
    .toList();
```

### Avoid side effects 
PLEASE write functions without side effects, as side effects we can list 
* change a flag/variables somewhere in another part of the business logic 
* change something in input arguments 
* other subtle behavior changes

> Exception can be made if we are operating on a contextual object containing 
> all processing variables, the context itself is both input and output 
> of the function

```python
def processItems(ctx: ProcessingContext):
  items = ctx.items
  result = list(map(lambda x: x+1, items))
  ctx.processed = result
  return ctx
```

### Fail fast 
PLEASE check failure conditions AS SOON AS POSSIBLE,
thus is easier to check function failing condition simply by reading the
function code. e.g.

```python
def processedItems(ctx):
  if not ctx.items:
    raise Error("no items array provided in context")
  ...
```

### Validate function parameters and output from other functions 
PLEASE remember to validate input parameters at the beginning of a function, plus validate output of nested calls.

```python
def sum(a, b):
  if a is None:
    raise Error("Invlid argument 'a'")
  if b is None:
    raise Error("Invlid argument 'b'")
  return a + b
```

### Hide complexity in common library 
Sometimes complexity is unavoidable, e.g. when implementing specific patterns. 
In these cases is useful to box the complexity within a single class, module,
library, dependency and expose a simple and self-explanatory api. 
As long as the complexity is boxed in easy to identify black box, 
it counts only as "one thing that just works" leaving the rest of the code 
more cleaner. 

### Structure by data lifecycle and user workflows 
Structure the codebase and software architecture to mimic the data lifecycle
and user workflows. Put together classes related to a single workflow. 

E.g. by foo and bar lifecycle 
```
package.
  foo.
    FooConfig
    FooController
    FooDto
    FooModel
    FooConverter
    FooProcessor
  bar.
    BarConfig
    BarController
    BarModel
    ...
```

### Resources as sync apis, commands as async events
If something is a *resource *then expose it as synchronous REST API. E.g. a list of users. 

See also: [naming convention](https://restfulapi.net/resource-naming/), [best practices](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api). 

If something is an *event*, a *command* of some kind, please use asynchronous queues (e.g. kafka, rabbitmq). 
You can convert events to rest api as if they are *resources* insead of *actions*, e.g. adding a Request resource and asking for a Response resouse. ### Small and documented workarounds It is
common to add workarounds for specific cases when the deadline is near. 

Please always write small workarounds with the
bare minimum impact and document them extensively in a comment. 
Always add a FIXME tag to mark them as something to be
fixed in the future. e.g.

```java 
def processItems(ctx):
  # FIXME temporary workaround, sometimes the context
  # has null items maybe it is a race condition but we need more
  # investingations for now just set items field if missing
  # instead to throw an error 
  if not ctx.items:
    ctx.items = []
    # raise Error("not items")
  ...
```

### String IDs
If you can, use String type for IDs. String type automatically supports future changes in id format 
(e.g. UUID, BSON, …). 

Please avoid custom types specific to a single database application (e.g. ObjectId), they bind you to use that software
forever or to write risky migrations. 

## Patterns

### Microservices 
Please split business logic in different microservices. 

Split choice should take in account following variables: 
* User workflow and data lifecycle (who acquire the data, how we need to process it, where we put the output,
  which actions and use cases the service should support) 
* Data model (who owns the data, which business logic unit has the authority over those data?) 
* Scalability (which business logic units must be replicated and parallelized independently by other ones?)
* Who manage the code 

### Dependency Injection 
Please leverage dependency injection for bigger services, like the ones in Java Spring Boot. 
This is usually not required for smaller services, like simple flask python services.

### Delegation pattern 
Please leverage the delegation pattern to generalize processing of families of similar artifacts/input data. 
The pattern permits to avoid infinite if-else/switch statements and make the code clearer and more modular. 
If you can design input metadata, please add an apiVersion string field or something similar. 
This field will be used to select the correct delegate which process the input. 
Please avoid to use more than one string field, one string field usually is enough, more fields add complexity. 
Instead to add more fields, use a string nesting format e.g. apiVersion: "group.subgroup.type". 

See also: [delegation pattern](https://en.wikipedia.org/wiki/Delegation_pattern) 

### Converters 
If you can leverage dependency injection, implement converters between models as singletons, 
in this way you can inject them when necessary and they can also access other dependencies. 

> If you implement converters as static functions you limit your conversion domain to the input itself, 
> instead as dependency you can add nested dependencies and other services to enrich the output. 
> If available, leverage libraries with json mapping and transformation, but please use caution with automapping
> libraries, they may hide fallpits difficult to debug (e.g. wrong mapped fields)

### Data model 
A proper data model should be provided. 

For every data object we need to know at least the following info: 
* who can read it 
* who can write it 
* which component has the ownership and it's authoritative on the data 

Then the code should enforce permissions both at query level (to avoid to propagate non-visible data through the 
business logic) and at endpoint level. Eventually security should be also enforced as an aspect of critical business
logic functions.
