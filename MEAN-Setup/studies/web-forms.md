# Editing simple Web Form with HTML, CSS and JavaScript

Creating a simple web form using HTML, CSS, and JavaScript involves three main steps: defining the structure with HTML, styling the form with CSS, and adding interactivity with JavaScript. Here’s a basic example to guide you through the process.

### Step 1: HTML Structure

First, let's create a basic HTML structure for the web form. This form will include fields for the user's name, email, and a message, along with a submit button.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Web Form</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="form-container">
        <form id="contactForm">
            <h2>Contact Us</h2>
            <div class="form-group">
                <label for="name">Name:</label>
                <input type="text" id="name" name="name" required>
            </div>
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>
            <div class="form-group">
                <label for="message">Message:</label>
                <textarea id="message" name="message" required></textarea>
            </div>
            <button type="submit">Submit</button>
        </form>
    </div>
    <script src="scripts.js"></script>
</body>
</html>
```

### Step 2: CSS Styling

Next, let's style the form using CSS. Create a file named `styles.css` and add the following styles:

```css
/* styles.css */
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
}

.form-container {
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 400px;
}

h2 {
    text-align: center;
    margin-bottom: 20px;
}

.form-group {
    margin-bottom: 15px;
}

label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
}

input[type="text"],
input[type="email"],
textarea {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
}

textarea {
    height: 100px;
    resize: vertical;
}

button {
    width: 100%;
    padding: 10px;
    border: none;
    border-radius: 4px;
    background-color: #28a745;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
}

button:hover {
    background-color: #218838;
}
```

### Step 3: JavaScript Interactivity

Finally, add interactivity with JavaScript. Create a file named `scripts.js` and add the following script:

```javascript
// scripts.js
document.addEventListener('DOMContentLoaded', function() {
    const form = document.getElementById('contactForm');

    form.addEventListener('submit', function(event) {
        event.preventDefault();

        // Get form values
        const name = document.getElementById('name').value;
        const email = document.getElementById('email').value;
        const message = document.getElementById('message').value;

        // Simple validation (more validation can be added as needed)
        if (!name || !email || !message) {
            alert('Please fill in all fields.');
            return;
        }

        // Display a success message (in a real application, you would send this data to a server)
        alert('Form submitted successfully!\n\n' +
              'Name: ' + name + '\n' +
              'Email: ' + email + '\n' +
              'Message: ' + message);

        // Reset the form
        form.reset();
    });
});
```

### Putting It All Together

Make sure all three files (`index.html`, `styles.css`, and `scripts.js`) are in the same directory. Open the `index.html` file in a web browser to see the form in action.

This example demonstrates how to create a simple web form, style it using CSS, and add basic interactivity with JavaScript. You can expand upon this foundation by adding more form fields, advanced validation, and backend integration as needed.