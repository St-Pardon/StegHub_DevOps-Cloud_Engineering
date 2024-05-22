# Introduction

The MEAN stack is a well-regarded JavaScript stack used for developing web applications. MEAN stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js. Below is an overview of each component:

**MongoDB:** MongoDB is a NoSQL database that stores data in a flexible, JSON-like format. It is widely used in web applications due to its scalability, flexibility, and performance.

**Express.js:** Express.js is a web application framework for Node.js. It offers features for building web applications and APIs, including routing, middleware support, and templating engines. Express.js simplifies the development of web applications by providing a higher-level abstraction over the HTTP protocol.

**AngularJS (or Angular):** AngularJS is a JavaScript framework developed and maintained by Google for creating dynamic web applications. It extends HTML with additional attributes and provides built-in directives for building interactive user interfaces. AngularJS follows the Model-View-Controller (MVC) architecture, making it easier to organize code and build complex web applications.

**Node.js:** Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. It enables developers to run JavaScript code outside of a web browser, allowing for the creation of server-side applications using JavaScript. Node.js is known for its event-driven, non-blocking I/O model, which makes it suitable for building scalable and high-performance web applications.

## Step 0: Prerequisites

1. **Launch EC2 Instance:**
   - An EC2 instance of type `t3.small` running Ubuntu 22.04 LTS (HVM) was launched in the `us-east-1` region using the AWS console.
   
   ![Launch Instance](../LEMP-Setup/assets/Screenshot%202024-05-12%20at%202.31.49%20PM.png)

2. **Attach SSH Key:**
   - An SSH key named `my-ec2-key` was attached to access the instance on port 22.

3. **Configure Security Group:**
   - The security group was set up with the following inbound rules:
     - Allow traffic on port 80 (HTTP) from anywhere.
     - Allow traffic on port 443 (HTTPS) from anywhere.
     - Allow traffic on port 22 (SSH) from any IP address (default).
     - Allow traffic on port 3300 (Custom TCP) from anywhere.

4. **Connect to the Instance:**
   - Change the private key file permissions and connect to the instance using SSH.
   ```bash
   chmod 400 my-ec2-key.pem
   ssh -i "my-ec2-key.pem" ubuntu@54.81.119.2
   ```
   - Here, `username=ubuntu` and `public IP address=54.81.119.2`.
   
   ![Connect to Instance](./assets/Screenshot%202024-05-21%20at%206.23.39%20PM.png)

## Step 1: Install Node.js

Node.js is a JavaScript runtime used in this tutorial to set up Express routes and AngularJS controllers.

1. **Update and Upgrade Ubuntu:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
   ![Update Ubuntu](./assets/Screenshot%202024-05-21%20at%206.24.36%20PM.png)

2. **Add Certificates:**
   ```bash
   sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
   curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   ```
   ![Add Certificates](./assets/Screenshot%202024-05-21%20at%206.29.25%20PM.png)

3. **Install Node.js:**
   ```bash
   sudo apt-get install -y nodejs
   ```
   ![Install Node.js](./assets/Screenshot%202024-05-21%20at%206.29.37%20PM.png)

## Step 2: Install MongoDB

Book records, containing book name, ISBN number, author, and number of pages, will be stored in MongoDB.

1. **Download the MongoDB Public GPG Key:**
   ```bash
   curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
   ```

2. **Add the MongoDB Repository:**
   ```bash
   echo "deb [ signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
   ```
   ![MongoDB GPG Key and Repo](./assets/Screenshot%202024-05-21%20at%206.31.46%20PM.png)

3. **Update Package Database and Install MongoDB:**
   ```bash
   sudo apt-get update
   sudo apt-get install -y mongodb-org
   ```
   ![Install MongoDB](./assets/Screenshot%202024-05-21%20at%206.53.15%20PM.png)

4. **Start and Enable MongoDB:**
   ```bash
   sudo systemctl start mongod
   sudo systemctl enable mongod
   sudo systemctl status mongod
   ```
   ![Start MongoDB](./assets/Screenshot%202024-05-21%20at%206.52.57%20PM.png)

5. **Install Body-Parser Package:**
   ```bash
   sudo npm install body-parser
   ```
   
6. **Create the Project Root Folder:**
   ```bash
   mkdir Books && cd Books
   npm init
   ```
   ![Initialize Project Folder](./assets/Screenshot%202024-05-21%20at%206.57.07%20PM.png)

7. **Create `server.js` in the `Books` Folder:**
   ```bash
   vim server.js
   ```
   Copy and paste the following code into `server.js`:
   ```javascript
   const express = require('express');
   const bodyParser = require('body-parser');
   const mongoose = require('mongoose'); // Ensure mongoose is installed and required
   const path = require('path'); // To handle static file serving
   const app = express();

   // Connect to MongoDB
   mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true, useUnifiedTopology: true })
     .then(() => console.log('MongoDB connected'))
     .catch(err => console.error('MongoDB connection error:', err));

   // Middleware
   app.use(bodyParser.json());
   app.use(express.static(path.join(__dirname, 'public')));

   // Routes
   require('./apps/routes')(app);

   // Start the server
   app.set('port', 3300);
   app.listen(app.get('port'), () => {
     console.log('Server up: http://localhost:' + app.get('port'));
   });
   ```
   ![Server.js](./assets/Screenshot%202024-05-21%20at%207.01.47%20PM.png)
   ![Server.js](./assets/Screenshot%202024-05-21%20at%207.02.10%20PM.png)

