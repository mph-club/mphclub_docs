# General Overview

This technical document aims to address the structure and relationships of the apps that make up the mph club project. The various apps that make up the mph club project are as follows:
- iOS client
  - Written in swift
- Main api service
  - Written in Go, using the echo framework
- Admin api service
  - Written in Go, using the echo framework
- Main frontend client
  - Written in JS, using the React framework
- Admin frontend client
  - Written in JS, using the React framework
- Database
  - Postgres
- Database Migration tool
  - Written in Go, using a framework called Goose
---

## Deployment/Interactions between apps
All of these apps (except for the iOS app) are deployed on top of a Kubernetes[(k8s)](https://kubernetes.io/) infrastructure/orchestration software. The k8s configuration files currently work with a managed service- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/), although ideally it would be moved to a standalone cloud server like AWS EC2 instance (or [another](https://www.hetzner.com/) cloud server provider) running a [microk8s](https://microk8s.io/) instance on linux.

I made the choice to use k8s to not have to manually run these docker containers on systemd or something similar since there was a lack of resources for a system administrator or deployment specialist. These containers communicate with one another from within the same k8s cluster, providing an extra layer of security as the database is the only aspect not exposed via an ingress. Alternatively, if k8s is not desirable or if another technology is preferred, the entire app can be ported to Docker Compose.

The database migration script handles any changes to the database (new tables, changes in column types, creation of enums, database seeding, etc). It's run as a job on the cluster and must be deleted when completed.

As mentioned above, the Postgres database is not accessible from outside the cluster. Because of how storage works on k8s, we use the `Persistent Volume` [API](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) to configure our storage. Billing is different for volumes than pods (lowest unit of containers running on the GKE node).

The Go binaries are built for running on linux containers and they are also built to have the smallest image size possible (Averaging about 10MB per API)

For more details:
 - [Main API DockerFile](https://github.com/mph-club/api/blob/develop/docker/mphclub-rest-server/Dockerfile)
  - [Admin API Dockerfile](https://github.com/mph-club/cs-api/blob/develop/Dockerfile)

---

## Main API

The main API service provides services for the iOS app and main frontend client. The entire user model has been abstracted out to [AWS Cognito](https://aws.amazon.com/cognito/). This service handles the generation of JWTs for the application as a whole. The API does store some information to associate records with the users based on the unique ID provided by cognito. The information that is associated with that user ID is related to actions taken by the user on the app- associating the drivers license, trip rentals, or listed cars.

#### Public endpoints (endpoints that do not require auth token) are accessible by any client at `mphclub.biz/api/v1`:

`GET /home`

Stubbed out heartbeat resource that currently returns a string- “api home!!!!” when the service is up and working.

Example Return:


`GET /service`

A duplicate stubbed out heartbeat resource that currently returns a string- “api service!!!!” when the service is up and working.

Example Return:


`GET /vehicles?page=&limit=&type=`

This endpoint retrieves the list of approved vehicles to display on both the iOS client and web client. It takes optional query parameters for pagination and also car type.

Example Return:


`GET /vehicles/:id`

This endpoint provides the detailed resource for a single car with the id, a vehicle uuid is a slug here.

Example Return:

`GET /users/:id`

This resource returns the cars listed by the uuid for a user in the slug.

Example Return

`GET /explore`

This resource provides 3 of each car type of SUV, Sports Car, and Sedans.

Example Return

#### Private Endpoints that require a user to be logged in- ie A valid authorization token must be passed in the Authorization header.

`GET /getMyCars`

This resource returns a users listed cars for them to see on their iOS client.

Example Return

`GET /driversLicense`

This resource shows the user their drivers license data

Example Return

`GET /account`

This resource returns the users account settings

Example Return

`GET /reserve`

This resource returns the users reservations

Example Return

`POST /updateUser`
This payload uploads the user based on the authorization token

Example Post body

`POST /listCar`

This payload updates or inserts a new car from the client

Example Post body

Example Return

`POST /uploadCarPhoto`

This payload accepts form files that are pictures of a car and uploads them to S3 then adds to the array of car photos for a vehicle in the database

Example Post Body

Example Return

`POST /uploadUserPhoto`

This payload accepts form files that are pictures of the user and uploads them to S3 then replaces the users photo url in the database

Example Request

Example Return

`POST /driverLicense`

This payload accepts payload relating to the users drivers license info scanned from Jumio.

Example Request

Example Return

`POST /reserve`

This payload reserves a car for the user as identified by the authorization token. The users car dates for that vehicle are blocked out in future requests.

Example Request

`POST /insurance`

The payload for this endpoint contains user supplied insurance information

Example Request

`POST /cardInfo`

The payload for this endpoint sends users card information to the Konnective api
