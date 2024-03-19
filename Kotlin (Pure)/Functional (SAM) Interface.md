SAM stands for Single Abstract Method.
Here, Abstract method - method without body declared in interface.

```kotlin
fun interface IntPredicate { 
  fun invoke(): Boolean
}
```

SAM interface conversion that helps the code to be more readable by using lambda expressions.

```kotlin
val isEven = IntPredicate { it % 2 == 0 }
```

