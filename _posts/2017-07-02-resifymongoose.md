---
title: "ResifyMongoose"
date: 2017-07-02 05:42:28 
author: shajeebhameed
layout: page
slug: resifymongoose
status: publish
---

**MongoAdmin** https://github.com/mrvautin/adminMongo **Package.json**

{

"name": "rest5",

"version": "0.1.0",

"scripts": {

"start": "node app.js"

},

"dependencies": {

"express-restify-mongoose": "^4.1.3",

"mongoose": "^4.0.0"

}

}

**App.js**

var restify = require('restify');

var restifyMongoose = require('restify-mongoose');

var mongoose = require('mongoose');

var server = restify.createServer({

name:'restify.mongoose.examples.notes',

version:'1.0.0'

});

mongoose.connect('mongodb://localhost/meandev')

server.use(restify.acceptParser(server.acceptable));

server.use(restify.queryParser());

server.use(restify.bodyParser());

// Models and API - START

var AddressSchema = new mongoose.Schema({

address1: { type:String, required:true },

address2: { type:String },

phone: { type:String, required:true },

city: { type:String, required:true },

state: { type:String, required:true },

country: { type:String, required:true },

pincode: { type:String, required:true },

})

var AddressModel = mongoose.model('Address', AddressSchema)

var address = restifyMongoose(AddressModel);

server.get('/address', address.query());

server.get('/address/:id', address.detail());

server.post('/address', address.insert());

server.put('/address/:id', address.update());

server.del('/address/:id', address.remove());

var UserSchema = new mongoose.Schema({

fname : { type :String, required :true },

lname : { type :String, required :true },

age : { type :Number, required :true },

addressid: { type:mongoose.Schema.Types.ObjectId, ref:'Address', required :true },

});

var UserModel = mongoose.model('User', UserSchema);

var user = restifyMongoose(UserModel);

server.post('/user', user.insert());

server.put('/user/:id', user.update());

server.del('/user/:id', user.remove());

server.get('/user', function(req, res, next) {

varlookupQuery= { from:"addresses", localField:"addressid", foreignField:"_id", as:"Address" };

UserModel.aggregate().lookup(lookupQuery).exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

})

});

server.get('/user/:id', function(req, res, next) {

varmatchQuery= {'_id':mongoose.Types.ObjectId(req.params.id)};

varlookupAddressQuery= { from:"addresses", localField:"addressid", foreignField:"_id", as:"Address" };

UserModel.aggregate().match(matchQuery).lookup(lookupAddressQuery)

.limit(1).exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

})

});

server.get('/user/:search/:option', function(req, res, next) {

varqueryContainsString= {fname:{'$regex' :req.params.search, '$options' :'i'}};//'string'

varqueryStartsWith= {fname:{'$regex' :'^'+req.params.search, '$options' :'i'}}; //'^string'

varqueryExactCase= {fname:{'$regex' :'^'+req.params.search+'$', '$options' :'i'}}; //'^string$'

varqueryEndsWith= {fname:{'$regex' :req.params.search+'$', '$options' :'i'}};//'string$'

varqueryNotContainsString= {fname:{'$regex' :'^((?!'+req.params.search+').)*$', '$options' :'i'}};//'^((?!string).)*$'

varquery=queryContainsString;

if(req.params.option){

switch (req.params.option) {

case0: query=queryContainsString; break;

case1: query=queryStartsWith; break;

case2: query=queryExactCase; break;

case3: query=queryEndsWith; break;

case4: query=queryNotContainsString; break;

default: query=queryContainsString;

}

}

UserModel.find(query)

.exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

});

})

var CustomerSchema = new mongoose.Schema({

name: { type:String, required:true },//

comment: { type:String }

})

var CustomerModel = mongoose.model('Customer', CustomerSchema)

var customer = restifyMongoose(CustomerModel);

server.get('/customer', customer.query());

server.get('/customer/:id', customer.detail());

server.post('/customer', customer.insert());

server.put('/customer/:id', customer.update());

server.patch('/customer/:id', customer.update());

server.del('/customer/:id', customer.remove());

server.get('/customer/:search/:option', function(req, res, next) {

varqueryContainsString= {name:{'$regex' :req.params.search, '$options' :'i'}};//'string'

varqueryStartsWith= {name:{'$regex' :'^'+req.params.search, '$options' :'i'}}; //'^string'

varqueryExactCase= {name:{'$regex' :'^'+req.params.search+'$', '$options' :'i'}}; //'^string$'

varqueryEndsWith= {name:{'$regex' :req.params.search+'$', '$options' :'i'}};//'string$'

varqueryNotContainsString= {name:{'$regex' :'^((?!'+req.params.search+').)*$', '$options' :'i'}};//'^((?!string).)*$'

varquery=queryContainsString;

if(req.params.option){

switch (req.params.option) {

case0: query=queryContainsString; break;

case1: query=queryStartsWith; break;

case2: query=queryExactCase; break;

case3: query=queryEndsWith; break;

case4: query=queryNotContainsString; break;

default: query=queryContainsString;

}

}

CustomerModel.find(query)

.exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

});

})

var InvoiceSchema = new mongoose.Schema({

custid: { type:mongoose.Schema.Types.ObjectId, ref:'Customer' },

amount: { type:Number, required:true }

})

var InvoiceModel = mongoose.model('Invoice', InvoiceSchema)

/**

* Model Schema

* const Todo = require('../models/todo')

*/

var invoice = restifyMongoose(InvoiceModel);

server.get('/invoice', invoice.query());

//server.get('/invoice/:id', invoice.detail());

server.post('/invoice', invoice.insert());

server.patch('/invoice/:id', invoice.update());

server.del('/invoice/:id', invoice.remove());

server.get('/invoice/:customerid', function(req, res, next) {

varquery= { custid:req.params.customerid };

// InvoiceModel.find(query, function(err, doc) { if (err) { log.error(err)

// return next(new errors.InvalidContentError(err.errors.name.message)) } res.json(doc) next() })

InvoiceModel.find(query)

.exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

});

})

// db.invoice.aggregate([{$lookup: { from: "customer", localField: "customer", foreignField: "_id", as: "CD" }}]).pretty()

// db.invoice.aggregate([{$lookup: { from: "Customer", localField: "customer", foreignField: "_id", as: "CD" },

// $project: { "CD.name":1, "_id":0}}])

server.get('/inv1', function(req, res, next){

InvoiceModel.aggregate({

$lookup: { from:"customers", localField:"custid", foreignField:"_id", as:"Customer" },

},function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

})

});

server.get('/custinv', function(req, res, next){

CustomerModel.aggregate()

.lookup({ from: "invoices", localField: "_id", foreignField: "custid", as: "Invoices" })

.exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

});

//res.send('hello ' + req.params.ppp);

});

server.get('/custinv/:id', function(req, res, next){

varmatchQuery= {'_id':mongoose.Types.ObjectId(req.params.id)};

varlookupQuery= { from:"invoices", localField:"_id", foreignField:"custid", as:"Invoices" };

CustomerModel.aggregate().match(matchQuery).lookup(lookupQuery)

.exec(function(err, doc) {

if (err) {

log.error(err)

returnnext(newerrors.InvalidContentError(err.errors.name.message))

}

res.json(doc)

next()

});

//res.send('hello ' + req.params.ppp);

});

server.get('/mak/:id/:includes', function(req, res, next){

res.send('Hello mak you have sent id ='+req.params.id+', and you have sent includes ='+req.params.includes);

});

// Models and API - END

server.listen(3000, function () {

console.log('%s listening at %s', server.name, server.url);

});1
