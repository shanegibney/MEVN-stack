MEVN - notes

These notes are adopted from an original tutorial https://www.youtube.com/watch?v=9Kju3DovLrg but to use vue-cli3.x.

node 10.15.0
npm 6.4.1
vue 3.3.0
mysql 8.0.13
phpmyadmin 8.0.13

MySQL-Express-Vue-cli3.x.-Node

Using phpmyadmin or from command line create a new MySQL database called 'nodejs-tasks'.

Use user 'root' and add a password.

Create a table called tbl_tasks with just two columns.
 Id (int) auto-increment

 task_name (text)

Create a home directory for this project,

$ mkdir MEVN-MySQL-Express-Vue-Node

and cd into it,

$ cd MEVN-MySQL-Express-Vue-Node

In the project's home directory MEVN-MySQL-Express-Vue-Node run

$ npm init

This will create a package.json files

Add dependencies,

$ npm install body-parser cors express mysql2 nodemon sequelize --save

The package.json should now look like this,

{
"name": "mevn-mysql-express-vue-node",
"version": "1.0.0",
"description": "mysql express vue-cli node CRUD app",
"main": "index.js",
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
},
  "author": "me",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.18.3",
    "cors": "^2.8.5",
    "express": "^4.16.4",
    "mysql2": "^1.6.4",
    "nodemon": "^1.18.9",
    "sequelize": "^4.42.0"
  }
}

Install these dependencies,

$ npm i

This should create an addition directory node_modules in your project's home directory.

Create a new file server.js

$ touch server.js

This file needs to be,

var express = require("express")
var bodyParser = require("body-parser")

// var tasks = require("./routes/tasks")
var cors = require("cors")

var port = 3000

var app = express()
app.use(cors())

app.use(bodyParser.json())
app.use(bodyParser.urlencoded({extended: false}))

// app.use("/api", tasks)

app.listen(port, function(){
  console.log('Server started on port ' +  port)
})

In the project's home directory create a new directory, change directory into it (cd) and then create a new file db.js

$ mkdir database
$ cd database
$ touch db.js

Into db.js put

const Sequelize = require("sequelize")
const db = {}
const sequelize = new Sequelize("nodejs_tasks", "root", "my-passowrd-goes-here",{
  host: "localhost",
  dialect: "mysql",
  operatorsAliases: false,

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
})

db.sequelize = sequelize

module.exports = db

Remember to put in the database password.

Sequelize is a promise-based ORM for Node.js v4 and up

In the project's home directory create a directory called model and place inside it a file called Task.js

$ mkdir model
$ cd model
$ touch Task.js

const Sequelize = require("sequelize")
const db = require("../database/db.js")

module.expports = db.sequelize.define(
  "tbl_tasks",
  {
    id: {
      type: Sequelize.INTEGER,
      primaryKey: true,
      autoIncrement: true
    },
    task_name: {
      type: Sequelize.STRING
    }
  },
  {
    timestamps: false
   }
)

Again in the project's home directory create a directory called routes and in it place a file called tasks.js

$ mkdir routes
$ cd routes
$ touch tasks.js

var express = require("express")
var router = express.Router()
const Task = require("../model/Task")

router.get("/tasks", (req, res) => {
  Task.findAll()
  .then(tasks => {
    res.json(tasks)
  })
  .catch(err => {
    res.send("error: " + err)
  })
})

Next create a script called start to run server.js which will use nodemon. Nodemon will automatically restart the server if any changes are made. This is very useful in production and saves you having to stop and start the server every time you change something.

In package.json add to scripts "start" like this,

"scripts": {
  "start": "nodemon server.js",
  "test": "echo \"Error: no test specified\" && exit 1"
},

Then to start server running,

$ npm start

To the end of tasks.js add,

//Add task
router.post("/task", (req, res) => {
  if(!req.body.task_name){
    res.status(400)
    res.json({
      error: "Bad Data"
    })
  }else{
    Task.create(req.body)
    .then(() => {
      res.send("Task Added")
    })
    .catch(err => {
      res.send("Error: " + err)
    })
  }
})

module.exports = router

Must use x-www-form-urlencoded radio button option under 'Body' in Postman.

To Postman add a POST request to http://localhost:3000/api/task and add a key of task_name and value i.e. 'blah blah'.

In Postman test for GET with http://localhost:3000/api/tasks

To tasks.js add the ability to delete by id,

router.delete("/task/:id", (req, res) => {
 Task.destroy({
   where: {
     id: req.params.id
   }
 })
 .then(() => {
   res.send("Task Deleted!")
 })
 .catch(err => {
   res.send("error: " + err)
 })
 })

Test it in Postman using DELETE and the url http://localhost:3000/api/task/1 to delete the entry with id 1, and use JSON in the Postman dropdown menu.

Add the code for updating the data,

router.put("/task/:id", (req, res) => {
  if(!req.body.task_name) {
    res.status(400)
    res.json({
      error: "Bad Data"
    })
  }else{
    Task.update(
      { task_name: req.body.task_name },
      { where: { id: req.params.id }}
    )
      .then(() => {
         res.send("Task Updated")
      })
      .error(err => res.send(err))
  }
})

In Postman test with PUT and http://localhost:3000/api/task/2 to update the entry with id 2, and in the key add task_name and then a string in value.

