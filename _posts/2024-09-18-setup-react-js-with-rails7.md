---
layout: post
title: Setup A Ruby on Rails 7 API With React JS
subtitle: Quickly setup a Ruby on Rails 7 API only app with React JS
tags: [ruby on rails, react js, setup, tutorial]
comments: true
thumbnail-img: /assets/img/books.gif
---

This blog shows how to setup a React JS app with a Rails 7 API and how it communicates with the api to display the Books data on the UI.

#### Step 1. First lets create a rails api only application
``` rails new bookstore --api ```

This way it doesnt create views etc.

#### Step 2. Update Gemfile

Check rails version to be ~7.0.0 and Ruby ~3.0.0. Now un-comment or add these lines to whitelist frontend react application to access our api that other people cant for security.

{% highlight ruby %}
 gem 'rack-cors'
{% endhighlight %}

run ``` bundle ``` to install.

Similarly, update config/initializers/cors.rb origins with the url for whitelisting. You can add * to whitelist all URLs or add your react server URL.

<p align="center">
  <img src="/assets/img/cors-rb-file-origin-update.png" alt="cors rb file" />
</p>


### Step 3. Create a Book scaffold with a Title and Body.

Run command ``` rails g scaffold book title: string body: text ``` -- verify controller, model and the db migration file generated. By default it should also create the CRUD methods inside controller with json response.

``` rails db:migrate``` -- to setup db

### Step 4. Update Routes

It's good to have a versioned api namespace so I updated routes.rb file this way.

{% highlight ruby %}
  Rails.application.routes.draw do
    namespace :api do
      namespace :v1 do
        resources :books
      end
    end
  end
{% endhighlight %}

You have to move your books_controller.rb file into a new folder controllers/api/v1 and update the namespace like ``` class Api::V1::BooksController```

### Step 6. Create Books Data in DB.

Open Rails console from terminal and create data. (Pre-requesite: Setup MySql on your machine.)

``` rails c ``` -- should open rails console
{% highlight ruby %}
  Book.create!([
    { title: 'First Book', body: 'Hello book world!' },
    { title: 'Second Book', body: 'Hello sir!' }
  ])
{% endhighlight %}

Now you have 2 Books in the table. Can verify in browser by accessing http://localhost:3000/api/v1/books. You should be able to see json data for both books you created.

<p align="center">
  <img width="300" height="300" src="/assets/img/books-data.png" alt="books json data" />
</p>

Now that we are good with our API. Let's create the React app!

### Step 5. Create React App

``` npx create-react-app bookable ``` -- should create a bookable folder under bookstore.

``` npm run start ``` -- start the server. Use another port. It's 3001 for me. You should be able to see your React app running on this port. Awesome!! Now let's do some changes to display book data on the UI.

#### Note: To only test React app integration you can just update App.js file with the code below. This will just display Hello World! You can see our React app is already setup. To see how to load Books Data, move to the next steps!

{% highlight javascript %}
import './App.css';
function App() {
  return (
    <div className="App">
      <h1> Hello World!</h1>
    </div>
  )
}

export default App;
{% endhighlight %}

<p align="center" border='5px'>
  <img src="/assets/img/hello-world-react.png" alt="Books UI" style="border: 2px solid darkblue;" />
</p>

### Step 6. Update bookable/src/App.js to show Books data

{% highlight javascript %}
import './App.css';
import axios from "axios";
import Books from './components/books';
import { useEffect, useState } from 'react';

const API_URL = "http://localhost:3000/api/v1/books";

async function getAPIData() {
  const response = await axios.get(API_URL);
  return response.data;
}

function App() {
  const [books, setBooks] = useState([]);

  useEffect(()=> {
    let mounted = true;
    getAPIData().then((items) => {
      if (mounted){
        setBooks(items);
      }
  })
  return () => {mounted = false} ;
}, []);

  return (
    <div className="App">
      <h1>Hello</h1>
      <Books books = {books}></Books>
    </div>
  );
}

export default App;

{% endhighlight %}

### Step 7. Create a Books component.

Create a new folder 'components' under bookable/src. And create a Books.js file in it. Update code as below.

{% highlight javascript %}
import React from 'react'

function Books(props) {
  return (
    <div>
      <h1>These books are from the API:</h1>
      {props.books.map((book)=> {
        return (
          <div key={book.id}>
            <h2>{book.title}</h2>
            <p>{book.body}</p>
          </div>
        );
      })}
    </div>
  )
}

export default Books

{% endhighlight%}

As the app is running you should be able to see the books data on the UI on http://localhost:3001.

<p align="center">
  <img src="/assets/img/books-ui.png" alt="Books UI" style="border: 2px solid darkblue;"/>
</p>

#### Congrats! Here you have your fully integrated React JS and Ruby on Rails 7 application! To understand the react concepts like useEffect, useState follow the official [React Documentation](https://react.dev/learn). I will be explaining more concepts like Axios, Redux etc in further blogs. Have a good day!

Tutorial credits - [Deanin](https://youtu.be/sh4WrNGDvQM?si=g3I6l72Oh5ZKWMie) 