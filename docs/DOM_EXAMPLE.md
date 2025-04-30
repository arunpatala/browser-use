# DOM Representation Example

This document shows a concrete example of how an HTML page is represented in the context sent to the LLM, based on the actual implementation in Browser-Use.

## Original HTML Page

```html
<!DOCTYPE html>
<html>
<head>
    <title>Online Store</title>
    <style>
        .product-card {
            border: 1px solid #ccc;
            padding: 10px;
            margin: 10px;
        }
        .btn-primary {
            background-color: #007bff;
            color: white;
            padding: 5px 10px;
        }
    </style>
</head>
<body>
    <header>
        <nav>
            <div class="search-container">
                <input type="text" placeholder="Search products..." id="search-input">
                <button type="submit" class="search-btn">Search</button>
            </div>
            <a href="/cart" class="cart-link">Cart (2 items)</a>
        </nav>
    </header>

    <main>
        <h1>Featured Products</h1>
        
        <div class="product-card">
            <h2>Organic Apples</h2>
            <p>Fresh organic apples from local farms</p>
            <p class="price">$2.99/lb</p>
            <button class="btn-primary">Add to Cart</button>
        </div>

        <div class="product-card">
            <h2>Fresh Milk</h2>
            <p>Whole milk, 1 gallon</p>
            <p class="price">$3.49</p>
            <button class="btn-primary">Add to Cart</button>
        </div>

        <div class="product-card">
            <h2>Whole Grain Bread</h2>
            <p>Freshly baked whole grain bread</p>
            <p class="price">$4.99</p>
            <button class="btn-primary">Add to Cart</button>
        </div>
    </main>

    <footer>
        <p>© 2024 Online Store. All rights reserved.</p>
    </footer>
</body>
</html>
```

## DOM Processing Steps

1. **Element Selection (buildDomTree.js):**
   ```javascript
   // Elements are selected based on:
   - isInteractiveElement(element)  // Checks if element is clickable
   - isElementVisible(element)      // Checks if element is visible
   - isTopElement(element)          // Checks if element is not hidden
   - isInExpandedViewport(element)  // Checks if element is in viewport
   ```

2. **Element Processing (DomService):**
   ```python
   # Each element is processed into a DOMElementNode with:
   - tag_name: str
   - xpath: str
   - attributes: Dict[str, str]
   - is_interactive: bool
   - is_top_element: bool
   - is_in_viewport: bool
   - highlight_index: Optional[int]
   ```

3. **Context Building (clickable_elements_to_string):**
   ```python
   # Elements are formatted with:
   - Sequential indices
   - Essential attributes only
   - Text content from get_all_text_till_next_clickable_element()
   - Optimized attribute selection
   ```

## DOM Representation Sent to LLM

### Without Vision

```text
[Start of page]
[0]<input type='text' placeholder='Search products...' id='search-input' />
[1]<button type='submit' class='search-btn'>Search</button>
[2]<a href='/cart' class='cart-link'>Cart (2 items)</a>
Featured Products
[3]<h2>Organic Apples</h2>
Fresh organic apples from local farms
$2.99/lb
[4]<button class='btn-primary'>Add to Cart</button>
[5]<h2>Fresh Milk</h2>
Whole milk, 1 gallon
$3.49
[6]<button class='btn-primary'>Add to Cart</button>
[7]<h2>Whole Grain Bread</h2>
Freshly baked whole grain bread
$4.99
[8]<button class='btn-primary'>Add to Cart</button>
© 2024 Online Store. All rights reserved.
[End of page]
```

### With Vision

```text
[Start of page]
[0]<input type='text' placeholder='Search products...' id='search-input' />
[1]<button type='submit' class='search-btn'>Search</button>
[2]<a href='/cart' class='cart-link'>Cart (2 items)</a>
Featured Products
[3]<h2>Organic Apples</h2>
Fresh organic apples from local farms
$2.99/lb
[4]<button class='btn-primary'>Add to Cart</button>
[5]<h2>Fresh Milk</h2>
Whole milk, 1 gallon
$3.49
[6]<button class='btn-primary'>Add to Cart</button>
[7]<h2>Whole Grain Bread</h2>
Freshly baked whole grain bread
$4.99
[8]<button class='btn-primary'>Add to Cart</button>
© 2024 Online Store. All rights reserved.
[End of page]

Screenshot: [base64_encoded_image]
```

## Implementation Details

1. **Element Selection Criteria:**
   - Must be interactive (button, link, input, etc.)
   - Must be visible in the viewport
   - Must not be hidden by other elements
   - Must have a distinct interaction area

2. **Attribute Processing:**
   - Only attributes from `include_attributes` list are included
   - Default attributes: title, type, name, role, aria-label, placeholder, value, alt, aria-expanded
   - Redundant attributes are removed (e.g., role if same as tag)
   - aria-label is removed if same as text content
   - placeholder is removed if same as text content

3. **Text Content Processing:**
   - Text is collected using `get_all_text_till_next_clickable_element()`
   - Content is cleaned and formatted
   - Hierarchical structure is preserved
   - Text content is interleaved with interactive elements for context

4. **Performance Optimizations:**
   - Caches computed styles and bounding rectangles
   - Minimizes DOM traversals
   - Uses efficient element location strategies
   - Handles special cases (file uploads, etc.)

## How the LLM Uses This Information

When the LLM needs to interact with the page, it can:

1. **Reference Elements by Index:**
   ```python
   # Add Organic Apples to cart
   action = ClickElementAction(index=4)  # LLM uses the text content "Organic Apples" to identify the correct button
   
   # Add Fresh Milk to cart
   action = ClickElementAction(index=6)  # LLM uses the text content "Fresh Milk" to identify the correct button
   
   # Add Whole Grain Bread to cart
   action = ClickElementAction(index=8)  # LLM uses the text content "Whole Grain Bread" to identify the correct button
   ```

2. **Understand Element Context:**
   - Uses the interleaved text content to understand which button corresponds to which product
   - Makes decisions based on product information in the text content
   - Verifies actions against product details in the text content

3. **Make Informed Decisions:**
   - Chooses the correct element based on the text content around it
   - Verifies element existence before acting
   - Handles multiple similar elements by using the surrounding text content for context

This representation provides the LLM with the necessary information to interact with the page, using the interleaved text content to understand the context of each element. 