## Step 3: Install Express and Set Up Routes

Express is used to pass book information to and from our MongoDB database. Mongoose provides a schema-based solution to model the application data.

1. **Install Express and Mongoose:**
   ```bash
   sudo npm install express mongoose
   ```
   ![Install Express and Mongoose](./assets/Screenshot%202024-05-21%20at%206.58.10%20PM.png)

2. **Create `apps` Folder and Add Routes:**
   ```bash
   mkdir apps && cd apps
   vim routes.js
   ```
   Copy and paste the following code into `routes.js`:
```javascript
   const Book = require('./models/book');
   const path = require('path');

   module.exports = function(app) {
     // Get all books
     app.get('/book', async (req, res) => {
       try {
         const books = await Book.find({});
         res.json(books);
       } catch (err) {
         console.error(err);
         res.status(500).json({ error: 'Internal Server Error' });
       }
     });

     // Add a new book
     app.post('/book', async (req, res) => {
       try {
         const book = new Book({
           name: req.body.name,
           isbn: req.body.isbn,
           author: req.body.author,
           pages: req.body.pages
         });
         const result = await book.save();
         res.json({
           message: "Successfully added book",
           book: result
         });
       } catch (err) {
         console.error(err);
         res.status(500).json({ error: 'Internal Server Error' });
       }
     });

     // Update a book
     app.put('/book/:isbn', async (req, res) => {
       try {
         const updatedBook = await Book.findOneAndUpdate(
           { isbn: req.params.isbn },
           req.body,
           { new: true }
         );
         if (!updatedBook) {
           return res.status(404).json({ error: 'Book not found' });
         }
         res.json({
           message: "Successfully updated the book",
           book: updatedBook
         });
       } catch (err) {
         console.error(err);
         res.status(500).json({ error: 'Internal Server Error' });
       }
     });

     // Delete a book
     app.delete('/book/:isbn', async (req, res) => {
       try {
         const result = await Book.findOneAndRemove({ isbn: req.params.isbn });
         if (!result) {
           return res.status(404).json({ error: 'Book not found' });
         }
         res.json({
           message: "Successfully deleted the book",
           book: result
         });
       } catch (err) {
         console.error(err);
         res.status(500).json({ error: 'Internal Server Error' });
       }
     });

     // Serve static files
     app.get('*', (req, res) => {
       res.sendFile(path.join(__dirname, '../public', 'index.html'));
   });
   };
```
   ![Routes.js](./assets/Screenshot%202024-05-21%20at%206.59.08%20PM.png)

3. **Create `models` Folder and Add Book Model:**
   ```bash
   mkdir models && cd models
   vim book.js
   ```
   Copy and paste the following code into `book.js`:
   ```javascript
   const mongoose = require('mongoose');
   const Schema = mongoose.Schema;

   const bookSchema = new Schema({
     name: { type: String, required: true },
     isbn: { type: String, required: true, unique: true },
     author: { type: String, required: true },
     pages: { type: Number, required: true }
   });

   module.exports = mongoose.model('Book', bookSchema);
   ```

## Step 4: Set up Public Directory
Navigate to the root directory and make a public directory and navigate 
```sh
cd ../..
mkdir public && cd public
```
create a file server.js
```sh
vi script.js
```
Copy the following code int the file
```js
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
![](./assets/Screenshot%202024-05-21%20at%206.57.43%20PM.png)
Create a file `index.html` and paste the followimng code
```html
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

```


## Step 5: Deploy and Test the Application

1. **Run the Node.js Server:**
   ```bash
   node server.js
   ```
   ![Run Server](./assets/Screenshot%202024-05-21%20at%207.04.14%20PM.png)

2. **Access the Application:**
   Open a web browser and navigate to `http://your-IP:3300` to see the Angular front-end interacting with the Express API and MongoDB.

   ![Web Application](./assets/Screenshot%202024-05-21%20at%207.05.54%20PM.png)

   Addiong a book
   ![Web Application](./assets/Screenshot%202024-05-21%20at%207.07.08%20PM.png)

   Viewing the book route
   ![Web Application](./assets/Screenshot%202024-05-21%20at%207.09.11%20PM.png)

## Conclusion

The MEAN stack provides a comprehensive solution for developing full-stack web applications. This tutorial covered the installation and configuration of MongoDB, Node.js, Express, and AngularJS, as well as setting up an EC2 instance to host the application. By following these steps, you can create and deploy a functional web application that uses MongoDB for data storage, Express for the back-end, and AngularJS for the front-end.
