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
-More Resources
	-Docs - Aurelia.io/hub
	-gitter.im/aurelia/discuss
	-stackoverflow aurelia

	-PDFJS
```

Handling PDF files within a web application has always been painful to deal with. If you're lucky, your users only need to download the file. Sometimes, though, your users need more. In the past, I've been lucky, but this time, our users needed our application to display a PDF document so they could save metadata related to each individual page. In the past, one might have accomplished this with an expensive PDF plugin, such as Adobe Reader, running inside the browser. However, I found a better way. Today, we'll take a look a how we can simplify PDF handling, using [Aurelia](http://aurelia.io) and [PDF.JS]().

## Overview: The Goal

Our goal, today, is to build a PDF viewer component in Aurelia that allows two-way data flow between the viewer and our application. We have three main requirements.

 1. We want the user to be able to load the document, scroll, and zoom in and out, with decent performance.
 1. We want to be able to two-way-bind viewer properties (such as the current page, and the current zoom level) to properties in our application.
 1. We want this viewer to be a reusable component; we want to be able to drop multiple viewers into our application simultaneously with no conflicts and little effort.

Let's get started.

## Introducing PDF.JS

PDF.JS is a JavaScript library, written by the Mozilla Foundation. It loads PDF documents, parses the file and associated metadata, and renders page output to a DOM node (typically a `canvas` element). The default viewer included with the project powers the embedded PDF viewer in Chrome and Firefox, and can be used as a standalone page or as a resource (embedded within an iframe).

Here's an example of the viewer, embedded as an iframe, with a sample document. (You can also see it in action as a full page [here](https://mozilla.github.io/pdf.js/).)

<iframe style="width: 100%; height: 500px; margin-bottom: 1em;" src="https://mozilla.github.io/pdf.js/web/viewer.html"></iframe>


This is, admittedly, pretty cool. The problem here is that the default viewer, while it has a lot of functionality, is designed to work as a standalone web page. This means that while it can be integrated within a web application, it essentially would have to operate inside an iframe sandbox. The default viewer is designed to take configuration input through its query string, but we can't change configuration easily after the initial load, and we can't easily get info and events from the viewer. In order to integrate this with an Aurelia web application - complete with event handling and two-way binding - we need to create an Aurelia custom component.

## The implementation

To accomplish our goals, we're going to create an Aurelia custom element. However, we're not going to drop the default viewer into the element. Instead, we're going to create our own viewer that hooks into PDF.JS, so that we can have more control over our properties and our rendering.

### Setting up the sample project

For our initial proof-of-concept, we'll start with a skeleton Aurelia application. Download the starter kit from aurelia.io and build the project.

### Creating an Aurelia custom element

 1. Create two files: pdf-viewer.html and pdf-viewer.js.
 1. Basic custom element lifecycle methods, code sample.
 1. Creating bindable properties.

### Integrating PDF.JS

PDF.JS is split into three parts: the core library, the display library, and the web viewer plugin. For our purposes, we'll be using the core and display portions of the library; we'll be building our own viewer. PDF.JS has some examples for creating a basic viewer here and here. We'll build upon these examples for our custom component.

#### Web worker integration

PDF.JS uses a web worker, meaning that we have to load the file. Aurelia provides a loader abstraction so that we don't have to reference a static filepath (which could change if we bundle our application).

#### Implementing Scrolling

To implement scrolling, we'll use Aurelia's basic templating functionality, repeating a canvas element for each page in the PDF. When the PDF loads, we'll set the width and height of each canvas element based on the PDF page viewport.

When we scroll, we want to update the current page. We do that in our `pageChanged` property handler.

#### Implementing Zooming

When we zoom, we want to update the current zoom level. We do that in our `scaleChanged` property handler.

#### Improving performance

We'll use Aurelia's binding behaviors to improve performance, only rendering pages after scrolling has stopped.

### The final result
Let's review our target goals:

1. We want the user to be able to load the document, scroll, and zoom in and out, with decent performance.
1. We want to be able to two-way-bind viewer properties (such as the current page, and the current zoom level) to properties in our application.
1. We want this viewer to be a reusable component; we want to be able to drop multiple viewers into our application simultaneously with no conflicts and little effort.

The final result is the code here (http://www.github.com/jedd-ahyoung/aurelia-pdfjs-example) and the demonstration here (http://www.jedd-ahyoung.com/aurelia-pdfjs-example). While there is room for improvement, we've hit our target goals.

## Moving forward: analysis and next steps

There's always room for improvement, and it's always a good practice to do a post-mortem and analyze areas of improvement. These are some things that I'd like to improve in terms of the PDF viewer implementation:

### Individual page components

Currently, this proof-of-concept only allows for a scrolling viewport. Ideally, we'd be able to render any page anywhere, even outside of the viewer - for instance, generating PDF thumbnails as individual elements. Creating a `pdf-page` custom element or something along those lines could provide this functionality, while the viewer could simply use these elements via composition.

### Virtual scrolling and performance optimization

Aurelia provides a virtual repeater which vastly improves performance for very large datasets by only rendering a subset of the elements in the DOM. Ideally, the PDF viewer would be able to incorporate this for improved performance (to avoid having thousands of canvases in the DOM, which really hurts performance). This optimization, in conjunction with the individual page components, could really make a huge difference with large documents.

### Creating a plugin

Aurelia provides a plugin system. Converting this proof-of-concept into an Aurelia plugin would make it a drop-in resource for any Aurelia application.

## More Resources

For more resources for Aurelia, check out the following:

 - Aurelia documentation: http://aurelia.io/hub
 - Aurelia chat: http://www.gitter.im/aurelia/discuss
 - Q&A on Aurelia: http://www.stackoverflow.com/aurelia

If you're interested in using PDF.JS, these resources are good to start with:

 - PDFJS homepage: https://mozilla.github.io/pdf.js/
 - PDFJS documentation:
 - PDFJS github:
 - PDFJS examples:
