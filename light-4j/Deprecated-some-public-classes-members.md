# Deprecated some public APIs and classes members

I found the access level of member in classes were not properly set, based on [Alibaba-Java-Coding-Guidelines](https://github.com/alibaba/Alibaba-Java-Coding-Guidelines):

> We should strictly control the access for any classes, methods, arguments and variables. Loose access control causes harmful coupling of modules. Imagine the following situations. For a private class member, we can remove it as soon as we want. However, when it comes to a public class member, we have to think twice before any updates happen to it.

But since it's our framework is already used by many projects, it is very dangerous to change  access modifiers of classes, properties, methods and etc. We need to follow by some rules to minimize the risk:

1. Make public modifier to be *Deprecated*, and add Javadoc to mark it as *@deprecated*.
2. Remain those class members realted to feature configs as it was, such as "ENABLE_VERIFY_JWT", those configs are very likely be used in other projects.
3. Package-private level classes members should be safe to be changed based on reference situations:
    1. If the usages are only in it's own class, change it to private.
    2. If the usages happens in tests, either provide a getter method or access it using reflection, or just remain as it is.
    3. if it's also referenced by other classes in same module, keep as it is.