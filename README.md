﻿# Daily Shipments App
## A Capstone Project for Udacity's Full Stack Nano Degree

### Motivation
In order to complete Udacity's Full Stack Nano Degree, students are required to submit a final project. They have the option to create a suggested app about a Casting Agency, or create their own app for a different business use case. So, I created this project based on a production app (built with MS Access) used at my work place.

## Introduction
The Daily Shipments App was created to keep track of the orders shipped from Ed's Coffee Depot Company. With this app, the managers can see what orders were processed, who did them, what is their tracking number, and also do some analysis about packages and weights and assess the productivity of the workers at the shipping department.

This is a Python/Flask app that is suitable to be deployed on the Heroku platform. Here's its **public URL**: https://dailyshipments.herokuapp.com
### Roles at the Shipping Dept.
 - Supervisor:
	 - Can read the data from all tables: Carriers, Packagers, Shipments.
	 - Can add to and modify workers in the app.
	 - Can delete processed shipments.
 - Packager:
	 - Can read the data from all tables: Carriers, Packagers, Shipments.
	 - Can create new shipments in the app.

## Getting Started
### Key Dependencies
 - Python 3.7
 - Flask
 - SQLAlchemy
 - Jose
 - Gunicorn
 - PostgreSQL. Installation instructions can be found [here.](https://www.postgresql.org/download/)


Install python dependencies by running:
```bash
pip install -r requirements.txt
```
Create a database called "shipping":
```
createdb shipping
```

Create tables and populate with sample data:
```
psql shipping < shipping.psql
```

### Running the server
- Edit the configuration file **setup.sh** with the proper variables. The run these commands:
```bash
source setup.sh
export FLASK_APP=app.py
export FLASK_ENV=development
flask run
```

#### Authentication
The authentication system used for this project is Auth0. The **setup.sh** contains the required authorization tokens for the app:
 - SUPERVISORJWT has the jwt token for the supervisor role
 - PACKAGERJWT has the jwt token for the packager role

### Endpoints and Restrictions by Roles

| Functionality            | Endpoint                      | Packager         |  Supervisor        | 
| ------------------------ | ----------------------------- | :----------------: | :----------------: | 
| Fetch the list of shipments | GET /shipments                   | :heavy_check_mark: | :heavy_check_mark: | 
| Fetch a list of packagers | GET /packagers                   | :heavy_check_mark: | :heavy_check_mark: | 
| Fetch a list of carriers | GET /carriers                   | :heavy_check_mark: | :heavy_check_mark: | 
| Create a new shipment          | POST /shipments                  | :heavy_check_mark: | :heavy_check_mark: | 
| Create a new packager         | POST /packagers                   |        :x:         | :heavy_check_mark: | 
| Create a new carrier         | POST /carriers                   |        :x:         | :heavy_check_mark: | 
| Modify a specific shipment    | PATCH /shipments/&lt;id&gt; |        :x:         | :heavy_check_mark: | 
| Modify a specific packager    | PATCH /packagers/&lt;id&gt; |        :x:         | :heavy_check_mark: | 
| Modify a specific carriers    | PATCH /carriers/&lt;id&gt; |        :x:         | :heavy_check_mark: | 
| Delete a specific shipment    | DELETE /shipments/&lt;id&gt;     |        :x:         | :heavy_check_mark:| 

#### Sample JSON Payloads for POST and PATCH requests
**POST /shipments**
```
{
    "reference":5000, 
    "carrier_id":6, 
    "packages":2, 
    "weight": 40, 
    "tracking": "QWE232323", 
    "packaged_by":2, 
    "create_date":"2020-11-17"
}
```
- tracking and create_date are optional fields; create_date defaults to current date

**POST /packagers**
```
{
    "first_name": "James",
    "last_name": "Assurance",
    "initials": "JA",
    "active": true
}
```
- active and last_name are optional fields; active defaults to "true"

**POST /carriers**
```
{
    "name": "Test Transport",
    "active": true
}
```
- active is optional and defaults to "true"

**PATCH /shipments/2**
```
{
    "tracking":"Q323223388K", 
    "weight": 12
}
```
- JSON only needs to have the fields with values to be changed.



### Tests
Running tests (Locally)
Make sure PostgreSQL is installed.
- shipping.psql contains database creation commands and sample data
```
createdb shipping_test
psql shipping_test < shipping.psql
python test_app.py
```
