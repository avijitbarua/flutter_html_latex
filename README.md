# flutter_html_latex

`flutter_html_latex` হলো Flutter HTML content এর ভিতরে LaTeX equation render করার একটি production-ready package।

এই package এ:

- Primary renderer: `flutter_math_fork` (KaTeX-style, fast, clean layout)
- Fallback renderer: `flutter_tex` (MathJax-based complex formulas)
- User-friendly widget API: `HtmlLatex(...)`

## কেন এই package?

অনেক API/CMS এইভাবে equation দেয়:

```html
<span class="math-tex">\(x^2 + 2x + 1\)</span>
```

`flutter_widget_from_html_core` একা এটা render করে না।
`flutter_html_latex` এই gap fill করে এবং customization দেয়।

## Renderer Strategy (Important)

Default behavior এখন:

- `mathJaxSupported = false` (default)
- Meaning: `flutter_math_fork` কে prefer করবে
- Only hard error হলে fallback যাবে

যখন data MathJax generated (যেমন inline `aligned`, `equation`) তখন:

- `mathJaxSupported = true` use করা recommended
- risky pattern হলে আগে থেকেই `flutter_tex` path এ যাবে

## Demo Images

Repository demo screenshots:

- `demos/MathJax-Supported-false.png`
- `demos/MathJax-Supported-true.png`

Quick understanding:

1. `mathJaxSupported: false`
   - KaTeX-style content এর জন্য best quality + alignment
   - কিছু MathJax-only pattern problematic হতে পারে
2. `mathJaxSupported: true`
   - MathJax-heavy content safer
   - fallback usage বেশি হতে পারে

## Installation

`pubspec.yaml` এ:

```yaml
dependencies:
  flutter_html_latex: ^1.0.0
```

## Project Setup (Must Read)

### 1) main.dart পরিবর্তন

`runApp` এর আগে runtime initialize করো:

```dart
import 'package:flutter_html_latex/flutter_html_latex.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await FlutterHtmlLatexRuntime.ensureInitialized();
  runApp(const MyApp());
}
```

### 2) AndroidManifest.xml পরিবর্তন

`example/android/app/src/main/AndroidManifest.xml` এর `<application>` tag এ এটা add করা recommended:

```xml
<application
    android:label="your_app"
    android:enableOnBackInvokedCallback="true"
    ...>
```

## Quick Usage

```dart
HtmlLatex(
  htmlData,
  style: const TextStyle(
    fontSize: 15,
    color: Colors.black87,
  ),
)
```

## HtmlLatex Parameters বিস্তারিত

`HtmlLatex(data, ...)` constructor:

1. `data` (`String`, required)
   - HTML string input

2. `style` (`TextStyle?`)
   - default equation style
   - color, fontSize, fontFamily, fontWeight, fontStyle, letterSpacing, wordSpacing, decoration map হয়

3. `config` (`LatexHtmlWidgetFactoryConfig?`)
   - low-level advanced config pass করতে

4. `customStylesBuilder` (`Map<String, String>? Function(dynamic)?`)
   - element ভিত্তিক CSS map return

5. `customMathBuilder` (`Widget? Function(String, LatexStyleData)?`)
   - specific formula custom widget দিয়ে override

6. `onMathError` (`Widget? Function(Object, String)?`)
   - render error হলে custom widget

7. `enableFallback` (`bool?`)
   - `flutter_math_fork` error হলে `flutter_tex` fallback হবে কি না

8. `mathJaxSupported` (`bool?`)
   - `false`: KaTeX-first mode (default behavior)
   - `true`: MathJax-aware heuristics enabled

9. `responsiveLayout` (`bool?`)
   - wide equation হলে horizontal scroll wrapper

10. `fallbackScaleInline` (`double?`)
    - inline fallback size normalize

11. `fallbackScaleBlock` (`double?`)
    - block fallback size normalize

12. `fallbackVerticalPadding` (`double?`)
    - fallback equation vertical rhythm ঠিক করার জন্য

## LatexHtmlWidgetFactoryConfig Parameters

`LatexHtmlWidgetFactoryConfig(...)`:

1. `baseFontSize` (default `12.0`)
2. `defaultColor`
3. `defaultFontFamily`
4. `customStylesBuilder`
5. `customMathBuilder`
6. `onMathError`
7. `enableFallback` (default `true`)
8. `mathJaxSupported` (default `false`)
9. `responsiveLayout` (default `true`)
10. `fallbackScaleInline` (default `0.86`)
11. `fallbackScaleBlock` (default `0.92`)
12. `fallbackVerticalPadding` (default `2.0`)
13. `hyphenationCharacter` (default soft-hyphen)

## Recommended Presets

### Preset A: KaTeX-first (recommended default)

```dart
HtmlLatex(
  data,
  mathJaxSupported: false,
  enableFallback: true,
)
```

Use when:

- তোমার data mostly KaTeX compatible
- তুমি `flutter_math_fork` quality/performance maximize করতে চাও

### Preset B: MathJax-heavy content

```dart
HtmlLatex(
  data,
  mathJaxSupported: true,
  enableFallback: true,
  fallbackScaleInline: 0.84,
  fallbackScaleBlock: 0.90,
  fallbackVerticalPadding: 1.5,
)
```

Use when:

- data-তে `aligned`, `equation` ইত্যাদি frequent
- freeze/crash-risk formulas safely render করতে চাও

## Supported HTML Targets

Package এই pattern detect করে:

```html
<span class="math-tex">\( ... \)</span>
<span class="math-display">\[ ... \]</span>
<span class="math-inline">\( ... \)</span>
<math>\( ... \)</math>
```

## Supported Delimiters

- `\(...\)`
- `\[...\]`
- `$...$`
- `$$...$$`

## Troubleshooting

1. Equation freeze/crash in debug
   - `mathJaxSupported: true` করে দেখো
   - `enableFallback: true` আছে কিনা নিশ্চিত করো

2. Fallback equation size mismatch
   - `fallbackScaleInline`, `fallbackScaleBlock`, `fallbackVerticalPadding` tune করো

3. Formula not rendering
   - delimiter syntax check করো
   - malformed LaTeX কিনা check করো

## Example Toggle (from example app)

```dart
const kMathJaxSupported = true; // false / true switch

HtmlLatex(
  item.explanation,
  mathJaxSupported: kMathJaxSupported,
  style: const TextStyle(fontSize: 15, color: Colors.black87),
  fallbackScaleInline: 0.84,
  fallbackScaleBlock: 0.90,
  fallbackVerticalPadding: 1.5,
)
```

## License

MIT License. See `LICENSE`.
