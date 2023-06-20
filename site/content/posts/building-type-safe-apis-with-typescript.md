---
title: "Building Type-Safe APIs with TypeScript"
date: 2023-06-14T12:03:00.908Z
tags: ["typescript","api development","type-safety"]
---


As static type-checking gains popularity as a way to catch programming errors early and more efficiently, TypeScript has been gaining traction within the developer community. With TypeScript, it is now feasible to specify types for your APIs, which can prevent runtime errors during API consumption, improve documentation, and allow for the safe refactoring of application code. In this post, we will discuss how to build a type-safe API using TypeScript.

## Setting up the project

To get started, create a new TypeScript project using the `tsc` command:

```bash
mkdir ts-api
cd ts-api
npm init -y
npm install typescript
npx tsc --init
```

This will set up the TypeScript project with the default settings. 

Next, create a new `index.ts` file in the `src` directory and a `dist` directory for the compiled output:

```bash
mkdir src
touch src/index.ts
mkdir dist
```

For the purpose of this post, we will be using the Express framework to set up the API. We will also use the `body-parser` and `cors` middleware. Install them by running the following command:

```bash
npm install express body-parser cors
```

With that set up, we can start building our API.

## Type-Safe Request and Response Objects

When building APIs, having strongly-typed request and response objects can prevent errors on both the server and client side. TypeScript provides the `interface` construct to define contracts between different parts of a system. We can use interfaces to define the shape of the request and response objects.

```typescript
interface User {
  name: string;
  email: string;
  password: string;
}

interface UserRequest {
  body: User;
}

interface UserResponse {
  id: number;
  createdAt: Date;
}
```

Here, we define a `User` object that the `UserRequest` interface will expect to receive in the API's body. The `UserResponse` interface is the response that will be sent back with a user's `id` and `createdAt` timestamp. With these interfaces defined, we can now use them with the Express framework.

```typescript
import express, { Request, Response } from 'express';
import bodyParser from 'body-parser';
import cors from 'cors';

const app = express();
app.use(bodyParser.json());
app.use(cors());

app.post('/users', (req: UserRequest, res: Response<UserResponse>) => {
  const { name, email, password } = req.body;

  // ... Create the user here and save to the database ...

  res.status(201).json({
    id: 1, // replace with actual user ID generated by the database
    createdAt: new Date(),
  });
});

app.listen(3000, () => {
  console.log('App is running on port 3000');
});
```

Here, we provide the types for the `req` and `res` objects. By doing so, TypeScript is able to catch errors when the wrong type of object is provided or returned from the API. 

## Error Handling

Effective error handling is crucial in any API development project. Unhandled runtime errors can result in unexpected behavior and cause serious issues in production environments. To avoid these issues, we can use TypeScript to define and catch errors before runtime. 

```typescript
class HttpException extends Error {
  status: number;

  constructor(message: string, status: number) {
    super(message);
    this.status = status;
  }
}

class UserNotFoundException extends HttpException {
  constructor(id: number) {
    super(`User with ID ${id} not found`, 404);
  }
}
```

Here, we define two classes for HTTP exceptions: The `HttpException` class provides a `status` property that every child class must define, while the `UserNotFoundException` class extends `HttpException` for the specific case when a user could not be found, returning a `404` status code. We can now use these exceptions in our Express route handlers.

```typescript
app.get('/users/:id', (req: Request, res: Response<string>) => {
  const userId = parseInt(req.params.id, 10);

  // we can use `if (!userId)` to check if userId is undefined or null

  if (isNaN(userId)) {
    throw new HttpException('Invalid user ID', 400);
  }

  const user = /* ... find user from the database by ID ... */

  if (!user) {
    throw new UserNotFoundException(userId);
  }

  res.json(user);
});
```

By using these custom exceptions, we can reduce the number of runtime errors and ensure that the API client is receiving detailed error messages that provide valuable feedback in case of a failure.

## Conclusion

In this post, we have explored how to leverage TypeScript to create type-safe request and response objects, customize error handling, and prevent common programming errors. Using TypeScript in API development can make your code base more maintainable and readable, catch errors in development, and ensure a more robust application.