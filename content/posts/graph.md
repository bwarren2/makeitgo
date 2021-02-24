---
title: "Interactive, inspectable AWS resource graphs"
date: 2020-12-01T17:45:26-05:00
draft: false
cytoscapeOn: true
mermaidOn: true
summary: "Rendering a Cloudformation resource graph in-browser"
---

I had a problem, so I made a solution.  This is a Cloudformation template, re-rendered as a Cytoscape graph with mouseovers to show the Cloudformation that produced that node and edges to show the dependencies between nodes.
{{<cyto id="bacd" file="graph.json" style="style.json" height="400" title="APIGateway API in CDK">}}

# The Problem

Cloudformation is a very powerful tool to declare what you want on AWS, and have them do the creation/deletion/management of those resources for you.  However, that means our flowchart for this process looks like this:

<div class="mermaid">
graph TD
	APIG[API Gateway] --> L[Lambda Tier]
</div>
<script async src="https://unpkg.com/mermaid@8.7.0/dist/mermaid.min.js"></script>
