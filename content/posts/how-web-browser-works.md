+++
date = '2025-06-10T08:11:23+07:00'
draft = false
title = 'How Web Browser Works'
author = 'ahmad'
categories = ['Insight']
featuredImage = '/images/how-web-browser-works/featured.png'
+++

This article covers the basic concepts of how a web browser works from a high level perspective. We will talk about how the browser requests a web page until it is displayed on the screen.

This article is intended for those who are new to web development and want to understand how web browsers work.

Why this knowledge is important?

- It helps you understand how web browsers work, which can help you debug issues in your web application.
- It helps you understand how web browsers render web pages, which can help you optimize your web application for better performance.
- It helps you understand how web browsers handle security, which can help you secure your web application.
- It helps you understand how web browsers handle user input, which can help you create better user experiences.
- Sometimes, interviewers ask this question to test your knowledge of web development. (Yes, I was asked this question in an interview once.)

## How Web Browser Works

The over-simplified flow of how a web browser works is like this:
```
[HTML string download]
      ↓
[Parse HTML → DOM]
      ↓
[Download & parse CSS → CSSOM]
      ↓
[Build render tree]
      ↓
[Layout calculation]
      ↓
[Paint & composite → Rendered page]
```

We will explain each step in detail below, plus some additional steps that are not shown in the flow above, such us how web browsers communicate with web servers.

### 1. Navigation to a URL

This is the first step when user interacts with a website URL (e.g. by clicking a link or entering a URL in the address bar).

#### DNS Lookup

When a user navigates to a URL (let's say `https://example.com`), the browser first needs to resolve the domain name to an IP address. This is done through a process called DNS (Domain Name System) lookup. The browser will query a DNS server to get the IP address associated with the domain name.

#### TCP Connection

Once the IP address is resolved, the browser establishes a TCP (Transmission Control Protocol) connection to the web server. This involves a three-way handshake process to ensure a reliable connection.

#### TLS Handshake (if using HTTPS)

If the URL uses HTTPS, the browser will perform a TLS (Transport Layer Security) handshake to establish a secure connection with the web server. This involves exchanging encryption keys and verifying the server's SSL certificate.

### 2. Sending HTTP Request

After establishing a connection, the browser sends an HTTP request to the web server. This request includes information such as the requested resource (e.g., HTML file), HTTP method (GET, POST, etc.), headers, and cookies.

### 3. Receiving HTTP Response

The web server processes the request and sends back an HTTP response. This response includes a status code (e.g., 200 for success, 404 for not found), headers, and the requested resource (e.g., HTML content).

### 4. Parsing HTML

#### Building the DOM

Once the browser receives the HTML content, it starts parsing it. The HTML is converted into a DOM (Document Object Model) tree structure, which represents the document's structure and content.

#### Preloading Resources

While parsing the HTML, the browser may encounter links to external resources such as CSS files, JavaScript files, images, etc. The browser will start preloading these resources in parallel to improve performance.
```html
<link rel="stylesheet" href="styles.css" />
<script src="my-script.js" async></script>
<img src="my-image.jpg" alt="image description" />
<script src="another-script.js" async></script>
```

#### Building the CSSOM

When the browser encounters a `<link>` tag for a CSS file, it sends an HTTP request to fetch the CSS file. Once the CSS file is received, the browser parses it and builds a CSSOM (CSS Object Model) tree, which represents the styles applied to the document.

#### JavaScript Execution

If the HTML contains `<script>` tags, the browser will pause parsing the HTML until the JavaScript is executed. The JavaScript can manipulate the DOM and CSSOM, which may trigger reflows or repaints.

### 5. Rendering the Page

Rendering the page involves three main processes: **style**, **layout**, and **paint**.

#### Style Calculation

The browser combines the DOM and CSSOM to determine the styles applied to each element. This process is called style calculation. The browser computes the final styles for each element based on the CSS rules and inline styles.

#### Layout Calculation

After calculating the styles, the browser performs layout calculation. This involves determining the size and position of each element on the page based on the styles applied. The result is a layout tree that represents the visual structure of the page.

#### Paint

Once the layout is calculated, the browser paints the pixels on the screen. This involves converting the layout tree into actual pixels, applying styles such as colors, backgrounds, borders, etc. The painted pixels are then composited to form the final rendered page.

## Summary
In summary, a web browser works by:
```
[HTML string download]
      ↓
      ↓
[Parse HTML → DOM]
      ├─ Tokenization
      ├─ Create DOM nodes
      └─ Build DOM tree
      ↓
[Download & parse CSS → CSSOM]
      ├─ Download CSS files
      ├─ Parse CSS rules
      └─ Build CSSOM tree
      ↓
[Build render tree]
      ├─ Combine DOM & CSSOM
      ├─ Filter non-visual elements
      └─ Calculate computed styles
      ↓
[Layout calculation]
      ├─ Calculate box dimensions
      ├─ Determine positions
      └─ Handle viewport constraints
      ↓
[Paint & composite → Rendered page]
      ├─ Create layer tree
      ├─ Paint each layer
      └─ Composite layers
```

This is a process that we must understand on how web browsers work. It helps us understand how web pages are rendered and how we can optimize our web applications for better performance.

## Additional Resources
- [Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work)
- [How Browsers Work: Behind the scenes of modern web browsers](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)
