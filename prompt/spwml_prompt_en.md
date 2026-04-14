# SpWML (Simple Widget Markup Language) Complete Reference

## What is SpWML?

SpWML is a text-based markup language for declaratively defining Flutter UI layouts.
It is built on top of SpBML (Simple Block Markup Language), a generic hierarchical block parser.
SpWML text is parsed into a widget tree and rendered as a Flutter UI.

A typical usage is: load a `.spwml` file as an asset, parse it with SpWMLBuilder, then call `build(context)` to get a Flutter Widget.

### Online Editor

SpWML can be previewed in real time using the online SpWML Editor:
https://simple-widget-markup-editor.web.app/

Pasting generated SpWML into this editor lets you instantly check the visual result without
running a Flutter app. This is useful for iterative design and quick validation.

Note: The editor is personally hosted and may occasionally be unavailable if traffic is high.

---

## Basic Syntax

### Block (Element) Format

```
(elementType, param1:value1, param2:value2)content text
```

- Parentheses `( )` wrap the type and parameters.
- Type name comes first, then comma-separated `key:value` pairs.
- Content text comes after the closing `)` on the same line.
- Multi-line content is supported: subsequent lines (without a leading `(`) are appended to the block's content.

### Nesting with `+`

Child elements are indicated by prepending `+` symbols:

```
(col)
+(row)
++(text)Hello
++(text)World
+(text)Second row
```

- One `+` = depth 1 child of the root element
- Two `++` = depth 2 (child of the depth-1 element)
- And so on.

### Comments

Lines starting with `//` are comments and are ignored by the parser.

### Escape

- To include special characters in parameter values, use backslash escaping:
  `\(`, `\)`, `\:`, `\,`, `\\`, `\ `
- If a content line starts with `+(`, prefix that line with `(esc)`.

---

## Structural Rules

- A SpWML file typically starts with a single root container element (e.g., `scroll`, `col`).
- Container elements (col, row, scroll, etc.) hold child elements.
- Leaf elements (text, btn, icon, img, etc.) contain only text content, not child elements.
  - Exception: `btn` with `type:block` can have a single child element.
- `table` must contain only `tableRow` (`tr`) children.
- `tableRow` must contain only text/menu type leaf elements.
- The following elements accept **exactly one direct child**:
  `card`, `block`, `circleAvatar`, `scroll`, `btn`, `badge`, `tooltip`
  Placing two or more direct children inside any of these is invalid.

```
// NG: btn has two direct children
(btn, type:block, sid:myBtn)
+(col)content
+(text)extra         // invalid — second child

// OK: wrap multiple items in a single container child
(btn, type:block, sid:myBtn)
+(col)
++(menu)line 1
++(menu)line 2
```

### Forbidden Syntax

**No blank lines.**
Blank lines are NOT treated as visual separators. A blank line is parsed as a literal newline
character appended to the preceding block's content, producing unintended output.
Use `//` comment lines if you need visual spacing in the source file.

```
// NG: blank line between elements
(text)Hello

(text)World

// OK: comment lines for visual separation
(text)Hello
////////////////////////////////////////////////////////////////////////////////
(text)World
```

**No leading spaces or tabs on element lines.**
Each line must start with `+` symbols (for nesting) or `(` (for root-level elements) directly.
Any space or tab before `(` or `+` is a syntax error.

```
// NG
  (text)indented        // leading spaces — invalid
    +(text)child        // leading spaces before + — invalid

// OK
(text)root level
+(text)child
++(text)grandchild
```

**Nesting depth must increment by exactly 1.**
A child element must have exactly one more `+` than its parent. Skipping levels is a syntax error.

```
// NG: (col) is at depth 0, so its child must use exactly one +
(col)
++(text)Hello           // invalid — skipped a level

// OK
(col)
+(text)Hello

// NG: (col) is at depth 1 (child of root), so its child must use ++
+(col)
+++(text)Hello          // invalid — skipped a level

// OK
+(col)
++(text)Hello
```

---

## Common Parameters

These parameters are available on most elements.

### Layout & Spacing

| Parameter | Shorthand | Description | Value |
|-----------|-----------|-------------|-------|
| mAll | - | Margin on all sides | double (dp) |
| mLeft / mTop / mRight / mBottom | mL / mT / mR / mB | Individual margins | double (dp) |
| pAll | - | Padding on all sides | double (dp) |
| pLeft / pTop / pRight / pBottom | pL / pT / pR / pB | Individual paddings | double (dp) |
| width / height | w / h | Fixed size. `Infinity` expands to parent size. | double (dp) or `Infinity` |
| minWidth / minHeight | minW / minH | Min size constraints | double (dp) |
| maxWidth / maxHeight | maxW / maxH | Max size constraints | double (dp) |

**Note on `w:Infinity` / `h:Infinity`:** Setting `Infinity` as the value expands the element to
fill the available space in the parent. This requires the parent to have a determined size in that
axis (same constraint as `wt`). Use this when you want an element to stretch to the parent's full
width or height without using flex weight.

```
// Stretch to full parent width
(text, w:Infinity)This text fills the parent width

// Stretch to full parent height inside a fixed-height row
(row, h:200)
+(block, w:100, h:Infinity, bgColor:blue[50])  // fills the 200dp height
+(text)right side
```
| weight | wt | Flex weight (like Expanded) | int |
| sid | - | String ID for runtime access | String |
| tag | - | Tag for state management | String |
| isGone | - | Hide the element | true / false |
| bgColor | - | Widget background color (structure elements only — see note) | Color string |

**`bgColor` vs `color`:**
- `bgColor` changes the background of the entire widget area. Use it only on structure elements
  that hold child views: `col`, `row`, `scroll`, `wrap`, `block`, `span`, `stack`, etc.
- For non-structure elements such as `btn`, `card`, `circleAvatar`, `icon`, `line`, etc.,
  use `color` to set the element's own color (background fill, icon color, line color, etc.).
  Setting `bgColor` on these elements will produce unexpected visual results.

```
// OK: bgColor on structure elements
(col, bgColor:#FAFAFA)
(row, bgColor:blue[50])
(block, bgColor:grey[100])

// OK: color on non-structure elements
(btn, type:filled, color:blue)
(card, color:white)
(circleAvatar, color:teal)
(line, color:grey)

// NG: bgColor on non-structure elements
(btn, type:filled, bgColor:blue)    // unexpected result
(card, bgColor:white)               // unexpected result
```

### Text Style Parameters

| Parameter | Description | Value |
|-----------|-------------|-------|
| fontSize | Font size | double |
| fontWeight | Font weight | w100 / w200 / w300 / w400 / w500 / w600 / w700 / w800 / w900 / thin / light / normal / regular / medium / bold / black |
| fontStyle | Font style | normal / italic |
| fontFamily | Font family name | String |
| textColor | Text color | Color string |
| textBGColor | Text background color | Color string |
| textAlign | Text alignment | left / right / center / justify / start / end |
| lineHeight | Line height multiplier | double |
| letterSpacing | Letter spacing | double |
| wordSpacing | Word spacing | double |
| textDeco | Text decoration | none / underline / overline / lineThrough |
| textDecoStyle | Decoration style | solid / double / dotted / dashed / wavy |
| textDecoColor | Decoration color | Color string |
| isSelectable | Text selectable | true / false |
| overflow | Text overflow | clip / ellipsis / fade / visible |
| maxLines | Max lines before overflow | int |

### Alignment Parameters

| Parameter | Description | Value |
|-----------|-------------|-------|
| hAlign | Horizontal alignment | start / end / center / spaceBetween / spaceAround / spaceEvenly |
| vAlign | Vertical alignment | start / end / center / spaceBetween / spaceAround / spaceEvenly |
| flexFit | Flex fit | tight / loose |

Note: For `col`, `hAlign` = CrossAxisAlignment, `vAlign` = MainAxisAlignment.
For `row`, they are reversed.

### Color Format

- Hex: `#RRGGBB` or `#AARRGGBB` (e.g., `#FF0000`, `#FFFF0000`)
- Material color: `red`, `blue`, `green`, `orange`, `purple`, `yellow`, `cyan`, `pink`, `teal`, `indigo`, `lime`, `amber`, `brown`, `grey`
- Material shade: `red[50]`, `blue[100]`, ... `blue[900]` (bracket notation, shades: 50/100/200/.../900)
- `transparent`
- `null` to use system theme color

**`white` and `black` do NOT use bracket notation.** They have dedicated suffixed names only.
Using `white[70]` or `black[54]` will cause a parse error.

Valid `white` variants:
`white`, `white10`, `white12`, `white24`, `white30`, `white38`, `white54`, `white60`, `white70`

