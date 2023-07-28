---
layout: post
title:  "Maven pom file support in vscode-xml"
date:   2023-07-28 09:00:00 -0400
categories: vscode xml maven
---

There has been a lot of discussion on [using vscode-xml (with lemminx-maven)](https://github.com/microsoft/vscode-maven/issues/47) to improve the `pom.xml` editing experience. There were various concerns (eg. performance, size, benefit, support) . I wanted to follow up with a comment advocating for the use of vscode-xml but pretty soon my "comment" started to look more like an article, so here we are.

I'd like to attempt to summarize what I see as clear advantages to depending directly on [vscode-xml](https://github.com/redhat-developer/vscode-xml/) from [vscode-maven](https://github.com/microsoft/vscode-maven/). These features are not just optional nice-to-haves but necessary to delivering a good user experience.

This will be split into support coming directly from vscode-xml, and support coming from [lemminx-maven](https://github.com/eclipse/lemminx-maven/), which is an extension contribution into the language server (LemMinX) that is provided by vscode-xml.


### vscode-xml (LemMinX)

The XML support for VS Code has its own [documentation](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#xml-features), but the examples are not pom file specific, so I thought I'd recreate some of the examples to drive the point.

#### [Formatting](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#formatting)
- Being able to easily format a pom file is an extremely useful feature, and the extension makes it very easy. It also provides **a lot** of ways to [customize the formatting](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Formatting.md)

![format-selected-element](/assets/vscode-xml-pomfile-support/format-selected-element.gif)

#### [Navigation](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#symbols-from-outline-and-breadcrumbs)
- pom files can be fairly large. Jumping to a particular element immediately is nice. (<kbd>ctrl</kbd>+<kbd>o</kbd> & start typing in the outline)

![xml-outline-navigation](/assets/vscode-xml-pomfile-support/xml-outline-navigation.gif)

#### Typing related features
  - [Syntax validation](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#syntax-validation)
  - [XML Tag Auto Close](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#xml-tag-auto-close)
  - [Auto Rename Tag](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#auto-rename-tag)
  - [Context aware completion](https://github.com/redhat-developer/vscode-xml/blob/main/docs/Features/XMLFeatures.md#completion-based-on-xsd)
    - vscode-xml knows about the schema that backs pom files, so it can make suggestions about what elements are valid for use.
    ![completion-context-aware](/assets/vscode-xml-pomfile-support/completion-context-aware.png)


### lemminx-maven extension

lemminx-maven still needs a nice way of integrating into vscode-xml. I got it working locally by cloning the repository, running `mvn clean verify -Pgenerate-vscode-jars -DskipTests`, and then ensuring it's detected by vscode-xml using :
```json
"xml.extension.jars": [
  "$HOME/git/lemminx-maven/lemminx-maven/target/vscode-lemminx-maven-jars/*.jar"
]
```

#### Hover for dependencies, configuration & properties

- A minor feature, but a convenient way to get documentation on particular elements

![hover-plugin-configuration-element](/assets/vscode-xml-pomfile-support/hover-plugin-configuration-element.png)

#### Go to definition on artifacts

![definition-artifact.](/assets/vscode-xml-pomfile-support/definition-artifact.gif)

#### Completions on phase, goals, & properties

![completion-properties](/assets/vscode-xml-pomfile-support/completion-properties.png)

#### Context aware completions
- lemminx-maven takes context aware completions a step further by using the plugin metadata to make correct suggestions (eg. goals and configuration elements)

![completion-artifactid](/assets/vscode-xml-pomfile-support/completion-artifactid.png)

![completion-plugin-configuration-elements](/assets/vscode-xml-pomfile-support/completion-plugin-configuration-elements.gif)

Overall, the completion support seems very similar in vscode-maven & lemminx-maven. vscode-maven is slightly more polished in showing a bit more information in the descriptions. lemminx-maven on the other hand supports completion of an entire `<dependency>..</dependency>` element even within `<dependencies>` tags (same for plugins).

I hope this fairly brief post has highlighted the power of vscode-xml as well as its extensibility.
