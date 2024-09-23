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



