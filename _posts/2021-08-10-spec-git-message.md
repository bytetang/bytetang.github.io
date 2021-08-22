---
title: "Write a canonical Git Commit Message"
author: Morse
date: 2021-08-10
categories: [Git]
tags: [Git]
math: true
mermaid: true
---

Templeted commits message is benefits to project maintain or printing the pretty change log. let's follow the better habit on daily development.

<h1>Best practice format</h1>

We can fllow the google angular spec:
[Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#)

So we aggreed the message format like this.

```
[<type>][<scope>]: <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

example:

```
[fix][CQ-2938] Fix small typo in docs  widget (tutorial instructions)

Rename the iVars to remove the common prefix.

https://celernetwork.atlassian.net/browse/CQ-3785
 
```

<h1>Add format tempate</h1>

`git config --local commit.template <PATH>`

Then use git commit without -m

![image](https://user-images.githubusercontent.com/6038077/130358497-7b76ce89-cdd3-4d24-b90f-65814d55bbb2.png)

you can see the full tips about template you set before.


<h1>Generate changes log(Pretty print)</h1>

```git log --oneline --decorate --color```

![image](https://user-images.githubusercontent.com/6038077/130358720-17e0249d-98c9-497c-9833-f217634fa678.png)


`--graph` shows you which branches the commits are on
`--since` fiter the start time, for example: 8/10/2020