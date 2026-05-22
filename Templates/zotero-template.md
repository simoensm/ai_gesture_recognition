---
title: "{{title}}"
authors: "{{authors}}"
year: {{date | format("YYYY")}}
journal: "{{publicationTitle}}"
citekey: {{citekey}}
tags: [literature, {{collections | join(", ")}}]
status: unread
---


## Metadata
- **Authors**: {{authors}}
- **Year**: {{date | format("YYYY")}}
- **Journal**: {{publicationTitle}}
- **DOI**: {{DOI}}

## Abstract
{{abstractNote}}

## Highlights & Annotations
{% for annotation in annotations %}
> {{annotation.annotatedText}}
*(p. {{annotation.page}})*
{% endfor %}

## My Notes
*Add synthesis here*

## Links
- [Open in Zotero]({{desktopURI}})