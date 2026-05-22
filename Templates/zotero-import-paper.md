---
title: "{{title}}"
authors: "{{authors}}"
year: {{date | format("YYYY")}}
journal: "{{publicationTitle}}"
citekey: {{citekey}}
tags: [literature, {{collections | join(", ")}}]
status: unread
doi: "{{DOI}}"
---

# {{title}}

## Metadata
- **Authors**: {{authors}}
- **Year**: {{date | format("YYYY")}}
- **Journal/Venue**: {{publicationTitle}}
- **DOI**: [{{DOI}}](https://doi.org/{{DOI}})
- **Zotero**: [Open in Zotero]({{desktopURI}})

## Abstract
{{abstractNote}}

## Highlights & Annotations
{% for annotation in annotations %}
> {{annotation.annotatedText}}

*(p. {{annotation.page}})* {% if annotation.comment %} — **Note**: {{annotation.comment}}{% endif %}

{% endfor %}

## Synthesis
> [!abstract] Core Claim
> *What is the central argument or contribution of this paper?*

**Method**: 

**Key Findings**:
- 

**Limitations**:
- 

**Relevance to my research**:

## Related Topics
*Link to [[Topics/]] notes*

## Related Papers
*Link to other [[Literature_Notes/]] notes*
