+++
date = '2025-07-30T07:01:52+01:00'
draft = false
slug = "docs-as-code"
title =  "Building a Centralized Documentation Hub"
categories = ["Blog"]
tags = ["Documentation", "C4"]
comments = true
keywords = ['docs as code', 'architecture diagrams']
+++


Recently, I wanted to refactor a centralized documentation hub I had put in place around 2 years ago, following the 'Docs as Code' principle. At the time I thought what I had done would work really well, but it was never widely adopted, mainly due to tooling limitations. At that time there wasn't any other viable options, particularly with diagramming, and IDE extensions (VS Code) were non-existent.

This small series of posts describes how I went about satisfying this requirement which will hopefully be useful to anyone else looking to do something similar, as it's a common issue I have come across in every place I've worked.

Drop any questions or comments at the bottom of the page.

## Requirements

As a brief outline of the requirements, I decided to revisit the user story and see if I could implement anything better. Here's a rough user story for implementation:


> **As a** developer  
> **I want** to be able to document how a system works  
> **And I want** to be able to construct architecture diagrams that link to other projects mine depends on  
> **So that** other team members can understand the system more easily  
> **And** complete future work on it more efficiently  
> **And** I want to be able to do this in a single place

Let's look next at how to fulfil this criteria.

## Phase 1: Conventions

The first thing you'll want to determine is a project folder structure that both new and existing projects can easily integrate.

I highly recommend encouraging the production of your docs as distinct areas. Another bonus to this structuring is being able to more easily restrict access to these areas by roles - useful for SSG in particular which by default will use folder structure navigation. Also having a containing folder makes production and deployment easier, as well as switching build tools at a later date if required.

### The Four Docs as Code Pillars
![Four Pillars](/images/four-pillars.jpg)

As we all love acronyms, here's one I use to describe the pillars of a successful docs as code implementation and the minimal standard you should strive for:

**TAGD:**

* Technical: Developer only notes. Think where secrets are stored, how to run apps locally, etc.
* ADR: Architecture Decision Records - Historical context of important architectural changes.
* Guides: End user guides, manuals, first line support guides/FAQ.
* Diagramming: Architecture and Workflow diagrams.

Taking the pillars defined above to a folder structure convention would look along the lines of:

```text
project/
    docs/               # Top level containing folder
        docs/           # All docs
            adr/        # Convention
            guides/     # Convention
            technical/  # Convention
            folder/     # You can add any other content any way you like with SSG
        diagrams/       # Diagrams
```

With that in place, we can now look at what tools we can use for production of these items.

## Phase 2: Toolset

After some research, I settled on the following which satisfies the story above and are widely used:

* [LikeC4][1] (Diagramming - Architecture)
* [adr-tools][2] (ADR / Architecture Decision Records)
* [Docusaurus][3] (SSG / Static Site Generator)
* Azure Static Web App (Hosting)
* [Markdown][4] (General doc production)
* [Mermaid][5] (Diagramming - Workflows, User Journeys etc.)

I'll briefly explain each of these below.

### Diagrams

I had used Structurizr as part of the previous attempt at team wide documentation adoption, but this failed due to some limitations it has:

* No IDE live editing - you had to use their web based DSL editor - which also had limitations - not able to view linked ADR, docs, etc.
* Paid subscription per DSL to be hosted there (Note: you can self host for free an on prem, but still doesn't solve the above)
* Diagram UI - though ok - wasn't that modern looking

The first step alone contributed as the main fail factor, as simple as it seems to use another window to do live DSL edits, it still was nonetheless a step outside the IDE nobody really wants to take.

#### LikeC4

[1]: https://likec4.dev/

LikeC4 was the answer to those limitations. It has an official VS Code extension, with live previews, and a much better UI, as well as being free. It also has multiple production options:

* SSG - Output all docs to a single static standalone website.
* React Components - Output to a react component you can then embed in another React app.

I tested both of these, and you can easily use either depending on your own criteria. If you wanted to host the diagrams separately, you could deploy the generated single website to it's own domain (diagrams.your-company.com etc.) which would be really straight forward.

For our purposes, I wanted to be able to embed these into and alongside the other documentation forms. This led me down the route of looking at having a parent React app that was able to display both the diagram components as well as the other docs written.

### ADR

[2]: https://github.com/npryce/adr-tools

`adr-tools` is a simple cli tool that generates ADR's for you in the standardized template format. This is important as other tools can utilize displaying these alongside diagrams, for example (LikeC4 currently does not support, but I expect it to eventually. Structurizr does support linking to ADRs).

### Docusaurus

[3]: https://docusaurus.io/

Docusaurus is another SSG, importantly React based which made it easy to choose to function as the parent app I needed to host all the docs including the diagrams. As the diagrams through LikeC4 were being built as a React component, it means that component can be integrated as a Page within Docusaurus.

It also supports displaying Mermaid diagrams (through an additional package and simple config) which we had quite a lot of for user journeys and user workflows.

### Azure Static Web Apps

This was another easy choice given the use of Docusaurus as an SSG so nothing complicated was required other than to simply host the static files. We were also able to use the built in auth for SSO via Entra ID so that ticked another box.

### Markdown

[4]: https://www.markdownguide.org/

Markdown is a widely adopted lightweight markup language used for formatting text, so is ideal for use in written docs.

### Mermaid

[5]: https://mermaid.ai/web/

Mermaid is a great tool supporting all sorts of diagrams - I have found the sequence, requirement, user journey, and mind maps to be most useful as great visual representations of flows through an application. There is also experimental support for LikeC4 as of writing, which I've not used yet.

### Composing documentation

Now we have our toolchain in place, we can proceed to integrating them into our development workflow.

---

In Part 2 I'll go through details of how I wired all this up and setup the deployment.