Valid `black` variants:
`black`, `black12`, `black26`, `black38`, `black45`, `black54`, `black87`

```
// NG
textColor:white[70]     // invalid — bracket notation not supported for white/black
bgColor:black[54]       // invalid

// OK
textColor:white70
bgColor:black54
```

---

## Key Concepts

### SID (String ID)
Elements marked with `sid:myId` can be retrieved at runtime via `builder.getElement("myId")`.
Used to set callbacks on buttons, read text field values, etc.

**`sid` is required on the following elements** — always set a unique `sid` on them:
- All button-type elements: `btn`, `switchBtn`, `checkbox`, `checkbox2`, `radioBtn`, `radioBtn2`,
  `dropdownBtn`, `dropdownBtn2`, `popupMenuBtn`, `popupMenuBtn2`, `segmentedBtn`, `segmentedBtn2`
- `textField`
- `split`
- `slider`
- `colorPalette`
- `progressIndicator` (no user interaction, but must be controlled from Dart)

### weight (wt)
Works like Flutter's `Expanded`. In a `row` or `col`, children with `wt:1` expand to fill available space proportionally.
The top-level element (e.g., `scroll`) should use `wt:1` to fill the screen.

**`wt` can only be used when the parent has a determined size.** It must NOT be used when the
parent's width or height is unbounded (infinite). Valid parent conditions:
- The parent has a fixed `w` / `h`
- The parent itself has `wt` (which is sized by its own parent)
- The parent is the screen root

If the parent's size is undetermined in the relevant axis, `wt` will cause a Flutter layout error.

```
// OK: scroll has wt:1 (sized by the screen), so col inside can use wt
(scroll, wt:1)
+(col, wt:1)
++(text)fills height

// OK: row has fixed width, so children can use wt
(row, w:400)
+(text, wt:1)left
+(text, wt:1)right

// NG: col has no fixed height and no wt — its child cannot use wt on height axis
(col)
+(block, wt:1)          // invalid — parent height is unbounded
```

**`wt` must not create infinite scroll.** A `scroll` element is unbounded in its scroll direction.
Therefore, a direct child of `scroll` must NOT use `wt` in the scroll axis — doing so causes a
Flutter layout error because it tries to expand inside an infinite constraint.

```
// NG: scroll is vertically unbounded — its child cannot use wt on the height axis
(scroll, wt:1)
+(col, wt:1)            // invalid — scroll direction (vertical) is infinite

// OK: use a fixed height, or omit wt on the scroll child
(scroll, wt:1)
+(col)                  // col grows naturally with its content
++(text)item 1
++(text)item 2

// OK: wt on the horizontal axis is fine inside a vertical scroll
(scroll, wt:1)
+(row, wt:1)            // invalid on height axis — same issue
// → do not use wt inside scroll at all unless the axis is perpendicular and bounded
```

**`wt` cannot be used inside `table` or `tableRow`.** Table cells must have fixed sizes only.

### tag
Used by `dropdownBtn2`, `popupMenuBtn2`, `segmentedBtn2`, `radioBtn2`, `checkbox2` to identify selected options.

---

## Element Reference

### STRUCTURE ELEMENTS (Containers)