Next add the frontend framework Vue-cli3. This will create a new directory called 'client'

To start a project in the old vue-cli you used 'vue init' but the new vue/cli use instead 'vue create project_name '. This will enable you to either use a preset or custom config. The new cli is heavily plugin driven. Plugins such as ESLint, PWA etc. This can then be used as a new preset for future projects. After the project has been set up you add plugins with,

$ vue add @vue/plugin-name

The old cli was installed with

$ sudo npm install -g vue-cli   (Don't use!!!)

The new vue/cli is installed globally with

$ sudo npm install -g @vue/cli

This will replace the vue binary which you used to run vue init.

To check the version,

$ vue --version
3.3.0

To create a new project, (In old cli to create projects you used $ vue init project_name)

$ vue create project_name

babel allows you to use next generation Javascript features. Best to select ESLint default and dedicated config files. But router does need to be installed and best to do it when project is being created.

Note: in your computer's home directory a hidden file .vuerc should have been created. This is the global configuration for the new vue/cli but nothing needs to be done with it here.

Navigate into the newly create directory project_name

$ cd project_name

To run the development server use the run 'serve', where 'serve' is an automatically generated script which can be found in the project's home directory in package.json. This script runs the development server.

$ npm run server

If git is not installed you will get errors which can be ignored. git can be easily installed globally with,

$ npm install -g git

You may need to place sudo before every npm command. To avoid this change the permissions on ~/.npm with,

$ sudo chown -R $(whoami) ~/.npm

Install axios and bootstrap

$ sudo npm install --save bootstrap axios

This will add a bootstrap and an axios directory to node_modules in your project.

It may be necessary to also,

$ sudo npm install

You need to add vue.config.js in the project's root directory next to package.json.
(For more on this see https://cli.vuejs.org/config/#devserver-proxy)

So vue.config.js becomes,

// vue.config.js
module.exports = {
  devServer: {
    proxy: {
      '^/api': {
        target: 'http://localhost:3000',
        changeOrigin: true
      }
    }
  }
}

Just after the other imports add to client/src/main.js

require('../node_modules/bootstrap/dist/css/bootstrap.css')

The major difference between require and import, is that require will automatically scan node_modules to find modules, but import, which comes from ES6, won't.

Remove the logo from src/views/Home.vue line,

<img alt="Vue logo" src="../assets/logo.png">

In src/router.js add the route for the new List component,

import Vue from 'vue'
import Router from 'vue-router'
import List from '@/components/List'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'List',
      component: List
    }
  ]
})

Create a new component by adding this file src/components/List.vue

<template>
  <div class="hello">
     <div id="todo-list-example" class="container">
        <div class="row">
           <div class="col-md-6 mx-auto">
              <h1 class="text-center">TODO List App</h1>
              <form v-on:submit.prevent="addNewTask">
                 <label for="tasknameinput">Task Name</label>
                 <input v-model="taskname" type="text" id="tasknameinput" class="form-control" placeholder="Add New Task">
                 <button v-if="this.isEdit == false" type="submit" class="btn btn-success btn-block mt-3">
                    Submit
                 </button>
                 <button v-else v-on:click="updateTask()" type="button" class="btn btn-primary btn-block mt-3">
                    Update
                 </button>
              </form>

              <table class="table">
                 <tr v-for="(todo) in todos" v-bind:key="todo.id" v-bind:title="todo.task_name">
                    <td class="text-left">{{todo.task_name}}</td>
                    <td class="text-right">
                       <button class="btn btn-info" v-on:click="editTask(todo.task_name, todo.id)">Edit</button>
                       <button class="btn btn-danger" v-on:click="deleteTask(todo.id)">Delete</button>
                    </td>
                 </tr>
              </table>
           </div>
        </div>
     </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
name: 'List',
  data() {
     return {
        todos: [],
        id: '',
        taskname: '',
        isEdit: false
     }
  },
  mounted () {
     this.getTasks()
  },
  methods: {
     getTasks(){
        axios.get("/api/tasks").then(
           result => {
              console.log(result.data)
              this.todos = result.data
           },
           error => {
              console.error(error)
           }
        )
     },
     addNewTask() {
         axios.post("/api/task", {task_name: this.taskname})
         .then((res) => {
             this.taskname = ''
             this.getTasks()
         }).catch((err) => {
             console.log(error)
         })
     },
     editTask(title, id){
        this.id = id
        this.taskname = title
        this.isEdit = true
     },
     updateTask() {
        axios.put(`/api/task/${this.id}`, {task_name: this.taskname})
        .then((res) => {
           this.taskname = ''
           this.isEdit = false
           this.getTasks()
           console.log(res)
        })
        .catch((err) => {
          console.log(err)
        })
      },
      deleteTask(id){
         axios.delete(`/api/task/${id}`)
            .then((res) => {
            this.taskname = ''
            this.getTasks()
            console.log(res)
            })
            .catch((err) => {
                console.log(err)
            })
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

In App.vue you will need,

<script>
import List from './components/List'
export default {
  name: 'App',
  components: {
    List
  }
}
</script>

Navigating to localhost:8080 should allow you to see the dashboard.
Of course the mysql server and Express server needs to also be running.
