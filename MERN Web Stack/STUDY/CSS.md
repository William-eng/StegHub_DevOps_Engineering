## What is Cascading Style Sheets (CSS)?
Cascading Style Sheets (CSS) is a stylesheet language used to describe the presentation of a document written in HTML 
or XML (including XML dialects like SVG or XHTML). CSS is used to control the layout, color, fonts, and overall visual appearance
of web pages. By separating content from design, CSS helps maintain consistency and reduces the amount of code needed to format
a webpage.

## Key Uses of CSS
1. Styling Web Pages: CSS is used to style HTML elements, such as text, images, and layout components.This includes setting colors, fonts, spacing, and positioning.
2. Layout Management: CSS allows for the creation of complex layouts using techniques like Flexbox and Grid. This helps in designing responsive and adaptable web pages that work across different devices and screen sizes.
3. Responsive Design: CSS media queries enable the design of web pages that adjust their layout and styling based on the screen size, orientation, and resolution of the device.
4. Animations and Transitions: CSS provides properties for creating animations and transitions, adding dynamic effects and interactivity to web elements.
5. Design Consistency: By defining styles in a separate CSS file, you can ensure a consistent look and feel across multiple pages of a website.

### Basic Syntax
CSS syntax consists of selectors and declaration blocks. Here’s a breakdown:

### Selector: Targets the HTML elements you want to style.

Declaration Block: Contains one or more declarations, each consisting of a property and a value, enclosed in curly braces {}.


    selector {
      property: value;
    }
    Example
    
    /* CSS to style a paragraph element */
    p {
      color: blue; /* Text color */
      font-size: 16px; /* Font size */
      line-height: 1.5; /* Line height */
      margin: 10px; /* Margin around the element */
      padding: 5px; /* Padding inside the element */
    }
