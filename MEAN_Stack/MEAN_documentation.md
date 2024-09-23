# MEAN Stack Deployment to Ubuntu in AWS

MEAN Stack is a combination of following components:
- MongoDB (Document database) - Stores and allows to retrieve data.
- Express (Back-end application framework) - Makes requests to Database for Reads and Writes.
- Angular (Front-end application framework) - Handles Client and Server Requests
- Node.js (JavaScript runtime environment) - Accepts requests and displays results to end user

## Project Task
In this assignment we are going to implement a simple Book Register web form using MEAN stack.

## STEP ONE: INSTALL NODEJS
We need to run all the package updates rewuired with the commands:

    sudo apt update
    sudo apt upgrade

Then for nodejs, we need to add certificates to install with the commands:

    sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install -y nodejs
    
![insatllNode](https://github.com/user-attachments/assets/c19ce06d-c89f-411b-81d1-22b4f6f99802)

## STEP TWO: INSTALL MONGODB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed 
over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.
We install with the command:

    sudo apt-get install -y gnupg curl
    curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
     sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
     --dearmor

    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
Next we install the MongoDB server, Start the server and confirm it's running

    sudo apt-get install -y mongodb-org
    sudo systemctl start mongod
    sudo systemctl daemon-reload
    sudo systemctl status mongod

![MongoDBInstall](https://github.com/user-attachments/assets/8c547f75-6209-40c1-ba73-d03d7f81f911)

![mongoserviceRunning](https://github.com/user-attachments/assets/47cf0816-97d7-4a70-9fc3-1c54296a2ca4)

Install [npm](https://www.npmjs.com) - Node package manager.

    sudo apt install -y npm
We need to install 'body-parser package. We need 'body-parser' package to help us process JSON files passed in requests to the server.

    sudo npm install body-parser

![parser](https://github.com/user-attachments/assets/3d1bfd35-16e9-4994-a43e-e1758e602307)

Create a folder named 'Books' and in the Books directory, Initialize npm project:

    npm init

![npm-init4](https://github.com/user-attachments/assets/b52c55f4-d17a-4a6f-ae59-47ab47706046)

Add a file to it named server.js 

      const express = require('express');
    const bodyParser = require('body-parser');
    const mongoose = require('mongoose');
    const path = require('path');
    
    const app = express();
    const PORT = process.env.PORT || 3300;
    
    // MongoDB connection
    mongoose.connect('mongodb://localhost:27017/test', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    })
    .then(() => console.log('MongoDB connected'))
    .catch(err => console.error('MongoDB connection error:', err));
    
    app.use(express.static(path.join(__dirname, 'public')));
    app.use(bodyParser.json());
    
    require('./apps/routes')(app);
    
    app.listen(PORT, () => {
      console.log(`Server up: http://localhost:${PORT}`);
    });

## STEP THREE: INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications.
We will use Express in to pass book information to and from our MongoDB database.
We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. 
We will use Mongoose to establish a schema for the database to store data of our book register. run the command:

    sudo npm install express mongoose

![install-moongose](https://github.com/user-attachments/assets/900d462f-1ffa-44fe-95dd-e3af95a3b325)

In 'Books' folder, create a folder named apps, in the apps folder, create a file named routes.js Copy and paste the code below into routes.js;

        const Book = require('./models/book');
    const path = require('path');
    
    module.exports = function(app) {
      app.get('/book', async (req, res) => {
        try {
          const books = await Book.find();
          res.json(books);
        } catch (err) {
          res.status(500).json({ message: 'Error fetching books', error: err.message });
        }
      });
    
      app.post('/book', async (req, res) => {
        try {
          const book = new Book({
            name: req.body.name,
            isbn: req.body.isbn,
            author: req.body.author,
            pages: req.body.pages
          });
          const savedBook = await book.save();
          res.status(201).json({
            message: 'Successfully added book',
            book: savedBook
          });
        } catch (err) {
          res.status(400).json({ message: 'Error adding book', error: err.message });
        }
      });
    
      app.delete('/book/:isbn', async (req, res) => {
        try {
          const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
          if (!result) {
            return res.status(404).json({ message: 'Book not found' });
          }
          res.json({
            message: 'Successfully deleted the book',
            book: result
          });
        } catch (err) {
          res.status(500).json({ message: 'Error deleting book', error: err.message });
        }
      });
    
      app.get('*', (req, res) => {
        res.sendFile(path.join(__dirname, '../public', 'index.html'));
      });
    };
In the 'apps' folder, create a folder named models and in the models folder, create a file named book.js
Copy and paste the code below into 'book.js'

        const mongoose = require('mongoose');
        
        const bookSchema = new mongoose.Schema({
          name: { type: String, required: true },
          isbn: { type: String, required: true, unique: true, index: true },
          author: { type: String, required: true },
          pages: { type: Number, required: true, min: 1 }
        }, {
          timestamps: true
        });
        
        module.exports = mongoose.model('Book', bookSchema);

