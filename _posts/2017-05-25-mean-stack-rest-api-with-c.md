---
title: "MEAN Stack REST API with C#"
date: 2017-05-25 16:32:08 
author: shajeebhameed
layout: page
slug: mean-stack-rest-api-with-c
status: publish
---

**MEAN Stack REST API** **MOdels\Movie.js**

var mongoose = require('mongoose');

//Create a movie Schema

var MovieSchema = new mongoose.Schema({

title: {

type:String,

required:true

},

url: {

type:String,

required:true

}

});

//export movie schema

module.exports = MovieSchema;

**Controllers\MOvieController.js**

var restful = require('node-restful');

module.exports = function(app, route) {

//Setup the controller for REST

varrest=restful.model(

'movie',

app.models.movie

).methods(['get','put','post','delete']);

//Register this endpoint with the application

rest.register(app, route);

//Return middleware

returnfunction(req, res, next){

next();

};

};

**index.js**

var express = require('express');

var mongoose = require('mongoose');

var bodyParser = require('body-parser');

var methodOverride = require('method-override');

var _ = require('lodash');

//create the application

var app = express();

//Add middleware necessary for REST API

app.use(bodyParser.urlencoded({extended: true}));

app.use(bodyParser.json());

app.use(methodOverride('X-HTTP-Method-Override'));

//CORS Support - Need to READ UP ABOUT 'CORS'

app.use(function(req, res, next){

res.header('Access-Control-Allow-Origin', '*');

res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE');

res.header('Access-Control-Allow-Hearders', 'Content-Type');

next();

});

app.use('/hello' ,function(req, res, next){

res.send('Hello World');

next();

});

//Connect to MongoDB

mongoose.connect('mongodb://localhost/mean22maydb');

mongoose.connection.once('open', function(){

// Load them models

app.models=require('./models/index');

//Load the routes

varroutes=require('./routes');

_.each(routes, function(controller, route){

app.use(route, controller(app, route));

});

console.log('Listening on the port 3000...');

app.listen(3000);

});

**package.json**

{

"name": "server",

"version": "1.0.0",

"description": "",

"main": "index.js",

"scripts": {

"test": "echo \"Error: no test specified\" && exit 1"

},

"author": "",

"license": "ISC",

"dependencies": {

"body-parser": "^1.17.2",

"express": "^4.15.3",

"lodash": "^4.17.4",

"method-override": "^2.3.9",

"mongoose": "^4.10.1",

"node-restful": "^0.2.6"

}

}

**routes.js**

module.exports ={

'/movie':require('./controllers/MovieController')

};

**C# Client**

**MovieUtil.cs**

