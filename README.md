# Tasks with auth kubernetes demo

Demo application to manage tasks with users registration and a mock auth.

## Requirements
- Docker installed
- Kubernetes (kubectl) installed
- Minikube installed
- Postman

## Setup

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


## Run it

1. Apply the users api manifest files
    
    ```console
    kubectl apply -f kubernetes/users-deployment.yaml -f kubernetes/users-service.yaml
    ```

2. Open a new terminal and expose the users service
    
    ```console
    minikube service users-service
    ```

    You will get an output like the following where you can get the `URL`of teh service

    ![expose-users](assets/expose-users.png)

    :warning: By default this command attempts to open the URL on your web browser, but you can close it.

## Usage

Once you expose the users-service you should get the `URL` of the service and the following endpoints will be availables:

- **Signup**

    - Endpoint: `${URL}/signup`

    - Method type: `POST`

    - Body Format: `raw` | `JSON`

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

- **Login**

    - Endpoint: `${URL}/login`

    - Method type: `POST`

    - Body Format: `raw` | `JSON`

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
