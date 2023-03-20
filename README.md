# API Todo

The **Todo App API** is a web application developed with Node.js, Typescript, Express and TypeORM, which offers functionalities to manage tasks. The application has authenticated routes, which guarantee the security of user data.

[![Doc](https://img.shields.io/badge/API%20Documentation-https%3A%2F%2Fapi--todo--docs.vercel.app%2F-blueviolet?style=for-the-badge&logo=vercel)](https://api-todo-docs.vercel.app/)

## Table of Contents

- [Table of Contents](#table-of-contents)
- [About](#about)
- [Users](#users)
  - [Create user](#create-user)
  - [Login](#login)
- [Tasks](#tasks)
	- [Create a new task](#create-a-new-task)
	- [List all user tasks](#list-all-user-tasks)
	- [Change the checked of the task by Id](#change-the-checked-of-the-task-by-id)
	- [Delete a task by Id](#delete-a-task-by-id)
- [Author](#author)

## About

Through this API, users can **create**, **update**, **list** and **delete** their tasks, as well as mark tasks as completed. The API uses TypeORM to manage the database, which can be configured with different databases supported by TypeORM.

Application routes are protected by **authentication**, using JWT tokens (JSON Web Token) to ensure that only authenticated users can access them. Authentication is done through a login system, where the user must provide their credentials (nickname and password) to obtain a valid JWT token.

## Users

The main route for those using the application for the first time is to create a user. Creating a user, passing valid data and using a nickname not used by others, you will be able to use the other functionalities of the application involving the tasks.

### Create task

The POST /users route is unique for creating a new user. It is not necessary to send an authorization in the request header. The following properties must be passed in the body of the request:

```json
{
	"username": "string",  // min - 3 | max - 40
	"nickname": "string",  // min - 3 | max - 40
	"password": "string",  // min - 4 | max - 20
}
```

With success, an object will be returned containing a property informing its creation date, the user's unique id and the data passed in the creation, except the password, for security reasons.

#### Expected return: 

##### ✅ Success - Create with valid data

Body:
```json
{
	"username": "John",
	"nickname": "JohnDoe",
	"password": "123456Abc"
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-201%20%7C%20CREATED-brightgreen)
```json
{
	"id": 3,
	"username": "John",
	"nickname": "JohnDoe",
	"createdAt": "2023-03-13T15:30:00",
}
```

----

##### ❌ Error - Nickname already used

Body:
```json
{
	"username": "John",
	"nickname": "JohnDoe",
	"password": "123456Abc"
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-409%20%7C%20CONFLICT-orange)
```json
{
	"message": "Already exists nickname"
}
```

----

##### ❌ Error - Entering invalid and wrong data

Body:
```json
{
	"username": true,
	"nickname": ["anything"],
	"password": "3"
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-400%20%7C%20BAD%20REQUEST-yellow)
```json
{
	"message": {
		"username": [
			"Expected string, received boolean"
		],
		"nickname": [
			"Expected string, received array"
		],
		"password": [
			"String must contain at least 4 character(s)"
		]
	}
}
```

### Login

Route to perform login, where, with success, it returns a valid token that will have authentication for the functionalities of the user's tasks.
To log in, you must send in the body of the request two properties that were used in the creation of the user, these properties are the `nickname` and the `password`. Just like in the example:

```json
{
	"nickname": "string", // min - 3 | max - 40
	"password": "string"  // min - 3 | max - 40
}
```

If the user sends data that does not exist in the database, an error message will be returned stating that the credentials are invalid.

#### Expected return: 

##### ✅ Success - Login with valid data

Body:
```json
{
	"nickname": "JohnDoe",
	"password": "123456Abc"
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-200%20%7C%20OK-brightgreen)
```json
{
	"token": "eyJhbGciOiJ..."
}
```

----

##### ❌ Error - Invalid email or password

Body:
```json
{
	"nickname": "nobody",
	"password": "987654321"
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20UNAUTHORIZED-red)
```json
{
	"message": "Invalid credentials"
}
```

----

##### ❌ Error - No keys required.

Body:
```json
{
	"id": 3,
	"user": "John"
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-400%20%7C%20BAD_REQUEST-yellow)
```json
{
	"message": {
		"nickname": [
			"Required"
		],
		"password": [
			"Required"
		]
	}
}
```

## Tasks

All routes referring to tasks are authenticated, that is, they only work if they send a Bearer Token from the logged in user.

### Create a new task

POST routes are used to send information and data to a server. In this case, the POST route "/tasks" will be used to create tasks for the logged in user, provided he has authorization to access them. To access this route, it is necessary to send an authorization token in the request header.

```sql
POST /tasks
Authorization: Bearer <token>
```

In the body of the request, the following properties must be sent:

```json
{
	"title": "string",  // max - 40
	"checked": false,   // ( opcional ) | default - false
}
```

With success, an object will be returned with the information of the task created, containing the properties "id", "title" and the "checked" of the task, in addition to the properties "createdAt", referring to the task creation date, and "updatedAt " which refers to the task update date.

#### Expected return: 

##### ✅ Success - Create with valid data

Body:
```json
{
	"title": "Read 1 hour a day",
	"checked": false
}
```

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-201%20%7C%20CREATED-brightgreen)
```json
{
	"id": 3,
	"title": "Read 1 hour a day",
	"checked": false,
	"createdAt": "2023-03-17T21:02:20.170Z",
	"updatedAt": "2023-03-17T21:02:20.170Z"
}
```

----

##### ❌ Error - Request without sending an authorization

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20ANAUTHORIZED-red)
```json
{
	"message": "Missing bearer token"
}
```

----

##### ❌ Error - Request without sending token

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20ANAUTHORIZED-red)
```json
{
	"message": "jwt malformed"
}
```

----

##### ❌ Error - Request with invalid token

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20ANAUTHORIZED-red)
```json
{
	"message": "invalid signature"
}
```

### List all user tasks

GET routes are used to fetch information from a server. In this case, the "/tasks" GET route will be used to list all the tasks that were created by the user, as long as he has authorization to access them. To access this route, it is necessary to send an authorization token in the request header.

```sql
GET /tasks
Authorization: Bearer <token>
```

As it is a GET route, a JSON body should not be sent in the request. On success, it should return to the user an array containing all the tasks that were created by the user. If the user has not created any tasks or deleted all tasks, an empty array will be returned. Tasks will come in object format containing the following properties:

```json
{
	"id": 1,    // ( Number )
	"title": "string",    // ( String )
	"checked": false,    // ( Boolean )
	"createdAt": "date",    // ( Date )
	"updatedAt": "date",    // ( Date )
}
```

#### Expected return: 

##### ✅ Success - Returning all tasks

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-200%20%7C%20OK-brightgreen)
```json
[
	{
		"id": 3,
		"title": "Read 1 hour a day",
		"checked": false,
		"createdAt": "2023-03-17T21:02:20.170Z",
		"updatedAt": "2023-03-17T21:02:20.170Z"
	},
	{
		"id": 4,
		"title": "Create todo API",
		"checked": true,
		"createdAt": "2023-03-15T15:00:20.170Z",
		"updatedAt": "2023-03-16T06:25:20.170Z"
	},
	...
]
```

----

##### ❌ Error - Request without sending an authorization

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20ANAUTHORIZED-red)
```json
{
	"message": "Missing bearer token"
}
```

----

##### ❌ Error - Request without sending token

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20ANAUTHORIZED-red)
```json
{
	"message": "jwt malformed"
}
```

----

##### ❌ Error - Request with invalid token

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-401%20%7C%20ANAUTHORIZED-red)
```json
{
	"message": "invalid signature"
}
```

### Change the checked of the task by Id

PATCH routes are used to update information from a server. In this case, the PATCH "/tasks/**:id**" route will be used to update a task, specifically the "checked" property, of course, provided that the logged in user has authorization to update it. To access this route, you need to send an authorization token in the request header.

```sql
PATCH /tasks
Authorization: Bearer <token>
```

In the request route, the id must be inserted after the "/tasks/" endpoint of the task you want to update. It is not necessary to send a body in the request, as the process of updating the task will be automatic just by sending the id in the route.

With success, an object will be returned referring to the task's information with the "checked" property updated.

#### Expected return: 

##### ✅ Success - Updating a valid task

Endpoint: `/tasks/3`

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-200%20%7C%20OK-brightgreen)
```json
{
	"id": 3,
	"title": "Read 1 hour a day",
	"checked": true, // With checked changed
	"createdAt": "2023-03-17T21:02:20.170Z",
	"updatedAt": "2023-03-19T13:01:20.170Z"
}
```

----

##### ❌ Error - Sending a task id from another user

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-403%20%7C%20FORBIDDEN-red)
```json
{
	"message": "User does not have permission for this non-owned task"
}
```

----

##### ❌ Error - Sending an id that doesn't exist

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-404%20%7C%20NOT_FOUND-yellow)
```json
{
	"message": "Task not found"
}
```

### Delete a task by Id

DELETE routes are used to delete or delete information or data from a server. In this case, the DELETE route "/tasks/**:id**" will be used to delete a task, as long as the logged in user has authorization to update it. To access this route, you need to send an authorization token in the request header.

```sql
DELETE /tasks
Authorization: Bearer <token>
```

In the request route, the id must be inserted after the "/tasks/" endpoint of the task you want to delete. It is not necessary to send a body in the request, as the task deletion process will be automatic just by sending the id in the route.

With success, nothing will be returned, only status 204 - no content

#### Expected return: 

##### ✅ Success - Deleting with a valid id

Endpoint: `/tasks/3`

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-204%20%7C%20NO_CONTENT-brightgreen)
```json
// Empty. Means the task has been deleted
```

----

##### ❌ Error - Sending a task id from another user

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-403%20%7C%20FORBIDDEN-red)
```json
{
	"message": "User does not have permission for this non-owned task"
}
```

----

##### ❌ Error - Sending an id that doesn't exist

Response:

![StatusCode](https://img.shields.io/badge/Status%20code%3A%20-404%20%7C%20NOT_FOUND-yellow)
```json
{
	"message": "Task not found"
}
```

## Author

[![Linkedin](https://img.shields.io/badge/linkedin-27f?style=for-the-badge&logo=linkedin&logoColor=)](https://linkedin.com/in/luanflorencioo)
[![Linkedin](https://img.shields.io/badge/github-ddd?style=for-the-badge&logo=github&logoColor=000)](https://github.com/LuanFlorencioo/)

![Luan Avatar](https://avatars.githubusercontent.com/u/71609088?s=120&v=4)

**______Luan Florencio**