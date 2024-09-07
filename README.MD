# The Angular Microblog

In this lab, you are going to extend an existing Angular microblog application to add authentication and authorization. You will also learn how to use Angular's reactive forms and routing. Furthermore, you will learn how to connect the Angular application
to the backend you built in the previous lab.

Since it might be the first time you are working with Angular, I heavily recommend you to use Github Copilot to help you write code. It is a great tool that can help you write code faster and more efficiently. You can find more information about it [here](https://copilot.github.com/).

### Submission

Similarly to the previous lab, you should push your work to Github.

## Project Setup

At the root of the repo you will notice two folders.

- `backend`: The backend app code (Express)
- `frontend`: The frontend app code (Angular)

We recommend having a similar setup for your project, where your frontend, backend, and any other parts of your project live in the same repo.

In this case, the `static` folder in `backend` can be removed. We will no longer need our
vanilla javascript frontend.

## Backend Organization

Similar the previous labs the backend has been organized in the following format

- `middleware/`: Any custom Express middleware is defined here
- `models/`: model files for user and messages
- `routers/`: we stored all endpoints in app.js before, now we separate out the CRUD for users and the CRUD for messages
- `static/`: this can be removed, since our frontend is served by Angular
- `app.js`: the entrypoint to the application becomes much cleaner!

## Starting the backend

1. `cd backend`
2. `npm install`
3. Start the backend with `npm run start:dev` or `nodemon app.js`

## Starting the frontend

1. open a new terminal window (Keep backend terminal running)
2. `cd frontend`
3. `npm install`
4. Start angular dev server with `ng serve`

## CORS problem

Now you may notice that all the HTTP requests are failing due to CORS. This is because the backend (`localhost:3000`) and the frontend dev server (`localhost:4200`) are considered as a different origin from the browser.

We can fix this by adding the relevant CORS headers to every API response. We can use a middleware to do this.

1. Install the [CORS](https://expressjs.com/en/resources/middleware/cors.html) middleware package with `npm install --save cors`
2. add the following in `app.js`

```javscript

import cors from "cors";

...

const corsOptions = {
  origin: "change me",
  credentials: true
};
app.use(cors(corsOptions));
```

To test your understanding:

1. what should `"change me"` be?
2. what does `credentials: true` do? (Hint: look at the documentation)

## Creating Forms with ReactiveFormsModule

In the lecture, we have looked at working with forms using two way binding (i.e. using `[(ngModel)]`). This is a convenient way to work with forms, but it has some limitations. For example, it is not possible to validate the form input. In this section, we will look at how to use Angular's reactive forms module to create forms.

[Reference documentation](https://angular.io/guide/reactive-forms#validating-form-input)

1. Go to `new-message.component.ts`
2. create a field in `messageForm` using `this.fb.group` for message. Make this field required and set the initial value to `''`

Syntax (see documentation site for more detail)

```javascript
fieldname: [initial value, [validators]],
```

3. Next in the html file add `[formGroup]="messageForm"` to the `<form>` element. This will associate the form with the FormGroup `messageForm` that we defined.
4. Next add `formControlName="fieldname"`
   to the appropriate input/textarea. The `fieldname` is the messageForm field that you want to bind the element to.

"Two-way binding" between an element in the html and a field in the component class allows you to change data in one place and have it automatically reflect in the other

5. Now you should be able to see the two way binding in action, with the Message and FormStatus updating as you type.

6. Now go to `/src/app/pages/sign-in` and complete the functionality with reactive forms.

Hint: you probably might need 2 form groups.

## API Service

1. in the `services` folder you will find `api.service.ts`. Similar to what you've learned in lecture, the api service is responsible for interacting with or backend api and is used by any frontend component that want to communicate with our backend.

2. Implement the remaining functions in the api service using `HttpClient`. Feel free to change the function headers if you need to.

[Documentation](https://angular.io/api/common/http/HttpClient#get)

`this.endpoint` is an env var and it is currently set to `localhost:3000`

Some endpoints are already implemented, you can use that as a reference.

## Using API service

1. Angular is famous for its dependency injection framework. If we want to use ApiService, we must declare it in the constructor. This applies for all the non components you want to use in another component, and mostly, it would be custom services you define.

2. in `app/pages/index/index.component.ts` adding following `private api: ApiService` to the constructor's parameters allow the component to use `ApiService`.

3. Now Angular now knows to inject `ApiService` into the index component. We can use api service by doing `this.api`. Implement the remaining functions in the index component. Note that Angular uses observables instead of promises. `postMessage` is already implemented, you can use that as reference.

4. the `sign-in` component also needs to use api service. Implement the remaining functions for that component as well.

Hint: if you want to access the values in a `FormGroup` do `theformgroupname.value`

## Routing

In the section, we have seen how to use Angular's `routerLink` to navigate between pages. In this section, we will look at how to use Angular's router to programmatically redirect to different pages.

Angular has client-side routing out of the box. This allows your app to appear to have to have multiple "pages" even though
its an SPA. If you look at `app-routing.module.ts`, we've it so that defined `/sign-in` to render the `sign-in` component
while any other url will render the `index` component.

We can also redirect to pages programmatically in Angular. In the `sign-in` component we want to redirect to the index
component after signin/signout are successful. To do this we will use the Angular router.

In order to access the router using typescript, add the import `import { Router } from '@angular/router';` to the componen
and define a router in the constructor so that Angular can provide it via Dependency injection (similar to api service).

`this.router.navigate(['/']);` will navigate back to our `index` component. Replace `router` with whatever name you used to define router in the constructor.

Now that you know how to use `ApiService` and `Router`, go to the `header.component.ts` and implement a signout function. This should call `signOut` in api service and if it succeeds redirect to `/sign-in`.

## Conditional Rendering

For the Post a message form, we only want to display it if the user is authenticated. If you signout and go directly to
`http://localhost:4200/` then you will notice that the form still shows even after we signed out. To conditionally render
than form we can use `*ngIf="some boolean variable"` in the `.component.html` file.

In `index.component.html` use `*ngIf` on the appropriate component and use `isAuth` as the boolean variable or your own
boolean expression. If done correctly the form should only render if that boolean expression is `true`

## (Please read) Note on CORS, Cookies and Express Session

1. When you are working on your project, there is a good chance that you will have a similar setup to this lab. Ie frontend dev server and backend server running on different origins. If you have such a setup then you will encounter CORS issues

2. To ensure that cookies and express session work for CORS requests, in this lab I added `withCredentials: true` in `api.interceptor.ts` on the frontend and on the backend `credentials: true` in the cors setup. This may vary depending on the library/framework, but should be similar.

[withCredentials](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials)

[backend credentials headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)

Another important thing, is that this setup would only apply for your development environment and production setup will be different.

3. This setup to handle CORS can be avoided with a proxy server. Essentially all traffic goes through the proxy server and the proxy server will forward the request to the backend etc. This avoids CORS issues as all requests go through the same origin. You will learn this more in detail later on in the course.

## Bonus: Deploy the frontend on Github Pages

Although in your project you should not use Github Pages to host your frontend, it is
nice to try it for the lab. Deploy your Angular application to Github Pages.

## Grading

1. (1pt) CORS Problem
2. (2pt) Reactive Forms
3. (1pt) API service
4. (3pt) Using API service
5. (2pt) Routing
6. (1pt) Conditional Rendering
