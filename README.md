# Using Firebase Cloud Functions with Angular

This tutorial will make a simple Angular app that connects to Firebase Cloud Functions.

This project uses Angular 14, AngularFire 7.4, and Firestore Web version 9 (modular). 

I assume that you know the basics of Angular (nothing advanced is required). No CSS or styling is used, to make the code easier to understand.

## Create a new project

In your terminal:

```bash
npm install -g @angular/cli
ng new FirebaseFunctionsTutorial
```

The Angular CLI's `new` command will set up the latest Angular build in a new project structure. Accept the defaults (no routing, CSS). Start the server:

```bash
cd FirebaseFunctionsTutorial
ng serve -o
```

Your browser should open to `localhost:4200`. You should see the Angular default homepage.

## Install AngularFire and Firebase

Open another tab in your terminal and install AngularFire and Firebase from `npm`.

```bash
ng add @angular/fire
```

Deselect `ng deploy -- hosting` and select `Firestore` and `Cloud Functions (callable)`.

It will ask you for your email address associated with your Firebase account. Then it will ask you to associate a Firebase project. Select `[CREATE NEW PROJECT]`. Call it `Firebase Functions Tutorial`.

If this doesn't work, open your Firebase console and make a new project. Call it `Greatest Computer Scientists`. Skip the Google Analytics.

Create your Firestore database.

## Add Firebase config to `environments` variable

Open the Firestore [Get started](https://firebase.google.com/docs/firestore/quickstart) section.

In your [Firebase Console](https://console.firebase.google.com), under `Get started by adding Firebase to your app` select the web app `</>` icon. Register your app, again calling it `Greatest Computer Scientists`. You won't need `Firebase Hosting`, we'll just run the app locally.

Under `Add Firebase SDK` select `Use npm`.

Return to your terminal and install Firebase in your new project.

```bash
npm install firebase
```

Then copy and paste the config values provided to your `environment.ts` file:

```ts
export const environment = {
  production: false,
  firebaseConfig: {
    apiKey: '<your-key>',
    authDomain: '<your-project-authdomain>',
    projectId: '<your-project-id>',
    storageBucket: '<your-storage-bucket>',
    messagingSenderId: '<your-messaging-sender-id>',
    appId: '<your-app-id>',
    measurementId: '<your-measurement-id>'
  }
};
```

Don't copy and paste the whole window that the Firebase console provides. Check that it looks like the above code. A `=` needs to be changed to `:` and a `;` needs to be dropped. Then check that your browser is still showing the demo app.

`Add Firebase SDK` also tells you to do several other things:

```
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";

// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Initialize Firebase
const app = initializeApp(firebaseConfig);
```

We'll use AngularFire instead of these items. Click `Continue to console`.

## Set up Functions
Open the official documentation for (Get started: write, test, and deploy your first functions)[https://firebase.google.com/docs/functions/get-started].

Stay in the project directory and install `firebase-tools`:

```bash
npm install -g firebase-tools
```

Install `firebase-functions` and `firebase-admin`.

```bash
npm install firebase-functions@latest firebase-admin@latest --save
```

I'm going to name my first-born child `Admin`.

Run `firebase login` to log in via the browser and authenticate the firebase tool:

```bash
firebase login
```

Initialize the Firestore database:

```bash
firebase init firestore
```

You'll get a warning:

```bash
Error: It looks like you haven't used Cloud Firestore in this project before.
```

Go to your Firebase console and click on `Firestore Database`. Open a new database.

Then initialize Firebase Cloud Functions:

```bash
firebase init functions
```

Choose `Initialize`, not `Overwrite`. It will then ask you to name your functions. Just call it `functions`.

It will ask you to choose between `JavaScript` and `TypeScript`. This tutorial will use TypeScript.

It will then ask if you want to use ESLint. I choose `no` because your functions won't deploy if ESLint finds anything to complain about, and ESLint can find a lot of unimportant things to complain about. I use Visual Studio Code to find my coding errors.

Then it asks if you want to install npm dependencies. Say `yes`.

### Fix `package.json`

Open `functions/package.json` and change:

```js
"main": "lib/index.js",
```

to

```js
"main": "src/index.ts",
```

### Directory structure
Look at your directory and you should see:

```bash
myproject
 +- .firebaserc    # Hidden file that helps you quickly switch between
 |                 # projects with `firebase use`
 |
 +- firebase.json  # Describes properties for your project
 |
 +- functions/     # Directory containing all your functions code
      |
      +- node_modules/ # directory where your dependencies (declared in # package.json) are installed
      |
      +- package-lock.json
      |
      +- src/
          |
           +- index.js  # main source file for your Cloud Functions code
      |
      +- tsconfig.json  # if you chose TypeScript
      |
      +- package.json  # npm package file describing your Cloud Functions code
```

## Setup `@NgModule` for the `AngularFireModule` and `AngularFireFunctionsModule`

Open the [AngularFire documentation](https://github.com/angular/angularfire/blob/master/docs/functions/functions.md) for this section.

Open `/src/app/app.module.ts` and import modules. This is set up for using the Firebase Cloud Function emulator, not for calling a Firebase Cloud Function in the cloud. Don't deploy a Firebase Cloud Function to the cloud until you've tested it in the emulator.

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { AngularFireModule } from '@angular/fire/compat';
import { AngularFireFunctionsModule, USE_EMULATOR } from '@angular/fire/compat/functions';
import { environment } from '../environments/environment';

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(environment.firebase),
    AngularFireFunctionsModule
  ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ],
  providers: [
    { provide: USE_EMULATOR, useValue: ['localhost', 5001] }
   ]
})
export class AppModule {}
```

## Inject `AngularFireFunctions` into Component Controller

Open `/src/app/app.component.ts` and import one AngularFire module.

What's going on with the template?

I have no idea what the code in the constructor does.

```ts
import { Component } from '@angular/core';
import { AngularFireFunctions } from '@angular/fire/compat/functions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  constructor(private fns: AngularFireFunctions) {}
 
  callMe() {
    console.log("Calling...");
    this.fns.httpsCallable('helloWorld');
  }
}
```

## Make the HTML view

Now we'll make the view in `app.component.html`. Replace the placeholder view with:

```html
<div>
    <button mat-raised-button color="basic" (click)='callMe()'>Call me!</button>
</div>
```

We made a button that calls a handler function in theb controller.

## Write your first Firebase Cloud Function

Open `functions/src/index.ts`. Import two Firebase modules and uncomment the default function.

```ts
// The Cloud Functions for Firebase SDK to create Cloud Functions and set up triggers.
const functions = require('firebase-functions');

// The Firebase Admin SDK to access Firestore.
const admin = require('firebase-admin');
admin.initializeApp();

export const helloWorld = functions.https.onRequest((request, response) => {
  functions.logger.info("Hello logs!", {structuredData: true});
  response.send("Hello from Firebase!");
});
```

## Run emulator

Install
