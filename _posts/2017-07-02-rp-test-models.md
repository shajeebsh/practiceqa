---
title: "Rp Test models"
date: 2017-07-02 06:11:14 
author: shajeebhameed
layout: page
slug: rp-test-models
status: publish
---

public class BaseModel { public string _id { get; set; } public string __v { get; set; } } public class Address : BaseModel { public string address1 { get; set; } public string address2 { get; set; } public string phone { get; set; } public string city { get; set; } public string state { get; set; } public string country { get; set; } public string pincode { get; set; } } public class Customer : BaseModel { public string name { get; set; } public string comment { get; set; } public List<Invoice> Invoices { get; set; } } public class Invoice : BaseModel { public string custid { get; set; } public double amount { get; set; } } public class User : BaseModel { public string fname { get; set; } public string lname { get; set; } public int age { get; set; } public string addressid { get; set; } public List<Address> Address { get; set; } // One to Many }