#### col
Vertical column container (Flutter's Column). Children stack vertically.

Parameters:
- hAlign: Cross-axis alignment (start / end / center / stretch / baseline)
- vAlign: Main-axis alignment (start / end / center / spaceBetween / spaceAround / spaceEvenly)
- mainAxisSize: min / max (default: max)
- elevation, borderRadius, borderColor, borderWidth, clipType, useMaterial

**Note on elevation:** `elevation` wraps the element with Flutter's Material widget, which can cause
visual interference with surrounding widgets. When using `elevation`, always combine it with
`materialPadding` to give the shadow room to render correctly.
In general, avoid `elevation` unless necessary — it is difficult to make look clean.

```
(col, bgColor:#FAFAFA, pAll:16, vAlign:center, hAlign:center)
+(h3)Title
+(text)Description text
```

---

#### row
Horizontal row container (Flutter's Row). Children are placed side by side.

Parameters:
- hAlign: Main-axis alignment
- vAlign: Cross-axis alignment
- mainAxisSize: min / max

```
(row, hAlign:spaceBetween, vAlign:center)
+(text, wt:1)Left content
+(btn, sid:actionBtn)Action
```

---

#### block
Single-child container. The primary pattern for embedding custom Flutter widgets into a SpWML layout.

Place a `block` with a `sid` in the markup as a named placeholder, then call `builder.replace()` from Dart to inject any Flutter widget at that position.
This keeps the overall layout structure readable in the `.spwml` file while allowing arbitrary widgets to be inserted at well-defined points.

```
(col, pAll:16)
+(h3)My Page
+(block, sid:mapView, w:400, h:300)
+(btn, type:filled, sid:submitBtn)Submit
```

In Dart:
```dart
b.replace("mapView", GoogleMap(...));
```

Use `replaceUnderStructure` when you want to replace the children of a container element:
```dart
b.replaceUnderStructure("listContainer", [card1, card2, card3]);
```

---

#### scroll
Scrollable single-child container. Use `wt:1` when this is the root element.

Parameters:
- axis: horizontal / vertical (default: vertical)
- scrollBehavior: always / never / auto
- isPrimary: true / false
- alignCenter: true / false

**Recommended:** Always set `elevation:0` on `scroll`. This wraps it with Flutter's Material,
which prevents child widgets (especially ink effects and overlays) from visually bleeding
outside the scroll area into the surrounding UI.

```
(scroll, wt:1, pAll:8, elevation:0)
+(col)
++(text)Content here
```

---

#### card
Material card container.

Parameters:
- cardElevation, borderRadius, borderColor, borderWidth
- shape: roundRect / stadium / bevel / circle
- shadowColor

**Note on elevation vs cardElevation:** Do NOT use `elevation` on `card` — it produces unnatural
double-shadow rendering. Use `cardElevation` instead to control the card's shadow correctly.
As with other elements, avoid elevation in general unless the design specifically requires it.

```
(card, mAll:8, cardElevation:2)
+(col, pAll:16)
++(titleM)Card Title
++(bodyM)Card content here.
```

---

#### span
Inline row of elements (like HTML `<span>`). Children rendered inline.

```
(span)
+(text, textColor:red)Red
+(text, textColor:blue) and Blue
```

---

#### stack
Layered container (Flutter's Stack). Children rendered on top of each other.

Parameters: shiftX, shiftY for child positioning.

```
(stack)
+(block, w:200, h:100, bgColor:#EEE)
+(text, textColor:white)Overlay text
```

---

#### wrap
Wrapping container (Flutter's Wrap). Children wrap to next line when they exceed width.

Parameters: hAlign / vAlign, axis: horizontal / vertical

```
(wrap)
+(block, w:150, h:80, bgColor:blue[50])
+(block, w:150, h:80, bgColor:green[50])
+(block, w:150, h:80, bgColor:red[50])
```

---

#### expTile
Expandable/collapsible tile (accordion). Content text = tile title. Children shown when expanded.

```
(expTile)FAQ Question 1
+(text)Answer to question 1.
```

---

#### table
Table container. Must contain only `tableRow` (`tr`) children.

Parameters:
- hNum: Number of columns (int, required)

```
(table, hNum:3, mT:8)
+(tr)
++(menu, w:100, h:40, textAlign:center)Header 1
++(menu, w:100, textAlign:center)Header 2
++(menu, w:100, textAlign:center)Header 3
+(tr)
++(menu, w:100, h:40)Cell 1
++(menu, w:100)Cell 2
++(menu, w:100)Cell 3
```

---

#### tableRow (shorthand: tr)
A row inside a `table`. Contains leaf text elements as cells.

---

#### circleAvatar
Circular container for profile images or initials.

Parameters: radius, color (background fill), fgColor

```
(circleAvatar, radius:32, color:teal)
+(menu, textColor:white)AB
```

---

#### badge
Adds a notification badge overlay to its child.

Parameters:
- label: Badge text (omit for small dot)
- smallSize, isLabelVisible: true / false
- offsetX / offsetY: Position offset

```
(badge, label:3)
+(icon, iconNum:0xe22a)
```

---

#### tooltip
Shows a tooltip on hover/long press. Content text = tooltip message.

Parameters: offsetY, triggerMode (tap / longPress / manual), text style parameters

```
(tooltip)Open email
+(btn, type:icon, iconNum:0xe22a, sid:emailBtn)
```

---

#### split
Resizable split pane with two children separated by a draggable bar.

Parameters:
- axis: horizontal / vertical (default: horizontal)
- barSize, color: Divider bar
- clampMin / clampMax: Ratio limits (0.0–1.0)
- splitPane1MinPx / splitPane2MinPx: Min pixel size per pane

```
(split, w:400, h:200, barSize:8, color:grey, sid:mySplit)
+(col, bgColor:blue[50])
++(text)Left pane
+(col, bgColor:green[50])
++(text)Right pane
```

---

### TEXT ELEMENTS (Leaf)

Text elements display text content and do NOT have child elements.

#### Material Design 3 (recommended)
| Type | Usage |
|------|-------|
| displayL/M/S | App name, page title, hero text |
| headlineL/M/S | Section headings, card titles |
| titleL/M/S | List items, subtitles, form labels |
| labelL/M/S | Button labels, menu items, captions |
| bodyL/M/S | Article text, descriptions, settings |

#### Material Design 2 (legacy)
| Type | Usage |
|------|-------|
| h1 – h6 | Headings (h1 largest) |
| subtitle1, subtitle2 | Subtitles |
| body1, body2 | Body text |
| caption | Small caption text |
| overline | All-caps small label |

#### Special Text Types
| Type | Usage |
|------|-------|
| text | General body text (same as body1 by default) |
| menu | No top margin, text selection disabled (see below) |
| href | Hyperlink. Content = URL. `alt` param = display text |
| ruby | Text with ruby annotation above. `rubyText` param = annotation |
| textField (tf) | Text input field |
| superscript | Superscript text |
| subscript | Subscript text |

#### text vs menu — When to use which

**`text` and non-Material3 types (h1–h6, body1/2, subtitle1/2, caption, overline) have the following defaults:**
- Top margin is applied automatically (to create visual spacing between elements).
- Text is selectable by default (user can long-press/drag to copy).

**`menu` has different defaults:**
- No top margin (zero spacing above the element).
- Text selection is disabled.

**Use `menu` in the following situations:**
- Inside a `btn` with `type:block` — the entire block is a button, so text selection would be wrong.
- Inside any interactive element (dropdown options, popup menu items, list tiles, etc.) where
  selectable text would feel broken or interfere with tap behavior.
- Anywhere you want text without the automatic top margin (e.g., tightly packed layouts).

```
// Wrong: text inside a block button — user can accidentally select text instead of tapping
(btn, type:block, sid:myCard)
+(col, pAll:16)
++(text)Card label        // NG: has top margin and is selectable

// Correct
(btn, type:block, sid:myCard)
+(col, pAll:16)
++(menu)Card label        // OK: no top margin, not selectable
```

#### href Example
```
(href, alt:Click here)https://example.com
```

#### ruby Example
```
(ruby, rubyText:かん字)漢字
```

#### textField (shorthand: tf) Parameters
- hint: Placeholder text
- labelText: Floating label text
- maxLength: Max character count
- maxLines: Max lines (1 = single line)
- readOnly: true / false
- type: material / rounded (default: material)
- mode: normal / password
- keyboardType: text / multiline / number / phone / datetime / emailAddress / url
- inputType: text / number / decimal / signed / signedDecimal / email / password / phone / date / dateTime / time
- isEnabled: true / false
- min / max: For number input validation
- suffixIconNum, suffixIconSize, suffixIconColor

```
(tf, hint:Enter email, labelText:Email, keyboardType:emailAddress, sid:emailField)
```

---

### BUTTON ELEMENTS

#### btn
Button with multiple style variants.

Parameters:
- type: text (default) / outlined / elevated / filled / filledTonal / icon / iconFilled / iconFilledTonal / iconOutlined / block / faSmall / fa / faLarge / faExtended
- iconNum: Icon hex code (e.g., `0xe047`)
- iconSize, iconColor, btnBGColor, elevation, borderRadius
- shape: roundRect / stadium / bevel / circle
- isEnabled: true / false
- isSelected: true / false (for icon-type buttons)
- selectedIconNum: Icon when selected

Notes:
- `type:block` makes entire child subtree act as a button.
- `type:icon` and FA types do not support content text.
- Most button types support child elements (since v13).

```
(btn, type:filled, iconNum:0xe047, btnBGColor:blue, textColor:white, sid:submitBtn)Submit
```

---

#### switchBtn
Toggle switch.

Parameters: type (normal / check), sid

---

#### checkbox / checkbox2
Checkbox. Children are `menu` elements as labels.
- `checkbox2` uses `tag` for multi-select group identification.

Parameters: isPrefixIcon (true/false), fillColor

```
(checkbox, sid:agree)
+(menu)I agree to the terms
```

---

#### radioBtn / radioBtn2
Radio button for single selection.
- `radioBtn2` uses `tag` for value identification. Children are `menu` elements.

---

#### dropdownBtn / dropdownBtn2
Dropdown selector. Children are `menu` elements as options.
- `dropdownBtn2` uses `tag` for selected value identification.

Parameters: isExpanded (true/false), hint, sid

```
(dropdownBtn2, sid:langSelect, hint:Choose language)
+(menu, tag:en)English
+(menu, tag:ja)Japanese
+(menu, tag:zh)Chinese
```

---

#### popupMenuBtn / popupMenuBtn2
Popup context menu. Children are `menu` elements.
- `popupMenuBtn2` uses `tag` for identification.

```
(popupMenuBtn, sid:optionsMenu)
+(menu)Edit
+(menu)Delete
+(menu)Share
```

---

#### segmentedBtn / segmentedBtn2
Segmented control. Children are `menu` elements as segments.
- `segmentedBtn2` uses `tag` for segment identification.

Parameters: isMultiSelection (true/false), allowEmpty (true/false)

---

### OTHER ELEMENTS

#### icon
Material icon display.

Parameters: iconNum (hex code), iconSize, iconColor

```
(icon, iconNum:0xe22a, iconSize:32, iconColor:blue)
```

---

#### img
Image display (asset or network).

Parameters:
- type: asset (default) / network / file
- fit: fill / contain / cover / fitWidth / fitHeight / none / scaleDown
- repeat: noRepeat / repeat / repeatX / repeatY
- clipType: oval / rRect
- clipRadius: Border radius for rRect clip
- alt: Accessibility label

```
(img, w:200, h:100, fit:cover)assets/images/photo.png
```

---

#### line
Horizontal divider line.

Parameters: thickness, color

---

#### vline
Vertical divider line.

Parameters: thickness, color, h (height)

---

#### progressIndicator
Loading indicator.

Parameters: type (circular / linear), indicatorColor, indicatorBGColor

---

#### slider
Value slider.

Parameters:
- min, max: Value range (double)
- divisions: Discrete steps (int)
- activeColor, inactiveColor
- label: Text label above thumb
- useAutoLabel: true = show current value
- isIntValue: true = round to integer
- sid: For value retrieval / callback

---

#### colorPalette
Color picker palette. Content = comma-separated color values.

Parameters:
- type: normal / simple / text / circle / simpleCircle
- cellWidth, cellHeight, cellMargin, cellBorderWidth, cellBorderColor
- hAlign, vAlign, sid

```
(colorPalette, sid:cp1)#FF0000,#00FF00,#0000FF
red[100], red[200], red[300]
```

---

## Complete SpWML Example

```
(scroll, wt:1, pAll:16)
+(col)
++(h2)User Profile
++(line)
++(row, mT:16)
+++(circleAvatar, radius:40)
++++(menu)AB
+++(col, mL:16)
++++(titleL)Alice Brown
++++(bodyM, textColor:grey)Software Engineer
++(textField, hint:Enter bio, mT:16, sid:bioField)
++(row, mT:16, hAlign:end)
+++(btn, type:outlined, mR:8, sid:cancelBtn)Cancel
+++(btn, type:filled, sid:saveBtn)Save
```

---

## Dart / Flutter Integration

### Package Setup

```yaml
dependencies:
  simple_widget_markup: ^latest
  simple_managers: ^latest
```

```dart
import 'package:simple_widget_markup/simple_widget_markup.dart';
```

---

### Basic Usage

```dart
SpWMLBuilder b = SpWMLBuilder(spwmlText, padding: EdgeInsets.zero);
Widget w = b.build(context);
```

---

### Loading from Assets

```dart
String? layout = SpWMLLayoutManager().getAssets(
  "assets/layout/ja/home/any/page_home.spwml",
  () { if (mounted) setState(() {}); },
  (e) { debugPrint("Load error: $e"); },
);

if (layout == null) {
  return const Center(child: CircularProgressIndicator());
}
SpWMLBuilder b = SpWMLBuilder(layout, padding: EdgeInsets.zero);
return b.build(context);
```

Asset path convention: `assets/layout/{lang}/{pageName}/{windowClass}/{fileName}.spwml`

---

### Accessing Elements by SID

Call `getElement()` and configure callbacks BEFORE `build(context)`.

#### Button Callback
```dart
BtnElement btn = b.getElement("submitBtn") as BtnElement;
btn.setCallback(() { /* handle press */ });
btn.setEnabled(false);
```

#### Button Loading State
`setLoading()` replaces the button's content with a `CircularProgressIndicator` and disables
the button so it cannot be pressed. Useful for async operations like form submission.

Because `BtnElement` is stateless, `setLoading()` must be called during the build process.
Manage an `isLoading` flag in the parent widget, and pass it to `setLoading()` conditionally
on each build.

```dart
// In StatefulWidget:
bool _isLoading = false;

// In build():
SpWMLBuilder b = SpWMLBuilder(layout);
BtnElement btn = b.getElement("submitBtn") as BtnElement;
btn.setCallback(() async {
  setState(() => _isLoading = true);
  await doSomething();
  setState(() => _isLoading = false);
});
if (_isLoading) btn.setLoading();
return b.build(context);
```

#### Text Field
```dart
TextFieldElement field = b.getElement("emailField") as TextFieldElement;
field.setText("user@example.com");
String? value = field.getText();
```

#### Dropdown (tag-based)
```dart
DropdownBtn2Element dd = b.getElement("myDropdown") as DropdownBtn2Element;
dd.setCallback((String? tag) { print("Selected: $tag"); });
```

#### Switch
```dart
SwitchBtnElement sw = b.getElement("mySwitch") as SwitchBtnElement;
sw.setCallback((bool value) { print("Switch: $value"); });
sw.setValue(true);
```

#### Checkbox
```dart
CheckboxElement cb = b.getElement("myCheckbox") as CheckboxElement;
cb.setCallback((bool? value) { print("Checked: $value"); });
```

#### Slider
```dart
SliderElement slider = b.getElement("mySlider") as SliderElement;
slider.setOnChanged((double v) { print("Value: $v"); });
slider.setValue(0.5);
```

#### Split Pane
```dart
SplitElement split = b.getElement("mySplit") as SplitElement;
split.setOnChanged((double ratio) { print("Ratio: $ratio"); });
```

#### Embedding Custom Flutter Widgets (Recommended Pattern)

The recommended way to embed a custom Flutter widget into a SpWML layout is:
1. Place a `(block, sid:myId)` as a named placeholder in the `.spwml` file.
2. Call `builder.replace("myId", myWidget)` from Dart before `build(context)`.

This makes the layout file readable — the overall structure is visible in SpWML,
and only the parts that require custom code are replaced at specific named positions.

```dart
// In .spwml:
// (block, sid:chartArea, w:400, h:300)

b.replace("chartArea", MyChartWidget());

// To replace children of a container:
b.replaceUnderStructure("listContainer", [card1, card2, card3]);
```

---

### StateManager Integration

```dart
final StateManager _sm = StateManager();

// In build:
SpWMLBuilder b = SpWMLBuilder(layout);
b.setStateManager(_sm);

// In dispose:
_sm.dispose();
```

---

### Full Page Example

```dart
class MyPage extends StatefulWidget {
  const MyPage({super.key});
  @override
  State<MyPage> createState() => _MyPageState();
}

class _MyPageState extends State<MyPage> {
  final StateManager _sm = StateManager();

  @override
  void dispose() {
    _sm.dispose();
    super.dispose();
  }

  String? _getLayout(BuildContext context) {
    return SpWMLLayoutManager().getAssets(
      "assets/layout/ja/my/any/my_page.spwml",
      () { if (mounted) setState(() {}); },
      (e) { debugPrint("Error: $e"); },
    );
  }

  void _initCallbacks(SpWMLBuilder b) {
    BtnElement saveBtn = b.getElement("saveBtn") as BtnElement;
    saveBtn.setCallback(() {
      TextFieldElement field = b.getElement("nameField") as TextFieldElement;
      String? name = field.getText();
    });
  }

  @override
  Widget build(BuildContext context) {
    final layout = _getLayout(context);
    if (layout == null) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }
    SpWMLBuilder b = SpWMLBuilder(layout, padding: EdgeInsets.zero);
    b.setStateManager(_sm);
    _initCallbacks(b);
    return Scaffold(body: SafeArea(child: b.build(context)));
  }
}
```

---

### Tips

- Always configure elements via `getElement()` BEFORE calling `build(context)`.
- `SpWMLLayoutManager` caches files — safe to call `getAssets` on every build.
- For responsive layouts, use `SpWMLLayoutManager.getWindowClass(context).name`
  to select compact / medium / expanded variants.
- SIDs must be unique within a SpWML document.
- `wt:1` on the root element fills the screen.
- **Preferred pattern for custom widgets:** Use `(block, sid:myId)` as a placeholder in the layout,
  then call `builder.replace("myId", myWidget)` from Dart. This keeps the layout structure
  readable in the `.spwml` file and avoids scattering widget construction logic throughout build code.

---

## SpWML Self-Check List

Before finalizing SpWML output, verify all of the following items.

### Syntax

- [ ] No blank lines anywhere in the file (blank lines insert literal newlines into content).
- [ ] No leading spaces or tabs before `(` or `+` on any line.
- [ ] Every element line starts with the correct number of `+` symbols.
      A child must have exactly one more `+` than its parent — no skipped levels.

### SID

- [ ] All interactive and Dart-controlled elements have a `sid` set:
      btn, switchBtn, checkbox, checkbox2, radioBtn, radioBtn2,
      dropdownBtn, dropdownBtn2, popupMenuBtn, popupMenuBtn2, segmentedBtn, segmentedBtn2,
      textField, split, slider, colorPalette, progressIndicator
- [ ] All `sid` values are unique within the document.

### Structure

- [ ] Single-child-only elements contain exactly one direct child:
      `card`, `block`, `circleAvatar`, `scroll`, `btn`, `badge`, `tooltip`
      must each have at most one direct child element.
      If multiple items are needed inside them, wrap in a `col` or `row` first.
- [ ] `table` contains only `tableRow` (`tr`) children.
- [ ] `tableRow` contains only leaf text/menu elements as cells.
- [ ] Leaf text elements (text, menu, h1–h6, bodyL/M/S, etc.) have no child elements.
      Exception: `btn` with `type:block` may have children.

### Parameters

- [ ] No invalid `type` values.
      Check valid types per element:
      - btn: text / outlined / elevated / filled / filledTonal / icon / iconFilled /
             iconFilledTonal / iconOutlined / block / faSmall / fa / faLarge / faExtended
      - switchBtn: normal / check
      - textField: material / rounded
      - img: asset / network / file
      - progressIndicator: circular / linear
      - colorPalette: normal / simple / text / circle / simpleCircle
      - shape: roundRect / stadium / bevel / circle
      Never use `none` or other unlisted values for `type`.

- [ ] `fontWeight` uses only valid values:
      w100 / w200 / w300 / w400 / w500 / w600 / w700 / w800 / w900 /
      thin / light / normal / regular / medium / bold / black
      Invalid examples: `extraLight`, `semiBold`, `extraBold` — these do not exist.

- [ ] `bgColor` is used only on structure elements (col, row, scroll, wrap, block, span, stack, etc.).
      Non-structure elements (btn, card, circleAvatar, icon, line, etc.) use `color` instead.

- [ ] `white` and `black` color variants use suffixed names, NOT bracket notation:
      Valid white: white / white10 / white12 / white24 / white30 / white38 / white54 / white60 / white70
      Valid black: black / black12 / black26 / black38 / black45 / black54 / black87
      `white[70]` or `black[54]` are invalid and will cause a parse error.

- [ ] `wt` is used only when the parent has a determined size (fixed w/h, or parent has wt, or screen root).
      `wt` must NOT be used as a direct child of `scroll` in the scroll direction.
      `wt` must NOT be used inside `table` or `tableRow`.

- [ ] `w:Infinity` / `h:Infinity` is used only when the parent has a determined size in that axis.

- [ ] `elevation` on `col`/`row`/etc. is paired with `materialPadding`.
      `card` uses `cardElevation`, not `elevation`.
      `scroll` uses `elevation:0` (recommended).

### Text elements

- [ ] Inside `btn type:block` and other interactive containers, `menu` is used instead of `text`
      (no top margin, not selectable).
- [ ] `text`, `h1`–`h6`, `body1`/`body2`, etc. are not placed where text selection would be wrong.
