---
layout: post
title: Reusing Templates with Custom Angular Components
cover: /assets/img/reusing-templates-with-custom-angular-components/cover.jpg
---

In this short tutorial, we’ll look at how to reuse content children passed between custom component tags.

## Identifying the problem

Recently, while working with on an Ionic project using Angular, I had to refactor some code into a reusable component. It was a button that was shown at the top and bottom of some scrolling content

The same structure was used on different screens, but with different buttons. This might make more sense if I drop a sample of what I was working with

```html
<div>
    <div>
    <button>Action</button>
    </div>
    <!-- scrolling content here -->
    <div>
    <button>Action</button>
    </div>
</div>
```

I decided to factor out everything, but wanted to let the user of the component define the action button and its styling. Essentially, I tried the below.

```html
<my-navigation>
      <button>Home</button>
</my-navigation>
```

Where `my-navigation` is defined as follows:

```html
<div>
    <div>
    <ng-content></ng-content>
    </div>
    <!-- scrolling content here -->
    <div>
    <ng-content></ng-content>
    </div>
</div>
```

Unfortunately, this does not work. Angular only renders the last used `ng-content` in any component. After some researching, I stumbled upon this video on YouTube

<iframe width="560" height="315" src="https://www.youtube.com/embed/2SnVxPeJdwE?controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Using ngTemplateOutlet

The video introduced me to ngTemplateOutlet, which helps solve the problem. It need the my-navigation component to be refactored as follows:

```html
<div>
  <div>
    <ng-container [ngTemplateOutlet]="action"></ng-container>
  </div>
  <!-- scrolling content here -->
  <div>
    <ng-container [ngTemplateOutlet]="action"></ng-container>
  </div>
</div>
```

This states that this component expects a named template. In this case, the name of the template was action. This also means that we need to add a property to the typescript code as follows:

```typescript
@Component({...})
export class NavigationComponent {
    @ContentChild("action") action: TemplateRef<any>;
}
```

This property will search for any `ngTemplateOutlets` with the name action and assign it to the action property. The last change that’s required is in the component which uses my-navigation as follows:

```html
<my-navigation>
  <ng-template #action><button>Home</button></ng-template>
</my-navigation>
```

As you can see, the button can now be styled and defined by the component that makes use of the my-navigation component, while the navigation component can remain rather generic with only one property added.

## Passing around Variables

The other benefit to this approach is, my-component can pass in a context variable that the child component can then access with the `let-` style naming. The above video does a great job of showing you how to do this in a use-case that makes sense, but I’ll show you a pretty made up example of how it would be used. It starts by binding to another property named `ngTemplateOutletContext`, and passing an `$implicit` parameter.

```html
<div>
  <div>
    <ng-container
      [ngTemplateOutlet]="action"
      [ngTemplateOutletContext]="{ $implicit: 'hello' }"
    ></ng-container>
  </div>
  <!-- scrolling content here -->
  <div>
    <ng-container
      [ngTemplateOutlet]="action"
      [ngTemplateOutletContext]="{ $implicit: 'hello' }"
    ></ng-container>
  </div>
</div>
```

In the example above, I pass in a hardcoded string, however, you can imagine passing in anything. Next you can use this variable by using the `let-` syntax in the component using the custom component. Here’s an example:

```html
<my-navigation>
  <ng-template #action let-title><button>{{ title }}</button></ng-template>
</my-navigation>
```

You can see from the above snippet, I named the variable `let-title`, and although the `let-` is required, you’re free to name the variable whatever you want.

## Takeaways

Angular is a fairly large library with a large footprint. This can be difficult to solve some problems without diving into the specifics of the library.

Hopefully this tutorial showed you a new way of reusing templates from custom components in Angular.