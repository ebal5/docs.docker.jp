---
title: "Docker 用語"
description: "Docker に関して使われている用語集"
keywords: "glossary, docker, terms, definitions"
notoc: true
noratings: true
redirect_from:
- /engine/reference/glossary/
- /reference/glossary/
---
<!--
To edit/add/remove glossary entries, visit the YAML file at:
https://github.com/docker/docker.github.io/blob/master/_data/glossary.yaml

To get a specific entry while writing a page in the docs, enter Liquid text
like so:
{{ site.data.glossary["aufs"] }}
-->
<span id="glossaryMatch" />
<span id="topicMatch" />

## 用語一覧

用語の定義を参照する場合、またその用語がタグづけされたドキュメントを参照する場合は、以下の項目をクリックしてください。

{% for entry in site.data.glossary %}- [{{ entry[0] }}]
{% endfor %}

{% for entry in site.data.glossary %}[{{ entry[0] }}]: /glossary/?term={{ entry[0] }}
{: class="glossLink" data-content="{{ entry[1] | markdownify | strip_html | strip | truncatewords: 50, "..."}}" data-trigger="hover" id="popoverData{{ forloop.index }}" rel="popover" data-placement="bottom" data-original-title="用語定義: {{ entry[0]}}"}
{% endfor %}