In this project, we are tasked to implement a web solution based on MERN stack in AWS Cloud.

MERN Web stack consists of following components:

1. MongoDB: A document-based, No-SQL database used to store application data in a form of documents.
2. ExpressJS: A server side Web Application framework for Node.js.
3. ReactJS: A frontend framework developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components.
4. Node.js: A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.
   
## STEP ONE: Backend configuration
First we need to update and upgrade ubuntu by running the commands:

      sudo apt update && sudo apt upgrade
Then we need to get the location of Node.js software from Ubuntu repositories with the command:

      curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

### We then install Node.js with the command below:

      sudo apt-get install -y nodejs      

![install-node](https://github.com/user-attachments/assets/d3786175-51a8-4f26-962e-ba5d5905b832)
Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu,
it is used to install Node modules & packages and to manage dependency conflicts.

Verify the node installation with the command below:

            node -v 
            npm -v 

![node-npmVersion](https://github.com/user-attachments/assets/cc63a88a-efb3-4420-93ed-0a3472db5712)

Now, we create our application code setup by first creating a new directory for our "To-Do" project by the command:

            mkdir Todo 
and verify by the command

            ls
 Now change your current directory to the newly created one with the command:

             cd Todo
  ![createnpmdir](https://github.com/user-attachments/assets/7268d7f4-6df8-4ed0-8c30-ece52821f5b9)
           
  Next, you will use the command npm init to initialise your project, so that a new file named package.json will be created.
  This file will normally contain information about your application and the dependencies that it needs to run. 
  Follow the prompts after running the command. You can press Enter several times to accept default values, 
  then accept to write out the package.json file by typing yes.   
Run the command ls to confirm that you have package.json file created.

Next, we will Install ExpressJs and create the Routes directory.

![npm-config](https://github.com/user-attachments/assets/2b477983-66fa-4838-afa0-380707249794)

### We install Express.js
We must remember that Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. 
Therefore it simplifies development, and abstracts a lot of low level details. For example, Express helps to define routes of our application based on 
HTTP methods and URLs.

To use express, we install it using npm:

          npm install express
Now we create a file index.js with the command below
![installExpress](https://github.com/user-attachments/assets/d0f2f6a3-1f29-4715-ad19-546b01218a9b)

Now we create a file index.js with the command _touch index.js _ and install the dotenv module with the command:

        npm install dotenv

![installdotenv](https://github.com/user-attachments/assets/6df08c99-b34d-4798-bbde-4966bf3005b0)

Open the index.js file with the command below

        vim index.js
Type the code below into it and save. Do not get overwhelmed by the code you see. For now, simply paste the code into the file.

        
        const express = require('express');
        require('dotenv').config();
        
        const app = express();
        
        const port = process.env.PORT || 5000;
        app.use((req, res, next) => {
        res.header("Access-Control-Allow-Origin", "\*");
        res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        next();
        });
        
        app.use((req, res, next) => {
        res.send('Welcome to Express');
        });
        
        app.listen(port, () => {
        console.log(`Server running on port ${port}`)
        });
        
Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.


Now it is time to start our server to see if it works. We open our terminal in the same directory as your index.js file and type:   

        node index.js
If every thing goes well, you should see Server running on port 5000 in your terminal.

![5000port](https://github.com/user-attachments/assets/176e0f82-cfad-4c46-ad19-ab4c406670dd)

Now we need to open this port in EC2 Security Groups.We need to open port 5000 in the inbound rules , like this:

![Security5000](https://github.com/user-attachments/assets/3450f453-c247-4f59-85eb-530c6fc359e4)

On our browser we can try to access your server's Public IP or Public DNS name followed by port 5000:

![ExpressPage](https://github.com/user-attachments/assets/f8fe2532-aebe-4742-a9a0-0b283db352f9)

There are three actions that our To-Do application needs to be able to do:

1. Create a new task
2. Display list of all tasks
3. Delete a completed task

Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.
For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes
Change directory to routes folder, create a file api.js and copy below code in the file. 

          const express = require ('express');
          const router = express.Router();

          router.get('/todos', (req, res, next) => {

          });

          router.post('/todos', (req, res, next) => {

            });

          router.delete('/todos/:id', (req, res, next) => {

          })

          module.exports = router;

now, let create Models directory.

### Models
Now comes the interesting part, since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model.
A model is at the heart of JavaScript based applications, and it is what makes it interactive.
We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. (Seems like a lot of information, but not to worry, everything will become clear to you over time. I promise!!!)
In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties
To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.
Change directory back Todo folder with cd .. and install Mongoose

          npm install mongoose

Create a new folder with _mkdir models_ command
Change directory into the newly created 'models' folder with cd models.
Inside the models folder, create a file and name it _todo.js_

          touch todo.js
          mkdir models && cd models && touch todo.js

Open the file created with _vim todo.js_ then paste the code below in the file:


          const mongoose = require('mongoose');
          const Schema = mongoose.Schema;

          //create schema for todo
          const TodoSchema = new Schema({
          action: {
          type: String,
          required: [true, 'The todo text field is required']
          }
            })

          //create model for todo
          const Todo = mongoose.model('todo', TodoSchema);

          module.exports = Todo;


Now we need to update our routes from the file api.js in 'routes' directory to make use of the new model.

In Routes directory, open api.js with _vim api.js_, delete the code inside with _:%d_ command and paste there code below into it then save and exit

          const express = require ('express');
          const router = express.Router();
          const Todo = require('../models/todo');

          router.get('/todos', (req, res, next) => {

        //this will return all the data, exposing only the id and action field to the client
        Todo.find({}, 'action')
        .then(data => res.json(data))
        .catch(next)
          });

        router.post('/todos', (req, res, next) => {
        if(req.body.action){
        Todo.create(req.body)
        .then(data => res.json(data))
        .catch(next)
        }else {
        res.json({
        error: "The input field is empty"
        })
          }
        });

        router.delete('/todos/:id', (req, res, next) => {
        Todo.findOneAndDelete({"_id": req.params.id})
        .then(data => res.json(data))
          .catch(next)
          })

            module.exports = router;
The next piece of our application will be the MongoDB Database
### MongoDB Database
We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS), 
so to make life easy, you will need to sign up for a shared clusters free account, which is ideal for our use case. Sign up here.
Follow the sign up process, select AWS as the cloud provider, and choose a region near you.
Complete a get started checklist as shown on the image below



