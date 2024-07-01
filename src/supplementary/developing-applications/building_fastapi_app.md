# Building a Python FastAPI application

In this chapter, we will build a simple FastAPI application in Python using the Controller Service Repository pattern, referencing the scenario discussed in the [previous chapter](./controller_service_repository_pattern.md). 

## Reference 

The reference code is available in the [rutu-sh/bootstrapping-with-kubernetes-examples](https://github.com/rutu-sh/bootstrapping-with-kubernetes-examples/tree/main/apps/simple-restapi-server/src/python) repository. 

## Application Structure

Here is how you would structure your code in the Controller Service Repository pattern:

```bash
.
├── build
│   ├── Dockerfile
│   └── requirements.txt
├── docs
│   └── README.md
└── src
|    ├── common
|    │   ├── __init__.py
|    │   ├── common.py
|    │   └── config.py
|    ├── controller
|    │   ├── __init__.py
|    │   ├── health_check_controller.py
|    │   └── user_controller.py
|    ├── models
|    │   ├── __init__.py
|    │   ├── errors.py
|    │   └── models.py
|    ├── repository
|    │   ├── __init__.py
|    │   ├── db_common.py
|    │   └── user_repository.py
|    ├── service
|    │   ├── __init__.py
|    │   └── user_service.py 
|    └── main.py
└── Makefile
```

At the top level, this structure is divided into the following:

1. `build`: Contains build-specific files like Dockerfile and requirements.txt.
2. `docs`: Contains documentation for the application.
3. `src`: Contains the source code for the application.
4. `Makefile`: Contains commands to build, run, and push the application.


## Application Structure

The application contains the following components:

1. `common`: 
    - `common.py`: Contains common functions like logger, etc.
    - `config.py`: Contains configuration values like HOST, PORT, DATABASE_HOST, etc.
2. `models`: 
    - `errors.py`: Contains custom exceptions.
    - `models.py`: Contains the data models used by the different layers.

The above two components are used by the following components:

3. `controller`: Contains the routers and request handlers for processing incoming requests.
    - `health_check_controller.py`: Contains the health check routers. 
    - `user_controller.py`: Contains the user routers.

4. `service`: Contains the business logic for the application.
    - `user_service.py`: Contains the user service logic.

5. `repository`: Contains the database interaction logic.
    - `db_common.py`: Contains common database functions.
    - `user_repository.py`: Contains the user repository logic.

6. `main.py`: Contains the FastAPI application setup and configuration. This is the entry point of the application.


The flow of the application is as follows:

1. The FastAPI application is started in `main.py`. It imports the routers from the `controller` package. Based on the incoming request, the respective router is called.
2. The router calls the appropriate function in the `service` layer. The service layer interacts with the `repository` layer to perform data operations.
3. The `repository` layer interacts with the database to perform CRUD operations. 
4. Repository returns the data to the service layer, which processes it and returns it to the router. The router sends the response back to the client.


This structure separates the concerns of each layer and makes the codebase maintainable. You can use this structure as a reference to build and deploy your own applications.

If you move to a different database, you will only need to change the implementation of the repository layer. Since the service and controller layers are agnostic to the database, you won't need to make any changes there.
If you want to add a new feature, you can add it to the service layer. The controller layer will call the new function, and the repository layer will interact with the database to perform the operation.
If you want to change the request/response format, you can do it in the controller layer. The service and repository layers will remain unaffected.


## Summary

In this chapter, we discussed how to structure your application code in a way that is easy to maintain and scale. We used the Controller Service Repository pattern to build a simple FastAPI application. You can use the application structure as a reference to build and deploy your own applications.

