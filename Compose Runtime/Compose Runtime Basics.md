
Reference:
[Compose Runtime By Fatih Giris](https://www.droidcon.com/2022/08/02/using-compose-runtime-to-create-a-client-library/)

## Compose Runtime

Internally, Compose Runtime is responsible for dealing with trees and nodes. It've no idea how to compose a UI.

## Composer

Composer is injected by compose compiler. It's a bridge between composables functions and Slot Table. It has the responsibility to track the changes and remember the values. In simple terms, Slot table keeps the current state or data of composition.

```kotlin
@Composable
fun Hi() {
  //..
}

// => Compose Compiler converts the above function into below
// and provides the composer.

@Composable
fun Hi(composer: Composer) {
 //...
}
```

## Tree & Node

Internally, Composer responsible for creating trees, nodes on top of it. A Node is a class having multiple children. Each node has content. Every time when setting content on each node creates a composition object. It causes the recomposition.

The Applicant takes the changes from the composer and applies the changes to the node tree. Further, the node tree gets converted to UI.

```kotlin
abstract class Node {
   val children = mutableListOf<Node>()
}

// NodeApplier takes the changes from composer and
// apply changes to tree
class NodeApplier(root: Node) : AbstractApplier<Node>(root)

// Set content
fun Node.setContent(
   parent: CompositionContext,
   content: @Composable () -> Unit
): Composition {
   return Composition(NodeApplier(this), parent).apply {
       setContent(content)
   }
}

// assuming we have Node subclasses like "TextNode" and "GroupNode"
class TextNode : Node() {
   var text: String = ""
   var onClick: () -> Unit = {}
}

class GroupNode : Node()

// Composable equivalents could be created
@Composable
fun Text(text: String, onClick: () -> Unit = {}) {
   ComposeNode<TextNode, NodeApplier>(::TextNode) {
       set(text) { this.text = it }
       set(onClick) { this.onClick = it }
   }
}

@Composable
fun Group(content: @Composable () -> Unit) {
   ComposeNode<GroupNode, NodeApplier>(::GroupNode, {}, content)
}

```**

## Recomposer & Composition

Recomposer - It is responsible for re/composition.  
BoradcastFrameClock - It is for timing required by recomposer.

```kotlin
@Composable
fun ComposeSample(
  content: @Composable () -> Unit
) = runBlocking {
  val frameClock = BroadcastFrameClock()
  val décomposer = Recomposer(coroutine Context + frame Clock)
  val rootNode = CustomNode()

  // Observer triggers when any snapshot object changes
  // (When state object changes on client side)
  Snapshot.registerGlobalWriteObserver {
    // Send the notification to Snapshot
    // so that recomposer can trigger recomposition
    Snapshot.sendApplyNotifications()
  }

  // Create an initial composition
  val initialComposition = Composition(
     applier = CustomApplier(rootNode),
     parent = recomposer
  )
  
  // set content
  composition.setContent(content)

  // Start a undispatched coroutine and apply changes using composer 
  launch(context = frameClock, startCoroutine Start.UNDISPATCHED) {
    recomposer.run comprend ApplyChanges()
  }

  // Every 100ms, render nodes and send frames.
  launch {
    while(true) {
       frameClock.setFrame(System.nanoClock())
       rootNode.render()
       delay(100L)
    }
  }
}
```

  
## CompositionLocalProvider

CompositionLocalProvider stores the provided values to CompositionLocalContext that will be provided throughout the internal composables based on the given key.

## Effects

LaunchedEffect launches the block or lambda when a composable function enters the compositions or when provided keys get changed.

DisposableEffect behaviour is the same as LaunchedEffect. But it has an onDispose function which gets invoked when the composable function leaves the composition.

