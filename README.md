# Introduction 
This repository contains all the materials needed for a consultant to execute/deliver a basic Azure Kubernetes demo that focus on a developer centric command line<br>
approach, no DevOps, to containerization and K8s deployment of a simple .NET Core API Weather Forecast service.

# Repository Structure
## **docs**
Documenation for the presenter
- Instructions for creating a Demo VM with all neccessary software for a consultant to successfully run the demo
- Instructions / Steps for executing the demo with all command line commands
## **environment**
Assests required for the demo
- Postman Collection of Rest API Calls to Weather Forecast service
- VM ARM Template
- VM ARM Template Parameters
## **manifest**
K8s YAML manifest file used to deploy Weather Forecast service
## **WeatherForecast**
Weather Forecast service source code
