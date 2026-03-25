## Presentation Context & Audience

This slide deck demonstrates an agentic tool that audits NuGet package dependencies and support lifecycles across a .NET solution, then distils the findings into a single, business-readable report. The primary audience is **non-technical stakeholders** — product owners, delivery managers, and commercial decision-makers — who need to understand the health and risk posture of a software solution without requiring knowledge of package versioning, build tooling, or dependency graphs.

### Translating technical concepts into business outcomes

When writing or editing slides, always favour business-outcome language over technical jargon. The goal is to answer the question *"what does this mean for the product and the team?"* not *"what does the tool report?"*

| Technical concept | Business framing |
|---|---|
| Package version drift | Compounding upgrade cost — the longer it waits, the more expensive it becomes |
| CVE / security vulnerability | An active risk that could expose the product or customer data |
| End-of-life platform target | The platform will stop receiving security patches on a known date |
| Pre-release dependency in production | Unfinished third-party code is running in a live environment |
| Major version gap | A breaking-change upgrade is required — higher effort, higher risk |
| Blast radius (packages used by many projects) | A single change touches many parts of the product simultaneously |
| Licence metadata unknown | Legal exposure — we may be using code under terms we haven't agreed to |
| Health grade (A–F) | A single, scannable signal of overall solution hygiene suitable for a status dashboard |

### Memes are encouraged

Memes are a legitimate presentation tool. A well-chosen meme placed alongside a key point:
- Provides instant emotional context that words alone cannot
- Makes the slide memorable — audiences recall images far better than bullet points
- Signals confidence and approachability in a speaker

When selecting a meme, pick one that is **universally recognisable** (readable from the back of a room), **directly reinforces the point on the slide** (not just decoration), and **won't alienate the audience**. Store meme images in `slides/img/` and reference them as `bg right` or `bg left` backgrounds to juxtapose with the slide content.

Instruct the user what meame to search for and download; dont assume that you will be able to download it yoruself due to all sorts of issues

### Messaging principles

- **Lead with outcomes, not metrics.** "No known security vulnerabilities" is more meaningful to a product owner than "0 CVEs found".
- **Quantify the cost of inaction.** Every slide that highlights a risk should also communicate what happens if nothing is done.
- **Use analogies where appropriate.** Dependency drift is like deferred maintenance on a building — acceptable in the short term, expensive when ignored.
- **Avoid acronyms without explanation.** TFM, LTS, STS, CVE, EOL — spell them out or replace them with plain language on audience-facing slides.
- **Confidence, not alarm.** The report is a tool for informed decision-making, not a crisis report. Frame findings as actionable, not catastrophic.
- **One insight per slide.** Dense technical tables belong in speaker notes or appendices; the audience slide should carry a single clear message.

---

# MARP Specifics

- All slides must be written in Markdown format and use the [Marp](https://marp.app/) framework.
- Each slide deck must include the following frontmatter:
  ```yaml
  ---
  marp: true
  theme: custom-default
  paginate: true
  math: mathjax
  ---
  ```
- Use HTML comments (`<!-- -->`) for speaker notes.
- Use local directives for slide-specific settings:
  - `<!-- _class: lead -->` for title slides
  - `<!-- _class: invert -->` for dark slides
  - `<!-- _class: columns -->` for two-column layout
  - `<!-- _class: small -->` for dense content
  - `<!-- _paginate: skip -->` to hide page number
- Include the Mermaid script tag when using diagrams:
  ```html
  <script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
    mermaid.initialize({ startOnLoad: true });
  </script>
  ```
- Store slide decks in the `slides/` directory.
- Store custom themes in `slides/themes/`.
- Store images in `slides/img/`.
- Use CSS variables for theme customization (see `slides/themes/custom-default.css`).
- Ensure compatibility with Marp CLI v4 for PDF and presentation generation.
