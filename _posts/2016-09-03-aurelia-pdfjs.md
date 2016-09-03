---
uid: 04123a23-7815-4619-a258-65754e6c8a16
layout: post
title: 'Adventures in Aurelia: Creating a PDF viewer'
description: ""
date: 2016-09-03
tags: aurelia pdfjs
published: true
authorid: jahyoung
author: Jedd Ahyoung
---

```
Outline:
-Intro
-Overview: The Goal
-Introducing PDF.JS
-The implementation
	-Creating an Aurelia custom element
	-Integrating PDF.JS
		-Web worker integration.
		-Scrolling
		-Improving performance
	-The final result
-Moving forward: analysis and next steps
	-Creating a plugin
	-Virtual scrolling & performance optimization
	-Better error handling, fallback implementation, etc.
-Resources
	-Docs - Aurelia.io/hub
	-gitter.im/aurelia/discuss
	-stackoverflow aurelia

	-PDFJS
```

Handling PDF files within a web application has always been painful to deal with. If you're lucky, you only have to serve the file. If you aren't, you have to integrate PDF document handling into your application logic. In the past, I've been lucky, but this time, I wasn't; users of our application needed to be able to view multiple documents within the application and save metadata related to individual pages of the PDF itself. In the past, one might accomplish this with an expensive PDF plugin, such as Adobe Reader, running inside the browser. However, I found a better way. Today, we'll take a look a how we can simplify PDF handling, using [Aurelia](http://aurelia.io) and [PDF.JS]().

## Overview: The Goal

Our goal, today, is to build a PDF viewer component in Aurelia that allows two-way data flow between the viewer and our application. We want to be able to bind viewer properties to properties in our application. We want the user to be able to scroll through the PDF document, and zoom in and out, with decent performance. Finally, we want this viewer to be a reusable component; we want to be able to drop multiple viewers into our application simultaneously with little effort.

## Introducing PDF.JS

PDF.JS is a JavaScript library, written by the Mozilla Foundation, that loads PDF documents, parses the file and associated metadata, and renders page output to the DOM. The default viewer included with the project powers the embedded PDF viewer in Chrome and Firefox, and can be used as a standalone page or as a resource (embedded within an iframe).

The problem here is that the default viewer, while it has a lot of functionality, is designed to work as a standalone web page. This means that while it can be integrated within a web application, it essentially would have to operate inside an iframe sandbox. In order to integrate this with an Aurelia web application - complete with event handling and two-way binding - we need to create an Aurelia custom element.

## Building the element

## Improving performance

## The final result

## Moving forward - next steps