# Daily Shipments App
## A Capstone Project for Udacity's Full Stack Nano Degree

### Motivation
In order to complete Udacity's Full Stack Nano Degree, students are required to submit a final project. They have the option to create a suggested app about a Casting Agency, or create their own app for a different business use case. So, I created this project based on a production app (built with MS Access) used at my work place.

## Introduction
The Daily Shipments App is a Python/Flask based app which was created to keep track of the orders shipped from Ed's Coffee Depot Company. With this app, the managers can see what orders were processed, who did them, what is their tracking number, and also do some analysis about packages and weights and assess the productivity of the workers at the shipping department.

This is a  app that is suitable to be deployed on the Heroku platform. Here's its **public URL**: https://dailyshipments.herokuapp.com

## Getting Started

### Installing Dependencies

#### Python 3.7

Follow instructions to install the latest version of python for your platform in the [python docs](https://docs.python.org/3/using/unix.html#getting-and-installing-the-latest-version-of-python)

Note: Before installing the packages for this project, it's best to create and activate a [virtual environment](https://docs.python.org/3/library/venv.html)

#### PIP Dependencies


In the shipping directory, run the following to install all necessary dependencies:

```bash
pip install -r requirements.txt
```

#### PostgreSQL
 - PostgreSQL. Installation instructions can be found [here.](https://www.postgresql.org/download/)

Create a database called "shipping":
```
createdb shipping
```

Create tables and populate with sample data:
```
psql shipping < shipping.psql
```

### Running the server
- Edit the configuration file ``setup.sh`` with the proper variables. The run these commands:
```bash
source setup.sh
export FLASK_APP=app.py
export FLASK_ENV=development
flask run
```
For Windows, use the command ``set`` instead of ``export``. Note windows uses ``.bat`` files instead of ``.sh``.

## Data Modeling
#### database/models.py
The schema for the database and helper methods to simplify API behavior can be found in models.py:
- There are 3 tables in the database:
    - *carrier*, which stores the name of the shipping carriers used to dispatch the shipments;
    - *packager*, which stores the info about the packagers (workers) in the shipping department. The field "initials" can be filled with the first letter of the packager's first name and last name.
    - *shipment*, which stores all the daily activities in the shipping dept. Some column details:
        - *reference*, stores the invoice number for the order processed;
        - *tracking*, stores the carrier tracking number. It can be blank because not all carriers have a tracking system;
        - *packages*, and *weight*, have a database level constraint to only allow values greater than zero.
        - *carrier_id*, and *packaged_by* are foreign keys to the tables *carrier* and *packager* respectively.


## API Architecture and Testing
### Endpoint Library
*@app.errorhandler* decorators were used to format error responses as JSON objects. Custom *@requires_auth* decorator were used for Authorization based
on roles of the user. 

Two types of users are allowed for this API:
 - Supervisor:
	 - Can read the data from all tables: Carriers, Packagers, Shipments.
	 - Can add to and modify workers in the app.
	 - Can delete processed shipments.
 - Packager:
	 - Can read the data from all tables: Carriers, Packagers, Shipments.
	 - Can create new shipments in the app.

The authentication system used for this project is [Auth0](https://auth0.com). 
A token needs to be passed to each endpoint.
The ``setup.sh`` stores the required authorization tokens for the app in these environment variables:
 - SUPERVISORJWT, which has the jwt token for the *supervisor* user
 - PACKAGERJWT, which has the jwt token for the *packager* user

For getting new tokens, go to [this url](https://edutest2.auth0.com/authorize?audience=coffee&response_type=token&client_id=pp7mNhWac6AHCv0KlaW95t3U4t4VkiWr&redirect_uri=https://127.0.0.1:5000/login-results) and:
 - for the supervisor role, login with supervisor@mailinator.com and password Xkyuze404me
 - for the packager role, login with edutest4@mailinator.com and password were#,1234
 - extract the token part from the URL bar.

#### GET /packagers
Returns the list of packagers.

Required authentication role: `get:packagers`

User access: supervisor and packager

Sample cURL (application running on localhost):
```
curl -H "Authorization: Bearer $PACKAGERJWT" http://127.0.0.1:5000/packagers
```
Sample response output:
```
{
   "packagers":[
      {
         "Packager Initials":"JD",
         "id":1,
         "is_active":true
      },
      {
         "Packager Initials":"MS",
         "id":2,
         "is_active":true
      },
   ],
   "success":true
}
```
#### POST /packagers
Creates a new Packager.

Required authentication role: `post:packagers`

User access: supervisor

Required fields: first_name, initials

Sample cURL (application running on localhost):
```
curl -d '{"first_name":"Greg", "last_name":"Davis", "initials":"GD", "active":true}' -H "Content-Type: application/json" -H "Authorization: Bearer $SUPERVISORJWT" -X POST http://127.0.0.1:5000/packagers
```
Sample response output:
```
{
  "packager": {
    "Packager Initials": "GD",
    "id": 4,
    "is_active": true
  },
  "success": true
}
```

#### PATCH /packagers/{package_id}
Modifies Packager's data

Required authentication role: `patch:packagers`

User access: supervisor

Sample cURL (application running on localhost):
```
curl -d '{"first_name":"Jim"}' -H "Content-Type: application/json" -H "Authorization: Bearer $SUPERVISORJWT" -X PATCH http://127.0.0.1:5000/packagers/1
```
Sample response output:
```
{
  "packager": {
    "Packager Initials": "JD",
    "id": 1,
    "is_active": true
  },
  "success": true
}
```

#### GET /carriers
Returns the list of carriers

Required authentication role: `get:carriers`

User access: supervisor and packager

Sample cURL (application running on localhost):
```
curl -H "Authorization: Bearer $PACKAGERJWT" http://127.0.0.1:5000/carriers
```
Sample response output:
```
{
  "carriers": [
    {
      "Carrier": "Stephan Express",
      "id": 1,
      "is_active": true
    },
    {
      "Carrier": "Purolator",
      "id": 2,
      "is_active": true
    }
  ],
  "success": true
}
```

#### POST /carriers
Creates a new Carrier.

Required authentication role: `post:carrier`

User access: supervisor

Required fields: name

Sample cURL (application running on localhost):
```
curl -d '{"name":"DHL"}' -H "Content-Type: application/json" -H "Authorization: Bearer $SUPERVISORJWT" -X POST http://127.0.0.1:5000/carriers

```
Sample response output:
```
{
  "carrier": {
    "Carrier": "DHL",
    "id": 6,
    "is_active": true
  },
  "success": true
}
```

#### PATCH /carriers/{carrier_id}
Modifies Carrier's data.

Required authentication role: `patch:carrier`

User access: supervisor


Sample cURL (application running on localhost):
```
curl -d '{"name":"ST Transport"}' -H "Content-Type: application/json" -H "Authorization: Bearer $SUPERVISORJWT" -X PATCH http://127.0.0.1:5000/carriers/1

```
Sample response output:
```
{
  "carrier": {
    "Carrier": "ST Transport",
    "id": 1,
    "is_active": true
  },
  "success": true
}
```
#### GET /shipments
Returns the list of shipments.

Required authentication role: `get:shipments`

User access: supervisor and packager

Sample cURL (application running on localhost):
```
curl -H "Authorization: Bearer $PACKAGERJWT" http://127.0.0.1:5000/shipments
```
Sample response output:
```
{
  "shipments": [
    {
      "Date": "Sat, 14 Nov 2020 09:35:00 GMT",
      "Packaged By": 1,
      "Packages": 3,
      "Reference": 45322,
      "Weight": 30,
      "id": 1
    },
    {
      "Date": "Sat, 14 Nov 2020 10:35:00 GMT",
      "Packaged By": 2,
      "Packages": 2,
      "Reference": 45323,
      "Weight": 30,
      "id": 2
    },
    {
      "Date": "Sat, 14 Nov 2020 14:20:00 GMT",
      "Packaged By": 2,
      "Packages": 4,
      "Reference": 45350,
      "Weight": 30,
      "id": 3
    }
  ],
  "success": true
}
```

#### POST /shipments
Creates a new shipment.

Required authentication role: `post:shipments`

User access: supervisor or packager

Required fields:
- reference
- carrier_id
- packages
- weight
- packaged_by

*create_date* defaults to current date/time

Sample cURL (application running on localhost):
```
curl -d '{"reference":5000, "carrier_id":2, "packages":2, "weight": 40, "tracking": "QWE232323", "packaged_by":2, "create_date":"2020-11-17"}' -H "Content-Type: application/json" -H "Authorization: Bearer $PACKAGERJWT" -X POST  http://127.0.0.1:5000/shipments  
```
Sample response output:
```
{
  "shipment": {
    "Date": "Tue, 17 Nov 2020 00:00:00 GMT",
    "Packaged By": 2,
    "Packages": 2,
    "Reference": 5000,
    "Weight": 40,
    "id": 10
  },
  "success": true
}
```

#### PATCH /shipments/{shipment_id}
Modifies shipment data.

Required authentication role: `patch:shipments`

User access: supervisor

Sample cURL (application running on localhost):
```
curl -d '{"reference":99800}' -H "Content-Type: application/json" -H "Authorization: Bearer $SUPERVISORJWT" -X PATCH  http://127.0.0.1:5000/shipments/6 
```
Sample response output:
```
{
  "shipment": {
    "Date": "Sun, 15 Nov 2020 09:50:00 GMT",
    "Packaged By": 3,
    "Packages": 3,
    "Reference": 99800,
    "Weight": 18.9,
    "id": 6
  },
  "success": true
}
```
#### DELETE /shipments/{shipment_id}
Deletes a shipment

Required authentication role: `delete:shipments`

User access: supervisor

Sample cURL (application running on localhost):
```
curl -H "Authorization: Bearer $SUPERVISORJWT" -X DELETE  http://127.0.0.1:5000/shipments/7
```
Sample response output:
```
{
  "deleted": {
    "Date": "Sun, 15 Nov 2020 10:35:00 GMT",
    "Packaged By": 1,
    "Packages": 3,
    "Reference": 45380,
    "Weight": 30,
    "id": 7
  },
  "success": true
}
```

#### GET /
Returns the app's homepage.

In the future it will return an html response. For now just a JSON response.

Sample response output:
```
{
  "message": "Hurrah! You got to the home of the Daily Shipping App!",
  "success": true
}
```







### Tests
Running tests (Locally)
Make sure PostgreSQL is installed.
- shipping.psql contains database creation commands and sample data
```
createdb shipping_test
psql shipping_test < shipping.psql
python test_app.py
```

## Deployment
This is app is suitable to be deployed on the Heroku platform. An example deployment can be found at: https://dailyshipments.herokuapp.com

There is no frontend for this app yet.
