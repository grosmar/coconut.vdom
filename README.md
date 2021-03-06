# Coconut VDOM Renderer

This library provides [`js-virtual-dom`](https://github.com/back2dos/js-virtual-dom) based renderering to coconut.

## Adding a View to the DOM

Once you have a view, you can render it in the DOM using `parent.appendChild(theView.toElement())`. Simples.

## Element Ownership

Every view "owns" the `js.html.Element` that it last rendered to. Note that the element may have to be swapped, e.g. if its type changes, e.g. imagine the following:

```haxe
class Button extends coconut.ui.View<{ onclick:Void->Void, disabled:Bool, label:String }> {
  function render() '
    <if {disabled}>
      <span>{label}</span>
    <else>
      <button onclick={onclick}>{label}</button>
    </if>
  '
}
```

Disregarding the fact that this is would make for a hot contender in a competition for finding the stupidest way to render buttons, it's important to understand that if you create an enabled button and then disable it, the `span` is replaced by a `button` in the actual dom. Performance wise this *may* have quite far reaching implications. It is therefore suggested to **always wrap your views**, e.g.:

```haxe
class Button extends coconut.ui.View<{ onclick:Void->Void, disabled:Bool, label:String }> {
  function render() '
    <div class="button">
      <if {disabled}>
        <span>{label}</span>
      <else>
        <button onclick={onclick}>{label}</button>
      </if>      
    </div>
  ';
}
```

That said, you should *never* rely on a views element staying the same, certainly not from outside the view, but also not from the inside.

## View Life Cycle

Every view has the following life cycle hooks that you may override:

```haxe
function beforeInit():Void;
function afterInit(element:Element):Void;
function beforePatching(element:Element):Void;
function afterPatching(element:Element):Void;
function beforeDestroy(element:Element):Void;
function afterDestroy(element:Element):Void;
```

Views are initialized in two cases:

1. You call their `toElement` method.
2. When they are rendered for the first time by a parent view.

Once a view is initialized, it begins to observe the data it is meant to render. Should that data change, a rerender is scheduled. Rerendering entails:

1. Rendering the new data as VDOM and calculating the diff from the previous render.
2. Calling `beforePatch` with the current DOM element.
3. Applying the patch.
4. Calling `afterPatch` with the updated DOM element.

Note that changes to the DOM may lead to the strangest effects. Perform them at your own risk. The before/after patching methods are primarily meant for dealing with scroll position, focus and similar state that is not reflected in the DOM structure directly.

## Event Handling

Event handling in `coconut.vdom` is fully typed, meaning that the event type and the event target are what you'd actually expect them to be:

```haxe
class Input extends View<{ value:String, onchange:String->Void }>{
  function render() '
    <input type="text" value={value} onchange={onchange(event.target.value)} />
  ';
}
```

# Caveats

Here's the shortlist of things to keep in mind:

1. A view's element may change during its life cycle
2. A view may be replaced by an otherwise completely equal view
