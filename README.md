# Tasks with auth kubernetes demo

Demo application to manage tasks with users registration and a mock auth.

## **Requirements**
- Docker installed
- Kubernetes (kubectl) installed
- Minikube installed
- Postman

## **Setup**

1. Start a cluster with minikube
    ```console
    minikube start
    ```

2. Build the images
    ```console
    docker build -t kub-networking/auth-api auth-api/
    ```
    ```console
    docker build -t kub-networking/tasks-api tasks-api/
    ```
    ```console
    docker build -t kub-networking/users-api users-api/
    ```
    ```console
    docker build -t kub-networking/frontend frontend/
    ```

3. Load images in minikube
     ```console
    minikube image load kub-networking/auth-api
    ```
     ```console
    minikube image load kub-networking/tasks-api
    ```
     ```console
    minikube image load kub-networking/users-api
    ```
     ```console
    minikube image load kub-networking/frontend
    ```


## **Run it**

- ### **Backend**

    1. Apply the APIs manifest files

        ```console
        kubectl apply -f kubernetes/tasks-pv.yaml -f kubernetes/tasks-pvc.yaml -f kubernetes/auth-deployment.yaml -f kubernetes/auth-service.yaml -f kubernetes/tasks-deployment.yaml -f kubernetes/tasks-service.yaml -f kubernetes/users-deployment.yaml -f kubernetes/users-service.yaml
        ```

    2. Open a new terminal and expose the users service

        ```console
        minikube service users-service
        ```

        You will get an output like the following where you can get the `URL` of the service (`USERS_URL`)

        ![expose-users](assets/expose-users.png)

        :warning: By default this command attempts to open the URL on your web browser, but you can close it.

    3. Open a new terminal and expose the tasks service

        ```console
        minikube service tasks-service
        ```

        You will get an output like the following where you can get the `URL` of the service (`TASKS_URL`)

        ![expose-users](assets/expose-tasks.png)

        :warning: By default this command attempts to open the URL on your web browser, but you can close it.

- ### **Frontend**

    1. Apply the frontend manifest files

        ```console
        kubectl apply -f kubernetes/frontend-deployment.yaml -f kubernetes/frontend-service.yaml
        ```

    2. Open a new terminal and expose the frontend service

        ```console
        minikube service frontend-service
        ```

        You will get an output like the following where you can get the `URL` of the service (`FRONTEND_URL`)

        ![expose-frontend](assets/expose-frontend.png)

        :warning: By default this command attempts to open the URL on your web browser. If the browser it is not automatically open you can manually opent it an use the URL provided on the console output.

        ![frontend-open](assets/frontend-open.png)

## **Frontend Usage**

1. Open the frontend in a browser using the `FRONTEND_URL` provided when you exposed the frontend service

    ![frontend-open](assets/frontend-open.png)

    Here you will found the following components:

    - **Title text box:** form text box to provide the task title.
    - **Description text box:** form text box to describe the task.
    - **Add Task button:** action button to save the a new tasks with the form data.
    - **Fetch Tasks button:** action button to list the created tasks.


2. Fill the form with the data for a new task and click on **Add Task**.

    ![frontend-fill-form](assets/frontend-fill-form.png)

    The form will be cleanned and you need to go to next step to see the new task.

3. Click on **Fetch Tasks** to list the created tasks

    ![frontend-fetch](assets/frontend-fetch.png)
## **APIs Usage**

Once you expose the users-service you should get the `URL` of the service and the following endpoints will be availables:

- ### Signup

    Create a new user

    - Endpoint: 
        ```http
        POST ${USERS_URL}/signup/
        ```

    - Request Params:

        | Parameter | Type     | Description               |
        | --------- | -------- | ------------------------- |
        | `email`   | `string` | **Required** User email   |
        | `password`| `string` | **Required** User password|

    - Body example:

        ```JSON
        {
            "email": "test@test.com",
            "password": "test01"
        }
        ```
        ![signup-request](assets/signup-request.png)

    - Response:

        ```JSON
        {
            "message": "User created!"
        }
        ```
        ![signup-response](assets/signup-response.png)

- ### Login
    Login using user credentials

    - Endpoint: 
        ```http
        POST ${USERS_URL}/login/
        ```

    - Request Params:

        | Parameter | Type     | Description               |
        | --------- | -------- | ------------------------- |
        | `email`   | `string` | **Required** User email   |
        | `password`| `string` | **Required** User password|

    - Body example:

        ```JSON
        {
            "email": "test@test.com",
            "password": "test01"
        }
        ```
        ![login-request](assets/login-request.png)

    - Response:

        The API will send you the mocked token

        ```JSON
        {
            "token": "abc"
        }
        ```
        ![login-response](assets/login-response.png)

- ### Add Task

    Create a task

    - Endpoint: 
        ```http
        POST ${TASKS_URL}/tasks/
        ```

    - Headers:

        | Key               | Value                |
        | ----------------- | -------------------- |
        | **Authorization** | Bearer ${USER_TOKEN} |

        ![tasks-create-request-header](assets/tasks-create-request-header.png)

    - Request Params:

        | Parameter | Type     | Description                   |
        | --------- | -------- | -------------------------     |
        | `title`   | `string` | **Required** task title       |
        | `text`    | `string` | **Required** task description |

    - Body example:

        ```JSON
        {
            "title": "Test task", 
            "text": "Do this first"
        }
        ```
        ![tasks-create-request-body](assets/tasks-create-request-body.png)

    - Response:

        ```JSON
        {
            "message": "Task stored.",
            "createdTask": {
                "title": "Test task",
                "text": "Do this first"
            }    
        }
        ```
        ![tasks-create-response](assets/tasks-create-response.png)

- ### List Tasks

    List all the tasks

    - Endpoint: 
        ```http
        GET ${TASKS_URL}/tasks/
        ```

    - Headers:

        | Key               | Value                |
        | ----------------- | -------------------- |
        | **Authorization** | Bearer ${USER_TOKEN} |

        ![tasks-get-request-header](assets/tasks-get-request-header.png)

    - Response:

        The application will return the list of the created tasks

        ```JSON
        {
            "message": "Tasks loaded.",
            "tasks": [
                {
                    "title": "Test task",
                    "text": "Do this first"
                }
            ]
        }
        ```
        ![tasks-get-response-ok](assets/tasks-get-response-ok.png)

        :warning: If no task has been created, you will receive an error

        ```JSON
        {
            "message": "Loading the tasks failed."
        }
        ```
        ![tasks-get-response-failed](assets/tasks-get-response-failed.png)

## :stop_sign: Stop instructions

You can stop the cluster without losing the deployment
```console
minikube stop
```
Or you can delete the deployment, then stop the cluster.

Delete the backend configuration
```console
kubectl delete -f kubernetes/tasks-pv.yaml -f kubernetes/tasks-pvc.yaml -f kubernetes/auth-deployment.yaml -f kubernetes/auth-service.yaml -f kubernetes/tasks-deployment.yaml -f kubernetes/tasks-service.yaml -f kubernetes/users-deployment.yaml -f kubernetes/users-service.yaml
```

Delete the frontend configuration
```console
kubectl delete -f kubernetes/frontend-deployment.yaml -f kubernetes/frontend-service.yaml 
```
Stop the cluster

```console
minikube stop
```
