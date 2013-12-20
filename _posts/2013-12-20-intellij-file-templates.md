---
layout: post
title:  "Intellij Idea - Live templates."
date:   2013-12-20 22:00:00
categories: blog
tags: intellij ide
---

In Intellij Idea there is no default template for test cases. I write them with 'Given / When / Then' blocks like one below:

![GWT][live_templates_0]

Writting them manually for each test would be just a waste of time. Hopefully in Intellij Idea you can create custom templates, called live templates, that enables pasting such code snippet in no time.

- Press Ctrl+Shift+A (or Cmd+Shift+A on Mac)
- Type 'Live templates'
- In Live templates section press add button

![Press Add Button][live_templates_1]

- Fill in abbreviation, description and template text:

![Fill form][live_templates_2]

- Set where templates should be available using 'Change' link in the bottom - mark Java / Declaration:

![Mark Java][live_templates_3]


and you're done. Now template is ready to use.

Just press Ctrl+J (Cmd+J) and type 'gwt' abbreviation:

![GWT][live_templates_4]

Hit enter and voila!

![Result][live_templates_5]


[live_templates_0]: /gfx/content/idea_live_templates_0.png "GWT"
[live_templates_1]: /gfx/content/idea_live_templates_1.png "Press Add Button"
[live_templates_2]: /gfx/content/idea_live_templates_2.png "Fill form"
[live_templates_3]: /gfx/content/idea_live_templates_3.png "Mark Java"
[live_templates_4]: /gfx/content/idea_live_templates_4.png "GWT"
[live_templates_5]: /gfx/content/idea_live_templates_5.png "Result"
