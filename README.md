# Sito-Schedulables (specific use-case demo)
Code repository for a specific use-case of the sito_schedulables library.

The example provided here is more complex than the one used in the howto/documentation here :

https://github.com/sitodav/sito_schedulables (use this link if you want to learn more about the library itself, for example what it is , how it works, how to install etc..)

You can find a video of this use case on youtube :

**YOUTUBELINK**

## Use Case description

The code contained in this repository contains 3 different microservices:
- One called **studentsMicroservice** : it exposes two APIs. One for person registration , and one for person enrolment as student.
- One called **paymentsMicroservice** : it exposes only one API for payment-documents generation for a given student enrolment.
- The **orchestrator**.
- The **orchestrator dashboard (front end)**

The complete flow of operations is the following:
- Client sends list of persons data for a given year to the **[studentsMicroservice]/students/person/registration** endpoint. 
- The endpoint registers the persons (if not already present) and returns them-
- The client use the previous output, as request payload for the **[studentsMicroservice]/students/person/enrolment** endpoint.
- Here the microservice create a link between the persons and the year as year-enrolment. Now the persons are students for the year.
- The client uses previous step response (the students enrolments) as request payload for the other microservice's endpoint **[paymentsMicroservice]/payment_info/generation**.
- This endpoints create documents for the students enrolment to the given year, and returns to the client.


## Installation

The projects are 3 Maven projects.
Build them with **mvn clean install** in the root (where the POMs are) and launch them with **mvn spring-boot:run**.
All the projects need a running database (for the sake of simplicity they all point to the same DB, which is not a good practice for microservices, but this is an example!).

You need to install your Mysql instance, and change the *application.properties* following entries in each project.

```
spring.datasource.url=jdbc:mariadb://localhost:3306/students_payments_DB?createDatabaseIfNotExist=true&useUnicode=true&characterEncoding=utf-8&autoReconnect=true
spring.datasource.username=root
spring.datasource.password=password
```

For the angular project (the front end for the dashboard) just use **npm install** , change the values (if you are not deploying locally or with the default back-end ports) of the *environment* file in the project.

Run **ng serve**

## Doing things without orchestration, via REST-API

First let'see how things work without the orchestration library.
We have to contact each different microservice with its specific request payload.

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/without.jpg "Optional title")

If we go the swagger-ui page for the project (if you start the project with the provided application.properties it will be http://localhost:8087/students/swagger-ui.html# )
we can start with the first step : registering a person.

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/api.jpg "Optional title")

### Registering some persons

We will send a list of persons to be registered to **/students/person/registration**
Our payload will be :
```
{

  "persons": [
    {
      "bdayAsString": "28/04/1988",
      "cf": "RTIDDZI8D28FAAAN",
      "name": "Davide", 
      "surname": "Sito"
    },
    {
      "bdayAsString": "03/09/1992",
      "cf": "NMIFDHH8D28FHUYR",
      "name": "Maria", 
      "surname": "Lombradi"
    },
    {
      "bdayAsString": "12/05/1991",
      "cf": "NUUROPI8D28FURER",
      "name": "Mario", 
      "surname": "Rossi"
    },
    
    {
      "bdayAsString": "03/07/1993",
      "cf": "JBOROPI7D28FURZZ",
      "name": "James", 
      "surname": "Borg"
    }
  ],
  "yearMatriculation": "2024"
}
```

The endpoint will return the created persons response.
```
{
  "yearMatriculation": "2024" ,
  "persons": [
    {
      "id": 1,
      "studentNumber": null,
      "cf": "RTIDDZI8D28FAAAN",
      "name": "Davide",
      "surname": "Sito",
      "bdayAsString": "28/04/1988"
    },
    {
      "id": 2,
      "studentNumber": null,
      "cf": "NMIFDHH8D28FHUYR",
      "name": "Maria",
      "surname": "Lombradi",
      "bdayAsString": "03/09/1992"
    },
    {
      "id": 3,
      "studentNumber": null,
      "cf": "NUUROPI8D28FURER",
      "name": "Mario",
      "surname": "Rossi",
      "bdayAsString": "12/05/1991"
    },
    {
      "id": 4,
      "studentNumber": null,
      "cf": "JBOROPI7D28FURZZ",
      "name": "James",
      "surname": "Borg",
      "bdayAsString": "03/07/1993"
    }
  ] 
  
   
}
```

### Enrolment for the created persons 

Now we take this response, and send it as payload request for the other students API endpoint : **[studentsMicroservice]/students/person/enrolment** .

