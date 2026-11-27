# How to automatically destroy related objects

1. Class A has `has_many` relationship to class B: `has_many: ClassB`

2. Objects of class B should be destroyed with object of class A.

3. Add `dependent:` option to `has_many` relationship: `has_many: ClassB, dependent: :destroy`
