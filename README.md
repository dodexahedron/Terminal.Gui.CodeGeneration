# Terminal.Gui.CodeGeneration

This project contains code generation functionality intended for use by the Terminal.Gui project, to lighten the workload for contributors while also providing implementations of code that is 
consistent, reliable, robust, convenient, high-performance, or any useful combination of those, especially where doing it manually is daunting or possibly even simply not worth the time and effort for manual implementation, even if otherwise beneficial.

# Goals

Initial work will be targeted at generation of code related to hot and often expensive operations involving enum types declared in the Terminal.Gui assembly, and to provide some
consistency across enums decorated with the appropriate attribute.

More specifically, and in rough and by no means guaranteed order, initial intentions are to implement the following, all in the context of enums, and  any and all of which are subject to change, cancellation, etc...\
 - Alternatives to various built-in operations, with the generated alternatives having lower run-time CPU cost, typically at the expense of a slightly larger compiled assembly.
 - Avoiding heap interactions, especially unnecessary allocations and boxing of value types, as much as possible.
 - Attempting, within reason, to reduce or eliminate string manipulation and allocations when converting to or from a standard string representation of the enum.
 - When string-related operations are performed, prefer Span-based methods.
 - Use hard-coded constructs with linear or better runtime and minimal or no heap allocations for string-enum conversions, such as switch expressions, whenever possible.
 - Formal `readonly record struct` types which are convertible to and from their associated enum types, with additional capabilities which would be difficult, expensive, or even impossible to implement as extension methods for the enum type itself.
   - These will ideally include things like cast-free operators for equality and comparison of enum members with their underlying types, for both the implicit (enum -> underlying type) and explicit (underlying type -> enum) cases, along with interfaces that those operators implicitly implement.
     - Potentially also provide those operators for other compatible types (by size - not represented value), as well, especially when the numeric value serves multiple purposes (such as with KeyCode), where sign is irrelevant.
 - Ability to mark individual enum members for exclusion from generated code elements (probably a future enhancement unless I end up needing it right away).
 - Implement those structs in a way that enables them to be essentially drop-in compatible with the enum types they are based on, while also enabling more efficient, more expressive, and more intuitive use.
   - Examples:
     - Eliminating the need for manual bitwise operations anywhere they're currently needed because of enum use.
     - Constant time replacements for methods like Enum.IsDefined, for significantly faster operations involving or dependent on validation of numeric values
     - Exposing flag values (even those in non-Flags enums, such as in KeyCode) as actual booleans, backed by a BitVector32 or any other appropriate and more modern, flexible, and purpose-built value type.
       - That will probably require an explicit attribute to inform the source generator of which members are actually meant to be flags and the source generator should probably validate that they are exact powers of 2 and therefore single-bit values.
     - The ability to formally enforce mutual exclusions that may exist between certain flags and/or values.
     - Other things that enums just can't do since they can't have anything beyond constant numeric members.

Several enum types in the project already have both a helper type of some sort as well as extension methods, _anyway_, so why not just make the machine do the work? :)