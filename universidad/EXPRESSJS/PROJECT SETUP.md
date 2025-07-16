|CREAR UN PACKAGE BASICO
npm init -y
![[Pasted image 20250710231733.png]]Luego para añadir la dependencia de express
-- npm i express

In order to make our development easier: 
Nota: esto será una dependencia de dev
npm i --save-dev nodemon
to run it
npm run devStart : 
![[Pasted image 20250710232749.png]]

This basically work for restarting any changes me make in our server, instead of doing manually, it will make it automatically.

This is how it originally is:

![[Pasted image 20250710232502.png]]
{

  "name": "express",

  "version": "1.0.0",

  "main": "index.js",

  "scripts": {

    "devStart": "nodemon server.js" //here the line we've got to change server js will be our folder's name.

  },

  "keywords": [],

  "author": "",

  "license": "ISC",

  "description": "",

  "dependencies": {

    "express": "^5.1.0"

  },

  "devDependencies": {

    "nodemon": "^3.1.10"

  }

}

# CREATING THE ACTUAL EXPRESS JS SERVER

This all is made in the server.js folder:

const express = require ('express') //to call our dependency

const app = express()//here by calling express as a function, we create an application, which allows us to set up our entire server.

  

app.listen(3000)//here we set up what port the app will be listening on.

ROUTERS: 
app.get
app.delete
app.update
app.put

ALL THE HTTP METHODS


TO RENDER
We have to create a views folder, inside of it we create as many views as we want to.
![[Pasted image 20250711173512.png]]

 res.render('index')
and in the terminal in order to make that render to work, we will use the next command: 
npm i ejs

then:
app.set('view engine', 'ejs')
and we will change the html tag
![[Pasted image 20250711174038.png]]
and we will be able to see our view 
![[Pasted image 20250711174233.png]]


To render something coming straight from the server: 
in the index: 
<%= text %>
in the server: 
res.render("index", {text: "World"})
![[Pasted image 20250711175526.png]]

 res.render("index", {text123: "World"})
 here is about if "text" does not exist then it will print defualt 
 <%= locals.text  || 'Default' %>