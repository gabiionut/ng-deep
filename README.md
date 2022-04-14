# Styling child and third-party components in Angular

Angular provides a mechanism to encapsulate component CSS styles into the component’s view without affecting the rest of the application. This approach has many advantages but also causes a common problem that many developers encountered. Styling the child components in Angular is a common practice in Angular applications, but because of this mechanism used by the Angular framework, it can become a difficult task.

The `::ng-deep` pseudo-element can help up solve this problem, but using it comes with some drawbacks. In this article, we will look at the problems that can arise when using it and what are the alternatives.

## Angular View Encapsulation

To understand why the `::ng-deep` pseudo-element is used, let’s take a look at the Angular View Encapsulation mechanism.

View encapsulation is the Angular mechanism for defining what elements a component’s styles should apply to.

The two ViewEncapsulation values you will likely encounter are Emulated and None.

- `ViewEncapsulation.Emulated` means the styles defined in this component will only apply to the component’s HTML. This is the default value.

- `ViewEncapsulation.None` means the styles will be applied globally and can affect any HTML element present within the application.

Let's say that we want to style a Material Tab Group component, and change the background color of the tabs and the color of the nav bar, and also the color of the selected tab title.

![Custom tabs style](/assets/ng-deep.png)

For this we will have the following styles in the component's CSS:

```scss
.mat-tab-group {
  .mat-tab-header {
    background-color: #3f51b5;
  }
  .mat-tab-label {
    color: #fff;
  }

  .mat-tab-label-active {
    color: #f78eb1;
  }

  &.mat-primary .mat-ink-bar {
    background-color: #ff4081;
  }
}
```