The endpoint response will be :
```
{
   
  "enrolment": [
    {
      "enrolmentYearId": 13,
      "year": "2024",
      "personId": 1,
      "studentNumber": "0ffd83f2-5940-4fea-a860-f4afa835287b"
    },
    {
      "enrolmentYearId": 14,
      "year": "2024",
      "personId": 2,
      "studentNumber": "98110543-52f4-42cc-9b5c-8612c136372b"
    },
    {
      "enrolmentYearId": 15,
      "year": "2024",
      "personId": 3,
      "studentNumber": "1f602e4d-fb8b-47ba-9dbc-06e1f949284d"
    },
    {
      "enrolmentYearId": 16,
      "year": "2024",
      "personId": 4,
      "studentNumber": "1ce2a370-d5e7-4856-94ed-9e9cc89c371f"
    }
  ]
 
}
```
This means the persons have been enrolled for the year 2024.

### Generating payments

Now we can go to the other microservice (payments) swagger-ui page 

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/api2.jpg "Optional title")

Here we have the endpoint to use for creating payment infos/documents generation given one or more students enrolments for a year.

So we can send the response from the last call (students enrolments) to the **[paymentsMicroservice]/payment_info/generation** endpoint.

The last response will be :
```
{
   
  "payments": [
    {
      "idPaymentInfo": 18,
      "studentNumber": "0ffd83f2-5940-4fea-a860-f4afa835287b",
      "year": "2024",
      "fee": 350,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642826
    },
    {
      "idPaymentInfo": 19,
      "studentNumber": "0ffd83f2-5940-4fea-a860-f4afa835287b",
      "year": "2024",
      "fee": 1650,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642826
    },
    {
      "idPaymentInfo": 20,
      "studentNumber": "98110543-52f4-42cc-9b5c-8612c136372b",
      "year": "2024",
      "fee": 350,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642865
    },
    {
      "idPaymentInfo": 21,
      "studentNumber": "98110543-52f4-42cc-9b5c-8612c136372b",
      "year": "2024",
      "fee": 1650,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642865
    },
    {
      "idPaymentInfo": 22,
      "studentNumber": "1f602e4d-fb8b-47ba-9dbc-06e1f949284d",
      "year": "2024",
      "fee": 350,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642868
    },
    {
      "idPaymentInfo": 23,
      "studentNumber": "1f602e4d-fb8b-47ba-9dbc-06e1f949284d",
      "year": "2024",
      "fee": 1650,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642868
    },
    {
      "idPaymentInfo": 24,
      "studentNumber": "1ce2a370-d5e7-4856-94ed-9e9cc89c371f",
      "year": "2024",
      "fee": 350,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642873
    },
    {
      "idPaymentInfo": 25,
      "studentNumber": "1ce2a370-d5e7-4856-94ed-9e9cc89c371f",
      "year": "2024",
      "fee": 1650,
      "totalAmount": 2000,
      "status": "PUBLISHED_TOPAY",
      "fromDate": 1701096642873
    }
  ] 
}
```

This is the whole flow without using the orchestrator (like you would before using/installing the **sito_schedulables** library) .


## Using SITO_SCHEDULABLES orchestrator, via REST-API

With the orchestrator the client only sends the starting payload with some metadatas (that tell the orchestrator what task have to be executed and in what order/dependency).

The orchestrator does the rest, contacting each individual microservice, under a whole distribute transaction (if you want it to be).


![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/with.jpg "Optional title")

If you go to the orchestrator swagger-ui page at **http://localhost:8082/orchestrator/swagger-ui.html** you can use the task composer endpoint to create new groups of tasks.
For a detailed explanation about task trees/groups, you can read the documentation here https://github.com/sitodav/sito_schedulables .
So now we will define a group of tasks, linked sequentially so that when one finishes the next starts.
These tasks will be our 3 steps : **person registration**, **student year enrolment**, **payment generation for enrolment**

Our payload will look like this:

```
{
  "dependencyList": [
    "-1",
    "0",
    "1"
  ],
  "taskList": [
    {
      "programmedAt": "16/06/2022 12:03:00",
      "forwardResult": true,
      "persons": [
        {
          "bdayAsString": "28/04/1988",
          "cf": "RTIDDZI8D28FAAAN",
          "name": "Davide",
          "surname": "Sito"
        },
        {
          "bdayAsString": "03/09/1992",
          "cf": "NMIFDHH8D28FHUYR",
          "name": "Maria",
          "surname": "Lombradi"
        },
        {
          "bdayAsString": "12/05/1991",
          "cf": "NUUROPI8D28FURER",
          "name": "Mario",
          "surname": "Rossi"
        },
        {
          "bdayAsString": "03/07/1993",
          "cf": "JBOROPI7D28FURZZ",
          "name": "James",
          "surname": "Borg"
        }
      ],
      "yearMatriculation": "2024",
      "taskType": "STUDENT_PERSON_REGISTRATIONS"
    },
    {
      "readForwardedResult": true,
      "forwardResult": true,
      "taskType": "STUDENT_PERSON_ENROLMENTS"
    },
    {
      "taskType": "PAYMENT_PAYMENTSINFO_GENERATIONS",
      "readForwardedResult": true
    }
  ],
  "distributedCommit": false
}
```
The request is telling the orchestrator to create 3 different tasks.
Each task is present, in the taskList array, as a JSON object with the original payload (persons list) + some metadata 

