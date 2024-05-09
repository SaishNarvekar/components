# Theming Angular Material 2

## What is theming?

Angular Material's theming system lets you customize base, color, typography, and density styles for
components in your application. The theming system is based on Google's
[Material Design 2][material-design-theming] specification.

**This guide refers to Material 2, the previous version of Material. For the
latest Material 3 documentation, see the [theming guide][theming].**

**For information on how to update, see the section
on [how to migrate an app from Material 2 to Material 3](#how-to-migrate-an-app-from-material-2-to-material-3).**

This document describes the concepts and APIs for customizing colors. For typography customization,
see [Angular Material Typography][mat-typography]. For guidance on building components to be
customizable with this system, see [Theming your own components][theme-your-own].

[material-design-theming]: https://m2.material.io/design/material-theming/overview.html
[theming]: https://material.angular.io/guide/theming
[mat-typography]: https://material.angular.io/guide/typography
[theme-your-own]: https://material.angular.io/guide/theming-your-components

### Sass

Angular Material's theming APIs are built with [Sass](https://sass-lang.com). This document assumes
familiarity with CSS and Sass basics, including variables, functions, and mixins.

You can use Angular Material without Sass by using a pre-built theme, described in
[Using a pre-built theme](#using-a-pre-built-theme) below. However, using the library's Sass API
directly gives you the most control over the styles in your application.

## Palettes

A **palette** is a collection of colors representing a portion of color space. Each value in this
collection is called a **hue**. In Material Design, each hues in a palette has an identifier number.
These identifier numbers include 50, and then each 100 value between 100 and 900. The numbers order
hues within a palette from lightest to darkest.

Angular Material represents a palette as a [Sass map][sass-maps]. This map contains the
palette's hues and another nested map of contrast colors for each of the hues. The contrast colors
serve as text color when using a hue as a background color. The example below demonstrates the
structure of a palette. [See the Material Design color system for more background.][spec-colors]

```scss
$m2-indigo-palette: (
 50: #e8eaf6,
 100: #c5cae9,
 200: #9fa8da,
 300: #7986cb,
 // ... continues to 900
 contrast: (
   50: rgba(black, 0.87),
   100: rgba(black, 0.87),
   200: rgba(black, 0.87),
   300: white,
   // ... continues to 900
 )
);
```

[sass-maps]: https://sass-lang.com/documentation/values/maps
[spec-colors]: https://m2.material.io/design/color/the-color-system.html

### Create your own palette

You can create your own palette by defining a Sass map that matches the structure described in the
[Palettes](#palettes) section above. The map must define hues for 50 and each hundred between 100
and 900. The map must also define a `contrast` map with contrast colors for each hue.

You can use [the Material Design palette tool][palette-tool] to help choose the hues in your
palette.

[palette-tool]: https://m2.material.io/design/color/the-color-system.html#tools-for-picking-colors

### Predefined palettes

Angular Material offers predefined palettes based on the 2014 version of the Material Design
spec. See the [Material Design 2014 color palettes][2014-palettes] for a full list.

In addition to hues numbered from zero to 900, the 2014 Material Design palettes each include
distinct _accent_ hues numbered as `A100`, `A200`, `A400`, and `A700`. Angular Material does not
require these hues, but you can use these hues when defining a theme as described in
[Defining a theme](#defining-a-theme) below.

```scss
@use '@angular/material' as mat;

$my-palette: mat.$m2-indigo-palette;
```

[2014-palettes]: https://material.io/archive/guidelines/style/color.html#color-color-palette

## Themes

A **theme** is a collection of color, typography, and density options. Each theme includes three
palettes that determine component colors:

* A **primary** palette for the color that appears most frequently throughout your application
* An **accent**, or _secondary_, palette used to selectively highlight key parts of your UI
* A **warn**, or _error_, palette used for warnings and error states

You can include the CSS styles for a theme in your application in one of two ways: by defining a
custom theme with Sass, or by importing a pre-built theme CSS file.

### Custom themes with Sass

A **theme file** is a Sass file that calls Angular Material Sass mixins to output color,
typography, and density CSS styles.

#### The `core` mixin

Angular Material defines a mixin named `core` that includes prerequisite styles for common
features used by multiple components, such as ripples. The `core` mixin must be included exactly
once for your application, even if you define multiple themes. Including the `core` mixin multiple
times will result in duplicate CSS in your application.

```scss
@use '@angular/material' as mat;

@include mat.core();
```

#### Defining a theme

Angular Material represents a theme as a Sass map that contains your color, typography, and density
choices, as well as some base design system settings. See
[Angular Material Typography][mat-typography] for an in-depth guide to customizing typography. See
[Customizing density](#customizing-density) below for details on adjusting component density.

Constructing the theme first requires defining your primary and accent palettes, with an optional
warn palette. The `m2-define-palette` Sass function accepts a color palette, described in the
[Palettes](#palettes) section above, as well as four optional hue numbers. These four hues
represent, in order: the "default" hue, a "lighter" hue, a "darker" hue, and a "text" hue.
Components use these hues to choose the most appropriate color for different parts of
themselves.

```scss
@use '@angular/material' as mat;

$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette, 500);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);

// The "warn" palette is optional and defaults to red if not specified.
$my-warn: mat.m2-define-palette(mat.$m2-red-palette);
```

You can construct a theme by calling either `m2-define-light-theme` or `m2-define-dark-theme` with
the result from `m2-define-palette`. The choice of a light versus a dark theme determines the
background and foreground colors used throughout the components.

```scss
@use '@angular/material' as mat;

$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette, 500);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);

// The "warn" palette is optional and defaults to red if not specified.
$my-warn: mat.m2-define-palette(mat.$m2-red-palette);

$my-theme: mat.m2-define-light-theme((
 color: (
   primary: $my-primary,
   accent: $my-accent,
   warn: $my-warn,
 ),
 typography: mat.m2-define-typography-config(),
 density: 0,
));
```

#### Applying a theme to components

The `core-theme` Sass mixin emits prerequisite styles for common features used by multiple
components, such as ripples. This mixin must be included once per theme.

Each Angular Material component has a mixin for each [theming dimension](#theming-dimensions): base,
color, typography, and density. For example, `MatButton` declares `button-base`, `button-color`,
`button-typography`, and `button-density`. Each mixin emits only the styles corresponding to that
dimension of customization.

Additionally, each component has a "theme" mixin that emits all styles that depend on the theme
config. This theme mixin only emits color, typography, or density styles if you provided a
corresponding configuration to `m2-define-light-theme` or `m2-define-dark-theme`, and it always emits the
base styles.

Apply the styles for each of the components used in your application by including each of their
theme Sass mixins.

```scss
@use '@angular/material' as mat;

@include mat.core();

$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette, 500);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);

$my-theme: mat.m2-define-light-theme((
 color: (
   primary: $my-primary,
   accent: $my-accent,
 ),
 typography: mat.m2-define-typography-config(),
 density: 0,
));

// Emit theme-dependent styles for common features used across multiple components.
@include mat.core-theme($my-theme);

// Emit styles for MatButton based on `$my-theme`. Because the configuration
// passed to `m2-define-light-theme` omits typography, `button-theme` will not
// emit any typography styles.
@include mat.button-theme($my-theme);

// Include the theme mixins for other components you use here.
```

As an alternative to listing every component that your application uses, Angular Material offers
Sass mixins that includes styles for all components in the library: `all-component-bases`,
`all-component-colors`, `all-component-typographies`, `all-component-densities`, and
`all-component-themes`. These mixins behave the same as individual component mixins, except they
emit styles for `core-theme` and _all_ 35+ components in Angular Material. Unless your application
uses every single component, this will produce unnecessary CSS.

```scss
@use '@angular/material' as mat;

@include mat.core();

$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette, 500);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);

$my-theme: mat.m2-define-light-theme((
 color: (
   primary: $my-primary,
   accent: $my-accent,
 ),
 typography: mat.m2-define-typography-config(),
 density: 0,
));

@include mat.all-component-themes($my-theme);
```

To include the emitted styles in your application, [add your theme file to the `styles` array of
your project's `angular.json` file][adding-styles].

[adding-styles]: https://angular.io/guide/workspace-config#styles-and-scripts-configuration

#### Theming dimensions

Angular Material themes are divided along four dimensions: base, color, typography, and density.

##### Base

Common base styles for the design system. These styles don't change based on your configured
colors, typography, or density, so they only need to be included once per application. These
mixins include structural styles such as border-radius, border-width, etc. All components have a base
mixin that can be used to include its base styles. (For example,
`@include mat.checkbox-base($theme)`)

##### Color

Styles related to the colors in your application. These style should be included at least once in
your application. Depending on your needs, you may need to include these styles multiple times
with different configurations. (For example, if your app supports light and dark theme colors.)
All components have a color mixin that can be used to include its color styles. (For example,
`@include mat.checkbox-color($theme)`)

##### Typography

Styles related to the fonts used in your application, including the font family, size, weight,
line-height, and letter-spacing. These style should be included at least once in your application.
Depending on your needs, you may need to include these styles multiple times with different
configurations. (For example, if your app supports reading content in either a serif or sans-serif
font.) All components  have a typography mixin that can be used to include its typography
styles. (For example, `@include mat.checkbox-typography($theme)`)

##### Density

Styles related to the size and spacing of elements in your application. These style should be
included at least once in your application. Depending on your needs, you may need to include these
styles multiple times with different configurations. (For example, if your app supports both a
normal and compact mode). All components have a density mixin that can be used to include its
density styles. (For example, `@include mat.checkbox-density($theme)`)

##### Theme mixin

All components also support a theme mixin that can be used to include the component's styles for all
theme dimensions at once. (For example, `@include mat.checkbox-theme($theme)`).

The recommended approach is to rely on the `theme` mixins to lay down your base styles, and if
needed use the single dimension mixins to override particular aspects for parts of your app (see the
section on [Multiple themes in one file](#multiple-themes-in-one-file).)

### Using a pre-built theme

Angular Material includes four pre-built theme CSS files, each with different palettes selected.
You can use one of these pre-built themes if you don't want to define a custom theme with Sass.

| Theme                  | Light or dark? | Palettes (primary, accent, warn) |
|------------------------|----------------|----------------------------------|
| `deeppurple-amber.css` | Light          | deep-purple, amber, red          |
| `indigo-pink.css`      | Light          | indigo, pink, red                |
| `pink-bluegrey.css`    | Dark           | pink, blue-grey, red              |
| `purple-green.css`     | Dark           | purple, green, red               |

These files include the CSS for every component in the library. To include only the CSS for a subset
of components, you must use the Sass API detailed in [Defining a theme](#defining-a-theme) above.
You can [reference the source code for these pre-built themes][prebuilt] to see examples of complete
theme definitions.

You can find the pre-built theme files in the "prebuilt-themes" directory of Angular Material's
npm package (`@angular/material/prebuilt-themes`). To include the pre-built theme in your
application, [add your chosen CSS file to the `styles` array of your project's `angular.json`
file][adding-styles].

[prebuilt]: https://github.com/angular/components/blob/main/src/material/core/theming/prebuilt

### Defining multiple themes

Using the Sass API described in [Defining a theme](#defining-a-theme), you can also define
_multiple_ themes by repeating the API calls multiple times. You can do this either in the same
theme file or in separate theme files.

#### Multiple themes in one file

Defining multiple themes in a single file allows you to support multiple themes without having to
manage loading of multiple CSS assets. The downside, however, is that your CSS will include more
styles than necessary.

To control which theme applies when, `@include` the mixins only within a context specified via
CSS rule declaration. See the [documentation for Sass mixins][sass-mixins] for further background.

[sass-mixins]: https://sass-lang.com/documentation/at-rules/mixin

```scss
@use '@angular/material' as mat;

@include mat.core();

// Define a dark theme
$dark-theme: mat.m2-define-dark-theme((
 color: (
   primary: mat.m2-define-palette(mat.$m2-pink-palette),
   accent: mat.m2-define-palette(mat.$m2-blue-grey-palette),
 ),
  // Only include `typography` and `density` in the default dark theme.
  typography: mat.m2-define-typography-config(),
  density: 0,
));

// Define a light theme
$light-theme: mat.m2-define-light-theme((
 color: (
   primary: mat.m2-define-palette(mat.$m2-indigo-palette),
   accent: mat.m2-define-palette(mat.$m2-pink-palette),
 ),
));

// Apply the dark theme by default
@include mat.core-theme($dark-theme);
@include mat.button-theme($dark-theme);

// Apply the light theme only when the user prefers light themes.
@media (prefers-color-scheme: light) {
 // Use the `-color` mixins to only apply color styles without reapplying the same
 // typography and density styles.
 @include mat.core-color($light-theme);
 @include mat.button-color($light-theme);
}
```

#### Multiple themes across separate files

You can define multiple themes in separate files by creating multiple theme files per
[Defining a theme](#defining-a-theme), adding each of the files to the `styles` of your
`angular.json`. However, you must additionally set the `inject` option for each of these files to
`false` in order to prevent all the theme files from being loaded at the same time. When setting
this property to `false`, your application becomes responsible for manually loading the desired
file. The approach for this loading depends on your application.

### Application background color

By default, Angular Material does not apply any styles to your DOM outside
its own components. If you want to set your application's background color
to match the components' theme, you can either:
1. Put your application's main content inside `mat-sidenav-container`, assuming you're using
   `MatSidenav`, or
2. Apply the `mat-app-background` CSS class to your main content root element (typically `body`).

### Scoping style customizations

You can use Angular Material's Sass mixins to customize component styles within a specific scope
in your application. The CSS rule declaration in which you include a Sass mixin determines its scope.
The example below shows how to customize the color of all buttons inside elements marked with the
`.my-special-section` CSS class.

```scss
@use '@angular/material' as mat;

.my-special-section {
 $special-primary: mat.m2-define-palette(mat.$m2-orange-palette);
 $special-accent: mat.m2-define-palette(mat.$m2-brown-palette);
 $special-theme: mat.m2-define-dark-theme((
   color: (primary: $special-primary, accent: $special-accent),
 ));

 @include mat.button-color($special-theme);
}
```

### Reading hues from palettes

It is possible to read colors from a theme for use in your own components. For more information
about this see our guide on [Theming your own components][reading-colors].

[reading-colors]: https://material.angular.io/guide/theming-your-components#reading-color-values

## Customizing density

Angular Material's density customization is based on the
[Material Design density guidelines][material-density]. This system defines a scale where zero
represents the default density. You can decrement the number for _more density_ and increment the
number for _less density_.

The density system is based on a *density scale*. The scale starts with the
default density of `0`. Each whole number step down (`-1`, `-2`, etc.) reduces
the affected sizes by `4px`, down to the minimum size necessary for a component to render
coherently.

Components that appear in task-based or pop-up contexts, such as `MatDatepicker`, don't change their
size via the density system. The [Material Design density guidance][material-density] explicitly
discourages increasing density for such interactions because they don't compete for space in the
application's layout.

You can apply custom density setting to the entire library or to individual components using their
density Sass mixins.

```scss
// You can set a density setting in your theme to apply to all components.
$dark-theme: mat.m2-define-dark-theme((
  color: ...,
  typography: ...,
  density: -2,
));

// Or you can selectively apply the Sass mixin to affect only specific parts of your application.
.the-dense-zone {
  @include mat.button-density(-1);
}
```

[material-density]: https://m2.material.io/design/layout/applying-density.html

## Strong focus indicators

By default, most components indicate browser focus by changing their background color as described
by the Material Design specification. This behavior, however, can fall short of accessibility
requirements, such as [WCAG][], which require a stronger indication of browser focus.

Angular Material supports rendering highly visible outlines on focused elements. Applications can
enable these strong focus indicators via two Sass mixins:
`strong-focus-indicators` and `strong-focus-indicators-theme`.

The `strong-focus-indicators` mixin emits structural indicator styles for all components. This mixin
should be included exactly once in an application, similar to the `core` mixin described above.

The `strong-focus-indicators-theme` mixin emits only the indicator's color styles. This mixin should
be included once per theme, similar to the theme mixins described above. Additionally, you can use
this mixin to change the color of the focus indicators in situations in which the default color
would not contrast sufficiently with the background color.

The following example includes strong focus indicator styles in an application alongside the rest of
the custom theme API.

```scss
@use '@angular/material' as mat;

@include mat.core();
@include mat.strong-focus-indicators();

$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette, 500);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);

$my-theme: mat.m2-define-light-theme((
 color: (
   primary: $my-primary,
   accent: $my-accent,
 )
));

@include mat.all-component-themes($my-theme);
@include mat.strong-focus-indicators-theme($my-theme);
```

### Customizing strong focus indicators

You can pass a configuration map to `strong-focus-indicators` to customize the appearance of the
indicators. This configuration includes `border-style`, `border-width`, and `border-radius`.

You also can customize the color of indicators with `strong-focus-indicators-theme`. This mixin
accepts either a theme, as described earlier in this guide, or a CSS color value. When providing a
theme, the indicators will use the default hue of the primary palette.

The following example includes strong focus indicator styles with custom settings alongside the rest
of the custom theme API.

```scss
@use '@angular/material' as mat;

@include mat.core();
@include mat.strong-focus-indicators((
  border-style: dotted,
  border-width: 4px,
  border-radius: 2px,
));

$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette, 500);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);

$my-theme: mat.m2-define-light-theme((
 color: (
   primary: $my-primary,
   accent: $my-accent,
 )
));

@include mat.all-component-themes($my-theme);
@include mat.strong-focus-indicators-theme(purple);
```

[WCAG]: https://www.w3.org/WAI/standards-guidelines/wcag/glance/

## Theming and style encapsulation

Angular Material assumes that, by default, all theme styles are loaded as global CSS. If you want
to use [Shadow DOM][shadow-dom] in your application, you must load the theme styles within each
shadow root that contains an Angular Material component. You can accomplish this by manually loading
the CSS in each shadow root, or by using [Constructable Stylesheets][constructable-css].

[shadow-dom]: https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM
[constructable-css]: https://developers.google.com/web/updates/2019/02/constructable-stylesheets

## User preference media queries

Angular Material does not apply styles based on user preference media queries, such as
`prefers-color-scheme` or `prefers-contrast`. Instead, Angular Material's Sass mixins give you the
flexibility to apply theme styles to based on the conditions that make the most sense for your
users. This may mean using media queries directly or reading a saved user preference.

## Style customization outside the theming system

Angular Material supports customizing color, typography, and density as outlined in this document.
Angular strongly discourages, and does not directly support, overriding component CSS outside the
theming APIs described above. Component DOM structure and CSS classes are considered private
implementation details that may change at any time.

## How to migrate an app from Material 2 to Material 3

Angular Material does not offer an automated migration from M2 to M3 because the design of your app
is subjective. Material Design offers general principles and constraints to guide you, but
ultimately it is up to you to decide how to apply those in your app. That said, Angular Material's M3 themes were designed with maximum compatibility in mind.

### Update your components that use Angular Material themes to be compatible with Material 3

In order to facilitate a smooth transition from M2 to M3, it may make sense to have your components
support both M2 and M3 themes. Once the entire app is migrated, the support for M2 themes can be
removed.

The simplest way to accomplish this is by checking the theme version and emitting different styles
for M2 vs M3. You can check the theme version using the `get-theme-version` function from
`@angular/material`. The function will return `0` for an M2 theme or `1` for an M3 theme (see
[theme your own components using a Material 3 theme](https://material.angular.io/guide/theming#theme-your-own-components-using-a-material-3-theme)
for how to read values from an M3 theme).

```scss
@use '@angular/material' as mat;

@mixin my-comp-theme($theme) {
  @if (mat.get-theme-version($theme) == 1) {
    // Add your new M3 styles here.
  } @else {
    // Keep your old M2 styles here.
  }
}
```

### Pass a new M3 theme in your global theme styles

Create a new M3 theme object using `define-theme` and pass it everywhere you were previously passing
your M2 theme. All Angular Material mixins that take an M2 theme are compatible with M3 themes as
well.

### Update usages of APIs that are not supported for Material 3 themes

Because Material 3 is implemented as an alternate theme for the same components used for Material 2,
the APIs for both are largely the same. However, there are a few differences to be aware of:

- M3 expects that any `@include` of the `-theme`, `-color`, `-typography`, `-density`, or `-base`
  mixins should be wrapped in a selector. If your app includes such an `@include` at the root level,
  we recommend wrapping it in `html { ... }`
- M3 has a different API for setting the color variant of a component (see
  [using component color variants](https://material.angular.io/guide/theming#using-component-color-variants)
  for more).
- The `backgroundColor` property of `<mat-tab-group>` is not supported, and should not be used with
  M3 themes.
- The `appearance="legacy"` variant of `<mat-button-toggle>` is not supported, and should not be
  used with M3 themes.
- For M3 themes, calling `all-component-typographies` does _not_ emit the `typography-hierarchy`
  styles, as this would violate M3's guarantee to not add selectors. Instead, the
  `typography-hierarchy` mixin must be explicitly called if you want these styles in your app.
- The `typography-hierarchy` mixin outputs CSS class names that correspond to the names of M3
  typescale levels rather than M2 typescale levels. If you were relying on the M2 classes to style
  your app, you may need to update the class names.

### (Optional) add backwards compatibility styles for color variants

We recommend _not_ relying on the `color="primary"`, `color="accent"`, or `color="warn"` options
that are offered by a number of Angular Material components for M2 themes. However, if you want to
quickly update to M3 and are willing to accept the extra CSS generated for these variants, you can
enable backwards compatibility styles that restore the behavior of this API. Call the
`color-variants-backwards-compatibility` mixin from `@angular/material` with the M3 theme you want
to generate color variant styles for.

<!-- TODO(mmalerba): Upgrade to embedded example -->

```scss
@use '@angular/material' as mat;

$theme: mat.define-theme();

html {
  @include mat.all-component-themes($theme);
  @include mat.color-variants-backwards-compatibility($theme);
}
```

### (Optional) add backwards compatibility styles for typography hierarchy

Calling the `typography-hierarchy` mixin with an M3 theme generates styles for CSS classes that
match the names of M3 typescale names (e.g. `.mat-headline-large`) rather than ones that match M2
typescale names (`.mat-headline-1`). If you were using the M2 class names in your app we recommend
updating all usages to one of the new class names. However, to make migration easier, the
`typography-hierarchy` mixin does support emitting the old class names in addition to the new ones.
We have made a best effort to map the M2 classes to reasonable equivalents in M3. To enable these
styles, pass an additional argument `$back-compat: true` to the mixin.

<!-- TODO(mmalerba): Upgrade to embedded example -->

```scss
@use '@angular/material' as mat;

$theme: mat.define-theme();

@include mat.typography-hierarchy($theme, $back-compat: true);
```