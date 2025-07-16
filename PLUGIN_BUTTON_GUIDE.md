# 🔘 WinTool Plugin Button Guide

## Overview

This guide explains how buttons work in WinTool plugins and the critical patterns you must follow for them to function properly.

## 🚨 Critical Requirements

### 1. Container-Based Element Selection

**❌ WRONG - This won't work:**
```javascript
const button = document.getElementById('my-button');
```

**✅ CORRECT - Use container scoping:**
```javascript
const button = container.querySelector('#my-button');
```

### 2. Proper Plugin Initialization Pattern

**❌ WRONG - Don't use DOMContentLoaded:**
```javascript
document.addEventListener('DOMContentLoaded', function() {
    // This pattern doesn't work for plugins
});
```

**✅ CORRECT - Initialize immediately:**
```javascript
console.log('=== My Plugin JavaScript loaded! ===');

// Find the container
let container = null;
if (typeof tabContainer !== 'undefined') {
    container = tabContainer;
}
if (!container) {
    container = document.querySelector('[data-tab="my-plugin"]');
}

if (container) {
    initializePlugin(container);
}
```

## 🔧 Button Implementation Methods

### Method 1: addEventListener (Recommended)

```javascript
function initializePlugin(container) {
    const actionBtn = container.querySelector('#action-btn');
    
    if (actionBtn) {
        actionBtn.addEventListener('click', async function() {
            console.log('Button clicked!');
            // Your button logic here
        });
    }
}
```

### Method 2: onclick in HTML

```html
<button id="my-btn" onclick="handleButtonClick()">Click Me</button>
```

```javascript
// Make function globally available
window.handleButtonClick = function() {
    console.log('Button clicked via onclick!');
    // Your button logic here
};
```

## 📋 Complete Working Example

### HTML (index.html)
```html
<div class="plugin-container">
    <div class="plugin-header">
        <h2>My Plugin</h2>
    </div>
    
    <div class="plugin-content">
        <input type="text" id="user-input" placeholder="Enter something...">
        <button id="action-btn" class="btn btn-primary">
            <i class="fas fa-play"></i> Execute
        </button>
        
        <div id="result-area" style="display: none;">
            <h4>Result:</h4>
            <div id="result-content"></div>
        </div>
    </div>
</div>
```

### JavaScript (script.js)
```javascript
console.log('=== My Plugin JavaScript loaded! ===');

// Find the container
let container = null;
if (typeof tabContainer !== 'undefined') {
    container = tabContainer;
}
if (!container) {
    container = document.querySelector('[data-tab="my-plugin"]');
}

if (container) {
    initializePlugin(container);
}

function initializePlugin(container) {
    const actionBtn = container.querySelector('#action-btn');
    const userInput = container.querySelector('#user-input');
    const resultArea = container.querySelector('#result-area');
    const resultContent = container.querySelector('#result-content');
    
    if (actionBtn) {
        actionBtn.addEventListener('click', async function() {
            const inputValue = userInput.value.trim();
            
            if (!inputValue) {
                alert('Please enter a value');
                return;
            }
            
            try {
                // Disable button during processing
                actionBtn.disabled = true;
                actionBtn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> Processing...';
                
                // Process the input
                const result = processInput(inputValue);
                
                // Display result
                displayResult(result, container);
                
            } catch (error) {
                console.error('Error:', error);
                alert('An error occurred: ' + error.message);
            } finally {
                // Re-enable button
                actionBtn.disabled = false;
                actionBtn.innerHTML = '<i class="fas fa-play"></i> Execute';
            }
        });
    }
    
    // Signal that plugin is ready
    if (window.markTabAsReady && typeof tabId !== 'undefined') {
        window.markTabAsReady(tabId);
    }
}

function processInput(input) {
    // Your processing logic here
    return {
        input: input,
        processed: input.toUpperCase(),
        timestamp: new Date().toISOString()
    };
}

function displayResult(result, container) {
    const resultArea = container.querySelector('#result-area');
    const resultContent = container.querySelector('#result-content');
    
    if (resultContent) {
        resultContent.innerHTML = `
            <div><strong>Input:</strong> ${result.input}</div>
            <div><strong>Result:</strong> ${result.processed}</div>
            <div><strong>Time:</strong> ${result.timestamp}</div>
        `;
        resultArea.style.display = 'block';
    }
}
```

## 🐛 Common Issues & Solutions

### Issue: "Button does nothing when clicked"

**Cause:** Using `document.getElementById()` instead of container scoping

**Solution:** Always use `container.querySelector()`

### Issue: "Cannot read property 'addEventListener' of null"

**Cause:** Element not found due to incorrect selector or timing

**Solution:** 
1. Check that your HTML has the correct ID
2. Ensure you're using container scoping
3. Add null checks: `if (button) { ... }`

### Issue: "Plugin doesn't load"

**Cause:** Not following the initialization pattern

**Solution:** Use the container-finding pattern shown above

## 🎯 Best Practices

1. **Always use container scoping** for element selection
2. **Add console.log statements** for debugging
3. **Check for null elements** before adding event listeners
4. **Disable buttons during processing** to prevent double-clicks
5. **Provide user feedback** with loading states
6. **Handle errors gracefully** with try-catch blocks
7. **Call markTabAsReady()** when initialization is complete

## 🚀 Using the CLI

The WinTool Plugin CLI now generates plugins with the correct button patterns:

```bash
# Create a new plugin with working buttons
node cli/wintool-plugin-cli.js create my-plugin --type=basic --author="Your Name"
```

The generated plugin will include:
- ✅ Proper container scoping
- ✅ Working button event listeners
- ✅ Correct initialization pattern
- ✅ Debugging console logs
- ✅ Error handling examples

---

**Remember:** WinTool plugins work like tabs, not standalone web pages. Always follow the container-based patterns for reliable functionality!