(programmedAt is the date when the first task, the root, has to start, forwardResults tell the orchestrator to forward the task result as input request for the following, and readForwarded results tells the orchestrator to read the task request from a previous response)

The endpoint will create the group and respond with:
```
{
 
  "taskList": [
    {
      "taskId": "5238fcfa-fefe-4cc1-9c18-eba675954a06",
      "createDate": null,
      "endDate": null,
      "startDate": null,
      "status": "CREATED",
      "programmedAt": null,
      "error": null,
      "taskType": "STUDENT_PERSON_REGISTRATIONS",
      "percentage": null,
      "payloadRequestJson": null,
      "microserviceRootSwaggerUrl": null,
      "distributedCommit": false,
      "forwardResult": true,
      "readForwardedResult": false,
      "serializedTaskResultAsString": null,
      "dependsOn": [
        null
      ],
      "children": [
        {
          "taskId": "ae479092-b56f-4f96-8c35-4b78b9f5db12",
          "createDate": null,
          "endDate": null,
          "startDate": null,
          "status": "CREATED",
          "programmedAt": null,
          "error": null,
          "taskType": "STUDENT_PERSON_ENROLMENTS",
          "percentage": null,
          "payloadRequestJson": null,
          "microserviceRootSwaggerUrl": null,
          "distributedCommit": false,
          "forwardResult": true,
          "readForwardedResult": true,
          "serializedTaskResultAsString": null,
          "dependsOn": [
            "#5238fcfa-fefe-4cc1-9c18-eba675954a06"
          ],
          "children": [
            {
              "taskId": "ccbf93dd-01aa-4ab1-8707-a67e3fb124b5",
              "createDate": null,
              "endDate": null,
              "startDate": null,
              "status": "CREATED",
              "programmedAt": null,
              "error": null,
              "taskType": "PAYMENT_PAYMENTSINFO_GENERATIONS",
              "percentage": null,
              "payloadRequestJson": null,
              "microserviceRootSwaggerUrl": null,
              "distributedCommit": false,
              "forwardResult": false,
              "readForwardedResult": true,
              "serializedTaskResultAsString": null,
              "dependsOn": [
                "#ae479092-b56f-4f96-8c35-4b78b9f5db12"
              ],
              "children": []
            }
          ]
        }
      ]
    }
  ],
  "dependencyList": [
    null,
    "#5238fcfa-fefe-4cc1-9c18-eba675954a06",
    "#ae479092-b56f-4f96-8c35-4b78b9f5db12"
  ],
  "taskGroupUid": "fdb3af68-d8cc-42d5-94a6-b02ecf88d7a4",
  "groupDescription": "null",
  "distributedCommit": false
}
```

This is it telling us that the group of tasks has been created.

We can take the group ID and check its state.

Once created, the first task will start as soon as the provided scheduledAt date will be reached.


## Using the dashboard

What if we want to use the provided GUI ?
Once started, you can go to the application webpage at **http://localhost:422** .

You will be presented with the dashboard main page.

In this you can see the list of different task groups, and click on each group to inspect the task details for each element of the tree (group) of tasks.

In this first screen we have no task started yet.

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash1.png "Optional title")

If we go on the task creation page (second button on top) we can create a new group of tasks.

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash2.jpg "Optional title")

We can both load a previously saved tree (you can load the file contained in the orchestrator *resources/* folder, named *saved_gui_tree* ) or you can create the tasks one by one (after selecting the types).

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash3.jpg "Optional title")

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash4.jpg "Optional title")

If you click next you can see the whole payload that will be sent to the orchestrater compose endpoint.

Here you can choose if you want distributed commit or not.

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash5.png "Optional title")

If you click next you will land on the dashboard landing page again, and you can inspect the created orchestration details.

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash6.jpg "Optional title")

![img](https://github.com/sitodav/schedulables_usecase/blob/develop/images/dash7.jpg "Optional title")