using System; using System.Collections.Generic; using System.Linq; using System.Net.Http; using System.Net.Http.Headers; using System.Text; using System.Threading.Tasks; namespace WinApp { public class MovieUtil { private const string BASE_URL = "http://localhost:3000"; private const string GET_ALL_REQ = "/movie"; private const string GET_REQ = "/movie/{0}"; private const string POST_REQ = "/movie"; private const string PUT_REQ = "/movie/{0}"; private const string DELETE_REQ = "/movie/{0}"; private const string SuccessMessage = "SUCCESS"; HttpClient _httpClient = null; public MovieUtil() { InitializeAPI(); } private void InitializeAPI() { _httpClient = new HttpClient(); _httpClient.BaseAddress = new Uri(BASE_URL); //_httpClient.DefaultRequestHeaders.Add("appkey", "myapp_key"); // Add an Accept header for JSON format. _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json")); } public IEnumerable<Movie> GetMovies(out string ErrorMessage) { IEnumerable<Movie> returnValue = null; try { HttpResponseMessage response = _httpClient.GetAsync(GET_ALL_REQ).Result; if (response.IsSuccessStatusCode) { var movies = response.Content.ReadAsAsync<IEnumerable<Movie>>().Result; returnValue = movies; ErrorMessage = SuccessMessage; } else { ErrorMessage = string.Format("Error Code: {0}, Message :{1}", response.StatusCode, response.ReasonPhrase); } } catch (Exception ex) { throw ex; } return returnValue; } public Movie GetMovie(string id, out string ErrorMessage) { Movie returnValue = null; try { string requestString = string.Format(GET_REQ, id); HttpResponseMessage response = _httpClient.GetAsync(requestString).Result; if (response.IsSuccessStatusCode) { returnValue = response.Content.ReadAsAsync<Movie>().Result; ErrorMessage = SuccessMessage; } else { ErrorMessage = string.Format("Error Code: {0}, Message :{1}", response.StatusCode, response.ReasonPhrase); } } catch (Exception ex) { throw ex; } return returnValue; } public bool SaveMovie(Movie movie, out string ErrorMessage) { try { HttpResponseMessage response = string.IsNullOrEmpty(movie._id) ? _httpClient.PostAsJsonAsync(POST_REQ, movie).Result : _httpClient.PutAsJsonAsync(string.Format(PUT_REQ, movie._id), movie).Result; if (response.IsSuccessStatusCode) { ErrorMessage = SuccessMessage; return true; } else { ErrorMessage = string.Format("Error Code: {0}, Message :{1}", response.StatusCode, response.ReasonPhrase); return false; } } catch (Exception ex) { throw ex; } } public bool DeleteMovie(string id, out string ErrorMessage) { try { string requestString = string.Format(DELETE_REQ, id); HttpResponseMessage response = _httpClient.DeleteAsync(requestString).Result; if (response.IsSuccessStatusCode) { ErrorMessage = SuccessMessage; return true; } else { ErrorMessage = string.Format("Error Code: {0}, Message :{1}", response.StatusCode, response.ReasonPhrase); return false; } } catch (Exception ex) { throw ex; } } } }

**MOvie.cs**

public class Movie { public string _id { get; set; } public string title { get; set; } public string url { get; set; } //public string __v { get; set; } }

**package.config**

<packages> <package id="Microsoft.AspNet.WebApi.Client" version="5.2.3" targetFramework="net452" /> <package id="Newtonsoft.Json" version="10.0.2" targetFramework="net452" /> </packages>

**MovieList.Form**

using System; using System.Collections.Generic; using System.ComponentModel; using System.Data; using System.Drawing; using System.Linq; using System.Text; using System.Threading.Tasks; using System.Windows.Forms; using System.Net.Http; using System.Net.Http.Headers; using Newtonsoft.Json.Linq; using Newtonsoft.Json; namespace WinApp { public partial class MovieList : Form { public MovieList() { InitializeComponent(); } MovieUtil objMovieUtil = null; private string ErrorMsg; private void Form1_Load(object sender, EventArgs e) { objMovieUtil = new MovieUtil(); BindMovies(); } private void BindMovies() { var movies = objMovieUtil.GetMovies(out ErrorMsg); dgMovies.DataSource = movies; cmbMovies.DataSource = movies; cmbMovies.DisplayMember = "title"; cmbMovies.ValueMember = "_id"; } private void BindMovie(string id) { var movie = objMovieUtil.GetMovie(id, out ErrorMsg); List<Movie> movies = new List<Movie>(); movies.Add(movie); dgMovies.DataSource = movies; } private void btnExit_Click(object sender, EventArgs e) { Application.Exit(); } private void btnDelete_Click(object sender, EventArgs e) { string selectedMovideId = dgMovies.SelectedCells[0].Value.ToString(); if (!string.IsNullOrEmpty(selectedMovideId)) { objMovieUtil.DeleteMovie(selectedMovideId, out ErrorMsg); BindMovies(); } } private void btnAddNew_Click(object sender, EventArgs e) { MovieAdd objAddMovie = new MovieAdd(); objAddMovie.ShowDialog(); BindMovies(); } private void btnGet_Click(object sender, EventArgs e) { var movieId = cmbMovies.SelectedValue.ToString(); BindMovie(movieId); } private void btnEdit_Click(object sender, EventArgs e) { MovieAdd objAddMovie = new MovieAdd(); string selectedMovideId = dgMovies.SelectedCells[0].Value.ToString(); if (!string.IsNullOrEmpty(selectedMovideId)) { objAddMovie.SelectedMovie = new Movie { _id = dgMovies.SelectedCells[0].Value.ToString(), title = dgMovies.SelectedCells[1].Value.ToString(), url = dgMovies.SelectedCells[2].Value.ToString(), }; objAddMovie.ShowDialog(); BindMovies(); } } } }

**MovieAdd.Form**

using System; using System.Collections.Generic; using System.ComponentModel; using System.Data; using System.Drawing; using System.Linq; using System.Text; using System.Threading.Tasks; using System.Windows.Forms; namespace WinApp { public partial class MovieAdd : Form { public Movie SelectedMovie { get; set; } public MovieAdd() { InitializeComponent(); } private void btnSave_Click(object sender, EventArgs e) { if (!objMovieUtil.SaveMovie(new Movie() { _id = SelectedMovie._id, title = txtMovieName.Text, url = txtUrl.Text }, out ErrorMsg)) { MessageBox.Show(ErrorMsg); } this.Dispose(); } private void btnCancel_Click(object sender, EventArgs e) { this.Dispose(); } private string ErrorMsg; MovieUtil objMovieUtil = null; private void AddMovie_Load(object sender, EventArgs e) { objMovieUtil = new MovieUtil(); if(SelectedMovie != null) { txtMovieName.Text = SelectedMovie.title; txtUrl.Text = SelectedMovie.url; } } private void btnAdd_Click(object sender, EventArgs e) { if(!objMovieUtil.SaveMovie(new Movie() { title = txtMovieName.Text, url = txtUrl.Text }, out ErrorMsg)) { MessageBox.Show(ErrorMsg); } this.Dispose(); } } }