[StackBlitz demo](https://stackblitz.com/edit/angular-pvgjqm-pfhmuk?file=src/app/custom-tabs/custom-tabs.component.scss)

As you can see, the applied colors haven't changed. Why is this so?

The answer is quite simple: Angular uses the View Encapsulation mode, which attaches some extra attribute (`_ng-content-***-***`) to each DOM element to wrap the whole SCSS code to the component’s unique attribute.

To make it easier, let’s style a simple HTML paragraph:

```html
<p>Example text</p>
```

And style it:

```scss
p {
  color: red;
}
```

What we see when we inspect it:

![View encapsulation](/assets/view-encapsulation.png)

The unique attribute has been added automatically by the View Encapsulation mode. If we come back to our example with the tabs, we see that Angular Material doesn’t always add a special attribute to each DOM element:

![View encapsulation](/assets/mat-tab.png)

When we try to style our component in the scoped mode, we don’t see any results because the browser reads our styles like this:

```scss
.mat-tab-group[_ng-content-***-***] {
    .mat-tab-header[_ng-content-***-***] {
        background-color: #3f51b5;
    }

    ...
  }

  // -***-*** is unique numbers
```

*You can check [Angular view encapsulation documentation](https://angular.io/guide/view-encapsulation) for more details.*

## What is `::ng-deep`?

The `::ng-deep` CSS selector is often used in an Angular Component's CSS to override the styles of a third-party component or a child component. For example, if you're using Angular Material (or any other third-party library like this), some generated elements are outside of your component's area (as we saw above) and you can't access those elements directly from the component's CSS, using a regular CSS way.

This means that we can use it to style our Material Tab Group component:

```scss
::ng-deep .mat-tab-group {
  .mat-tab-header {
    background-color: #3f51b5;
  }
  .mat-tab-label {
    color: #fff;
  }

  .mat-tab-label-active {
    color: #f78eb1;
  }

  &.mat-primary .mat-ink-bar {
    background-color: #ff4081;
  }
}
```

## The problem with `::ng-deep`

To understand the unintended side effects of the `::ng-deep` selector let’s see what the [Angular documentation](https://angular.io/guide/component-styles#deprecated-deep--and-ng-deep) says about it.

> *Applying the ::ng-deep pseudo-class to any CSS rule completely disables view-encapsulation for that rule. Any style with ::ng-deep applied becomes a global style.*

Using `::ng-deep` is the same as adding these style overrides to a global stylesheet! This means that all the Material Tabs from the application will have these styles applied to them. If our intent is for these styles to apply globally then they belong in a global stylesheet, not in the component level CSS.

Another problem with using the `::ng-deep` selector is that the Angular team has decided to deprecate it and plans to drop the support for it in the future. This means that when this will happen, we will need to change our styling approach in order to be able to update the application.

## The alternatives

A good practice is to check the library's documentation to see if there is a way provided directly in the library to override the component's styles. For example, for the Material Tab Group, we can use the `color` and `backgroundColor` inputs to overwrite the component's default theme.

```html
<mat-tab-group color="accent" backgroundColor="primary">
  <mat-tab label="First"></mat-tab>
  <mat-tab label="Second"></mat-tab>
  <mat-tab label="Third"></mat-tab>
</mat-tab-group>
```

In our example, we want to overwrite more than those colors provided by the library's API. For this we have the following alternatives:

### Turning off the View Encapsulation mode

A popular alternative for styling the child component is disabling view encapsulation for your component. Turning off the view encapsulation mode for a component will result in the styles from the component's CSS files being applied globally. In this way, we can customize the styles of the child component.

This can be achieved by setting the `encapsulation` property to `ViewEncapsulation.None` in the component's metadata.

```typescript
import { Component, OnInit, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-tabs',
  templateUrl: './tabs.component.html',
  styleUrls: ['./tabs.component.scss'],
  encapsulation: ViewEncapsulation.None
})
export class TabsComponent implements OnInit {
  constructor() {}

  ngOnInit() {}
}
```

Using this approach, all the styles defined in the component's CSS files will become global styles and will affect any element present in the application. In order to avoid affecting other components, we can use the component's selector to target only the elements that belong to the component.

```scss
custom-tabs {
  .mat-tab-group {
    .mat-tab-header {
      background-color: #3f51b5;
    }
    .mat-tab-label {
      color: #fff;
    }

    .mat-tab-label-active {
      color: #f78eb1;
    }

    &.mat-primary .mat-ink-bar {
      background-color: #ff4081;
    }
  }
}
```

[StackBlitz demo](https://stackblitz.com/edit/angular-pvgjqm-6ueagk?file=src%2Fapp%2Fcustom-tabs%2Fcustom-tabs.component.scss)

<div style="padding: 15px; border: 1px solid transparent; border-color: transparent; margin-bottom: 20px; border-radius: 4px; color: #a94442; background-color: #f2dede; border-color: #ebccd1;">
Although this approach can be used to avoid using the `::ng-deep` selector, it should be used with caution. Since the component's styles are treated now as global, it is very easy to define styles outside of the component's selector, that will affect others components from the application.
</div>

### Using SCSS mixins

Another option is to create a mixin in the component's CSS file and import it into the global stylesheet. This approach will allow us also to access the colors defined in the application's theme and use them to style our components.

Componet's stylesheet file:

```scss
@use 'sass:map';
@use '@angular/material' as mat;

@mixin custom-tabs-mixin($theme) {
  $color-config: mat.get-color-config($theme);

  $primary-palette: map.get($color-config, 'primary');
  $accent-palette: map.get($color-config, 'accent');

  custom-tabs {
    .mat-tab-group {
      .mat-tab-header {
        background-color: mat.get-color-from-palette($primary-palette, 500);
      }
  
      .mat-tab-label {
        color: #fff;
      }
  
      .mat-tab-label-active {
        color: mat.get-color-from-palette($accent-palette, 300);
      }
  
      &.mat-primary .mat-ink-bar {
        background-color: mat.get-color-from-palette($accent-palette, 500);
      }
    }
  }
}
```

Global stylesheet:

```scss
@use '@angular/material' as mat;
@use '../app/custom-tabs/custom-tabs.component.scss' as tabs;

@include mat.core();

$theme-primary: mat.define-palette(mat.$indigo-palette);
$theme-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);

$theme: mat.define-light-theme(
  (
    color: (
      primary: $theme-primary,
      accent: $theme-accent,
    ),
  )
);

@include mat.all-component-themes($theme);
@include tabs.custom-tabs-mixin($theme);
```

[StackBlitz demo](https://stackblitz.com/edit/angular-pvgjqm-qgmcsy?file=src%2Fstyles%2Ftheme.scss)

Using this approach, only the styles defined in the mixin are treated as global styles and, since we used the component's selector to target only the elements that belong to the component, the styles will not affect any other element in the application.

### Defining global custom styles

If we plan to reuse the component's styles in other components, a good practice is to define them in the global stylesheet. Using the same approach as above, we can define a mixin in a separate CSS file which can be imported into the main theming stylesheet. We can replace the component selector with a custom class, which can be used in the entire application to apply the custom styles to a Material Tabs component.

```scss
@use 'sass:map';
@use '@angular/material' as mat;

@mixin tab-red-color($theme) {
  $color-config: mat.get-color-config($theme);

  $primary-palette: map.get($color-config, 'primary');
  $accent-palette: map.get($color-config, 'accent');

  .custom-tabs.mat-tab-group {
    .mat-tab-header {
      background-color: mat.get-color-from-palette($primary-palette, 500);
    }

    .mat-tab-label {
      color: #fff;
    }

    .mat-tab-label-active {
      color: mat.get-color-from-palette($accent-palette, 300);
    }

    &.mat-primary .mat-ink-bar {
      background-color: mat.get-color-from-palette($accent-palette, 500);
    }
  }
}
```

Global stylesheet:

```scss
@use '@angular/material' as mat;
@use './_custom-tabs' as tabs;

@include mat.core();

$theme-primary: mat.define-palette(mat.$indigo-palette);
$theme-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);

$theme: mat.define-light-theme((
  color: (
    primary: $theme-primary,
    accent: $theme-accent,
  )
));

@include mat.all-component-themes($theme);
@include tabs.tab-red-color($theme);
```

Component's HTML:

```html
<mat-tab-group class="custom-tabs">
  <mat-tab label="First"></mat-tab>
  <mat-tab label="Second"></mat-tab>
  <mat-tab label="Third"></mat-tab>
</mat-tab-group>
```

## Conclusion

As you can see, there are multiple alternatives to the `::ng-deep` pseudo-class selector. Some of them can cause a few problems in our project if they are not used properly (eg. turning off the view encapsulation mode). The best way to style our child and third-party components is to use classic global styling with a solid and clear pattern.
