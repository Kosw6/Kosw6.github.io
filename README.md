# Kosw6.github.io
Kosw6 깃허브 페이지

## Languages

- Korean pages keep their existing URLs. English editions live under `/en/`.
- Each English page declares `lang: en`, `locale: en-US`, and `translation_url`
  pointing to its Korean counterpart. Keep internal English links under `/en/`.
- The masthead links directly to the matching page in either language and keeps
  the selector visible outside the mobile overflow menu. No translation service
  or JavaScript is required to switch languages.
- English report editions cover the design, key measurements, and validation
  limits, with explicit links to full Korean reports and their experiment logs.
- Local overrides of the pinned Minimal Mistakes default layout, masthead, and
  SEO include provide per-page language and alternate-language metadata. The
  masthead must use `include`, not `include_cached`, because its URLs vary by page.
- Build locally with `bundle exec jekyll build`.
