## Basic HTML Form Structure
Here’s a simple form that collects a user's name, email, and message:

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
            <h2>Contact Us</h2>
            <form id="contactForm">
              <label for="name">Name:</label>
              <input type="text" id="name" name="name" placeholder="Enter your name" required>
        
              <label for="email">Email:</label>
              <input type="email" id="email" name="email" placeholder="Enter your email" required>
        
              <label for="message">Message:</label>
              <textarea id="message" name="message" placeholder="Your message" rows="5" required></textarea>
        
              <button type="submit">Submit</button>
            </form>
            <p id="responseMessage"></p>
          </div>
          
          <script src="script.js"></script>
        </body>
        </html>

## Adding CSS for Styling
Let's style the form to make it look more appealing using CSS:

      /* styles.css */
      body {
        font-family: Arial, sans-serif;
        background-color: #f4f4f9;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        margin: 0;
      }
      
      .form-container {
        background-color: #fff;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
        width: 300px;
      }
      
      h2 {
        text-align: center;
        color: #333;
      }
      
      form {
        display: flex;
        flex-direction: column;
      }
      
      label {
        margin-bottom: 5px;
        color: #333;
      }
      
      input, textarea {
        padding: 10px;
        margin-bottom: 15px;
        border: 1px solid #ccc;
        border-radius: 5px;
        font-size: 16px;
      }
      
      button {
        padding: 10px;
        background-color: #28a745;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
      }
      
      button:hover {
        background-color: #218838;
      }
      
      #responseMessage {
        color: #28a745;
        text-align: center;
        margin-top: 15px;
      }


## Adding Basic Form Validation and Response with JavaScript
To make the form functional, you can use JavaScript to handle form submissions and validate inputs:

      // script.js
      document.getElementById("contactForm").addEventListener("submit", function(event) {
        event.preventDefault(); // Prevent form from submitting normally
      
        const name = document.getElementById("name").value;
        const email = document.getElementById("email").value;
        const message = document.getElementById("message").value;
        const responseMessage = document.getElementById("responseMessage");
      
        // Simple validation
        if (name === "" || email === "" || message === "") {
          responseMessage.textContent = "Please fill out all fields.";
          responseMessage.style.color = "red";
          return;
        }
      
        // Simulate form submission
        responseMessage.textContent = "Thank you, " + name + "! Your message has been sent.";
        responseMessage.style.color = "green";
      
        // Reset the form
        document.getElementById("contactForm").reset();
      });

