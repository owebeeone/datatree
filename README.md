## *datatree* vs *dataclass* Decorators

# Introduction

[Datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) complements Python [dataclasses](https://docs.python.org/3/library/dataclasses.html). Dataclasses was introduced in the Python 3.7 standard distribution and greatly reduces the amount of boilerplate code when coding “data classes”. The question answered here is, when would one choose [datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) vs vanilla [dataclasses](https://docs.python.org/3/library/dataclasses.html). In short, since [datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) provides a superset of [dataclass](https://docs.python.org/3/library/dataclasses.html) functionality, one could use [datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) anywhere one would use [dataclass](https://docs.python.org/3/library/dataclasses.html). [Datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) introduces field injection and binding with some support for overriding field values and field level documentation support.

The impetus behind [datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) stems from building Python classes that represent 3D models using AnchorSCAD. Using [dataclasses](https://docs.python.org/3/library/dataclasses.html) greatly simplifies this; however, when designing complex composite hierarchical shapes, [dataclasses](https://docs.python.org/3/library/dataclasses.html) do not provide relief from boilerplate overload when many fields are needed when composing a class from many other dataclass objects.

[Datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py), like [Python’s dataclass](https://docs.python.org/3/library/dataclasses.html), provides for a less verbose approach to creating ‘data class’ like objects composed of other data class like objects. In fact, [datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) is a wrapper over [dataclass](https://docs.python.org/3/library/dataclasses.html) and uses all the same arguments but also provides a mechanism to inject annotated variables from other classes and tools to bind the variables when constructing the injected class’s objects.

# Example

For example, when building a composite of other ‘dataclass’ objects, the parameters may need to be reflected in the composite object. Assume we start with two dataclasses shown below.

```
from dataclasses import dataclass

@dataclass
class Thing():
    v1: float=1.0
    v2: float=2.0
    def dothing(self):
        return self.v1 + self.v2
    
@dataclass
class ThatOtherThing():
    v2: float=2.0
    v3: float=3.0
    def dothing(self):
        return self.v2 + self.v3
```

Composing from the classes above one could write the following code:

```
@dataclass
class Things():
    v1: float=1.5
    v2: float=2.0
    v3: float=3.0 
    def dothings(self):
        thing = Thing(v1=self.v1, v2=self.v2)
        that_other_thing = ThatOtherThing(v2=self.v2, v3=self.v3)
        return thing, that_other_thing

print(Things(v2=7).dothings())
```

Note in this example the repetition of v1, v2 and v3 in class AB with a potentially problematic difference in the default value of v1. Notably, any values added to class Thing that need to be reflected in Things will need to be done manually.

The example below uses datatree.

```
from anchorscad.datatree import datatree, Node

@datatree
class Things():
    v1: float=1.5
    nodeThing: Node=Node(Thing)
    nodeThatOtherThing: Node=Node(ThatOtherThing)
    def dothings(self):
        thing = self.nodeThing()
        that_other_thing = self.nodeThatOtherThing()
        return thing, that_other_thing

print(Things(v2=7).dothings())
```

Note the only specified value field required is v1 because it has a different default value. Both the examples above print:

```(Thing(v1=1.5, v2=7), ThatOtherThing(v2=7, v3=3.0))```

Note the sharing of v2, if the injected class fields have different default values, the first default value encountered is applied.

# Advantages

[Datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) eliminates significant portions of otherwise duplicated/boilerplate code in classes with a large number of fields. This enables breaking up complex hierarchy into smaller components without replicating field definitions between classes. More importantly, if the field definitions of the injected classes change, they are automatically reflected in the composing class reducing the overall maintenance load.

In practice, the ability to compose complex 3D shapes from many other ‘component’ shape classes is particularly useful when creating complex 3D shapes where many components share many parameters.

dataree.Node also allows the mapping of field names when they are injected. They can be mapped to a different name by:

* listing the names to be injected.
* passing dictionaries with the name mappings.
* or a “prefix” or “suffix” can be applied to other fields.
* or a collection of field names to preserve if they exist.

[datatree](https://bitbucket.org/owebeeone/anchorscad/src/master/src/anchorscad/datatree.py) can also bind functions instead of classes in much the same way as classes.
