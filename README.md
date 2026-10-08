# docs-render-action

A GitHub Action that renders AsciiDoc documentation to standalone HTML using Ruby [asciidoctor](https://rubygems.org/gems/asciidoctor), with support for:

- `[plantuml]` diagram blocks via the `asciidoctor-diagram` extension (renders SVG through a PlantUML jar)
- `[tabs]` tabbed sections via the `asciidoctor-tabs` extension
- client-side syntax highlighting (`source-highlighter=highlight.js`) language classes

The action produces one sibling `.html` file for each rendered document, matching the output of `make documentation` in [infinispan-operator](https://github.com/infinispan/infinispan-operator). Use it to keep what contributors see locally identical to what is published on https://infinispan.github.io.

## Usage

```yaml
- name: Render docs
  uses: infinispan/docs-render-action@<commit-sha> # v1
  with:
    docs_path: documentation/asciidoc/titles
```

Pin the action to a commit SHA, per project convention. When you bump it here, update every consumer workflow (see "Consumers" below).

## Inputs

| Name               | Default      | Description                                                        |
| ------------------ | ------------ | ------------------------------------------------------------------ |
| `docs_path`        | *(required)* | Directory containing the `.asciidoc` documents to render. Searched recursively; each document is rendered in place, next to its source file. |
| `plantuml_version` | `1.2026.8`   | Version of the PlantUML jar downloaded from GitHub releases and passed to asciidoctor-diagram via `DIAGRAM_PLANTUML_CLASSPATH`. Keep in sync with `PLANTUML_VERSION` in infinispan-operator's Makefile. |

## Requirements

- A runner image with Java on `PATH` (required by the PlantUML jar). GitHub's `ubuntu-latest` images include it.
- Network access to rubygems.org and github.com for gem installation and the PlantUML download.

## Maintenance

All tool versions are pinned in a single place, in [`action.yml`](./action.yml):

| Tool                  | Where to bump                                   | Notes                                                                    |
| --------------------- | ----------------------------------------------- | ------------------------------------------------------------------------ |
| asciidoctor           | `gem install ... asciidoctor -v`                | Keep in sync with the version used by infinispan-operator's Makefile.     |
| asciidoctor-diagram   | `gem install ... asciidoctor-diagram -v`        | Same as above.                                                            |
| asciidoctor-tabs      | `gem install ... asciidoctor-tabs -v`           | Still a prerelease on rubygems; pin explicitly until a stable release exists. |
| PlantUML              | `plantuml_version` input default                | Keep in sync with the operator Makefile and `PLANTUML_VERSION`.          |

To add another asciidoctor extension: install its gem in the "Install asciidoctor and extensions" step, require it on the `asciidoctor` command line in the "Render documents to HTML" step, and exercise it from [`examples/sample.asciidoc`](./examples/sample.asciidoc) so the repository's own CI smoke test covers it.

## Smoke test

The [ci.yml](./.github/workflows/ci.yml) workflow renders `examples/sample.asciidoc` with this action on every push and pull request, then asserts that highlighted source blocks, a PlantUML SVG image, and tab markup all appear in the generated HTML. Extend the sample document whenever you add features here.

## Consumers

Workflows using this action (update their pinned SHA when releasing changes):

- [infinispan/infinispan-operator](https://github.com/infinispan/infinispan-operator) — `.github/workflows/sync_docs.yaml`
- [infinispan/spring-ai-infinispan](https://github.com/infinispan/spring-ai-infinispan) — `.github/workflows/sync_docs.yml`
