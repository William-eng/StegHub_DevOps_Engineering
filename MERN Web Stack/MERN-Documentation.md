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
![Screenshot from 2024-09-20 21-28-05](https://github.com/user-attachments/assets/9ae68bd4-d763-4b4d-b359-8800f4aa5ba8)
Allow access to the MongoDB database from anywhere (Not secure, but it is ideal for testing)

IMPORTANT NOTE In the image below, make sure you change the time of deleting the entry from 6 Hours to 1 Week

![allowip](https://github.com/user-attachments/assets/4df5e5ca-c43f-4ee5-97d9-1818ed08799f)

We need to now create a MongoDB database and collection 

![collection](https://github.com/user-attachments/assets/572d7534-4f45-4cb4-b994-f8570d69ede6)

In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now. We create a file in your Todo directory and name it .env.
We then need to add the connection string to access the database in it, just as below:

            DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'

We must ensure to update <username>, <password>, <network-address> and <database> according to your setup

Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database.

Simply delete existing content in the file, and update it with the entire code below.

To do that using vim, follow below steps

- Open the file with vim index.js
- Press esc
- Type :
- Type %d
- Hit 'Enter'
- The entire content will be deleted, then,

Press i to enter the insert mode in vim
Now, paste the entire code below in the file.

      const express = require('express');
      const bodyParser = require('body-parser');
      const mongoose = require('mongoose');
      const routes = require('./routes/api');
      const path = require('path');
      require('dotenv').config();
      
      const app = express();
      
      const port = process.env.PORT || 5000;
      
      //connect to the database
      mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
      .then(() => console.log(`Database connected successfully`))
      .catch(err => console.log(err));
      
      //since mongoose promise is depreciated, we overide it with node's promise
      mongoose.Promise = global.Promise;
      
      app.use((req, res, next) => {
      res.header("Access-Control-Allow-Origin", "\*");
      res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
      next();
      });
      
      app.use(bodyParser.json());
      
      app.use('/api', routes);
      
      app.use((err, req, res, next) => {
      console.log(err);
      next();
      });
      
      app.listen(port, () => {
      console.log(`Server running on port ${port}`)
      });

Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

Start your server using the command:

      node index.js
      
We shall see a message 'Database connected successfully', if so - we have our backend configured. Now we are going to test it.

![Database-Connected-Sucessfully](https://github.com/user-attachments/assets/050d7516-9721-4aed-ae9c-46bc2b5bec7b)

## STEP FIVE : TESTING BACKEND CODE WITHOUT FRONTEND USING RESTful API
So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code.
We should test all the API endpoints and make sure they are working. For the endpoints that require body, you should send JSON back with the necessary fields since it’s what we setup in our code.
Now open your Postman, create a POST request to the API http://<PublicIP-or-PublicDNS>:5000/api/todos. This request sends a new task to our To-Do list so the application could store it in the database.


Note: make sure your set header key Content-Type as application/json

![POSTMANrequest](https://github.com/user-attachments/assets/370b2acb-7f27-4585-a7bc-7cd23f3b3292)

## STEP SIX: FRONTEND CREATION

Since we are done with the functionality we want from our backend and API, it is time to create a user interface for a Web client (browser) to interact with the application via API. To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app.

In the same root directory as your backend code, which is the Todo directory, run:

         npx create-react-app client

![npcCreate](https://github.com/user-attachments/assets/eefaed41-6695-46a9-8632-093c9b9283ec)


This will create a new folder in our Todo directory called _client_, where we will add all the react code.

Running a React App
Before testing the react app, there are some dependencies that need to be installed.

1. Install concurrently. It is used to run more than one command simultaneously from the same terminal window.

             npm install concurrently --save-dev
         
2. Install nodemon. It is used to run and monitor the server. If there is any change in the server code,
  nodemon will restart it automatically and load the new changes.

            npm install nodemon --save-dev

![dependenc](https://github.com/user-attachments/assets/9aa9d00e-968d-49c6-bead-e738ea41fcd0)

3. In Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below.

               "scripts": {
               "start": "node index.js",
               "start-watch": "nodemon index.js",
               "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
               },

  ### Configure Proxy in package.json
Change directory to '_client'_

               cd client
Open the package.json file

               vi package.json
Add the key value pair in the package.json file "proxy": "http://localhost:5000".
The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like _http://localhost:5000_ rather than always including the entire path like_ http://localhost:5000/api/todos_

Now, ensure you are inside the Todo directory, and simply do:

               npm run dev

 ![npmrundev](https://github.com/user-attachments/assets/a4669417-16aa-45e1-acaa-81469ade808a)
              
Your app should open and start running on localhost:3000

### Creating your React Components
One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component. From your Todo directory run

               cd client
move to the src directory

               cd src
Inside your src folder create another folder called components

               mkdir components
Move into the components directory with

               cd components
Inside 'components' directory create three files Input.js, ListTodo.js and Todo.js.

               touch Input.js ListTodo.js Todo.js
Open Input.js file

               vi Input.js
Copy and paste the following


         import React, { Component } from 'react';
         import axios from 'axios';
         
         class Input extends Component {
         
         state = {
         action: ""
         }
         
         addTodo = () => {
         const task = {action: this.state.action}
         
             if(task.action && task.action.length > 0){
               axios.post('/api/todos', task)
                 .then(res => {
                   if(res.data){
                     this.props.getTodos();
                     this.setState({action: ""})
                   }
                 })
                 .catch(err => console.log(err))
             }else {
               console.log('input field required')
             }
         
         }
         
         handleChange = (e) => {
         this.setState({
         action: e.target.value
         })
         }
         
         render() {
         let { action } = this.state;
         return (
         <div>
         <input type="text" onChange={this.handleChange} value={action} />
         <button onClick={this.addTodo}>add todo</button>
         </div>
         )
         }
         }
         
         export default Input

         
To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.

Move to the src folder, Move to clients folder and Install Axios

      npm install axios

Go to _'components'_ directory

      cd src/components
After that we open the ListTodo.js

      vi ListTodo.js

in the ListTodo.js we add the following code

      import React from 'react';

      const ListTodo = ({ todos, deleteTodo }) => {
      
      return (
      <ul>
      {
      todos &&
      todos.length > 0 ?
      (
      todos.map(todo => {
      return (
      <li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
      )
      })
      )
      :
      (
      <li>No todo(s) left</li>
      )
      }
      </ul>
      )
      }
      
      export default ListTodo

Then in our Todo.js file you write the following code

      import React, {Component} from 'react';
      import axios from 'axios';
      
      import Input from './Input';
      import ListTodo from './ListTodo';
      
      class Todo extends Component {
      
      state = {
      todos: []
      }
      
      componentDidMount(){
      this.getTodos();
      }
      
      getTodos = () => {
      axios.get('/api/todos')
      .then(res => {
      if(res.data){
      this.setState({
      todos: res.data
      })
      }
      })
      .catch(err => console.log(err))
      }
      
      deleteTodo = (id) => {
      
          axios.delete(`/api/todos/${id}`)
            .then(res => {
              if(res.data){
                this.getTodos()
              }
            })
            .catch(err => console.log(err))
      
      }
      
      render() {
      let { todos } = this.state;
      
          return(
            <div>
              <h1>My Todo(s)</h1>
              <Input getTodos={this.getTodos}/>
              <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
            </div>
          )
      
      }
      }
      
      export default Todo;

We need to make little adjustment to our react code. Delete the logo and adjust our App.js to look like this.
Move to the src folder in the src folder we open the App.js and add the following code:

         import Todo from './components/Todo';
         import './App.css';
         
         const App = () => {
         return (
         <div className="App">
         <Todo />
         </div>
         );
         }
         
         export default App;
After pasting, exit the editor, in the src directory open the _App.css_, and then paste the following code into App.css:

         .App {
         text-align: center;
         font-size: calc(10px + 2vmin);
         width: 60%;
         margin-left: auto;
         margin-right: auto;
         }
         
         input {
         height: 40px;
         width: 50%;
         border: none;
         border-bottom: 2px #101113 solid;
         background: none;
         font-size: 1.5rem;
         color: #787a80;
         }
         
         input:focus {
         outline: none;
         }
         
         button {
         width: 25%;
         height: 45px;
         border: none;
         margin-left: 10px;
         font-size: 25px;
         background: #101113;
         border-radius: 5px;
         color: #787a80;
         cursor: pointer;
         }
         
         button:focus {
         outline: none;
         }
         
         ul {
         list-style: none;
         text-align: left;
         padding: 15px;
         background: #171a1f;
         border-radius: 5px;
         }
         
         li {
         padding: 15px;
         font-size: 1.5rem;
         margin-bottom: 15px;
         background: #282c34;
         border-radius: 5px;
         overflow-wrap: break-word;
         cursor: pointer;
         }
         
         @media only screen and (min-width: 300px) {
         .App {
         width: 80%;
         }
         
         input {
         width: 100%
         }
         
         button {
         width: 100%;
         margin-top: 15px;
         margin-left: 0;
         }
         }
         
         @media only screen and (min-width: 640px) {
         .App {
         width: 60%;
         }
         
         input {
         width: 50%;
         }
         
         button {
         width: 30%;
         margin-left: 10px;
         margin-top: 0;
         }
         }

Then in the src directory we open the index.css, copy and paste the code below:


      body {
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
      "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
      sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      box-sizing: border-box;
      background-color: #282c34;
      color: #787a80;
      }
      
      code {
      font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
      monospace;
      }

We return to the Todo directory and run the command;

      npm run dev
![Rundevlast](https://github.com/user-attachments/assets/eee7b452-f393-46a3-85f1-7169733aaeb8)

      
on Port 3000, we can see our Todo app:
![TodoApp](https://github.com/user-attachments/assets/6faf828e-a4db-4fe7-a1a4-eebd4e77ed43)


      
