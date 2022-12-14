# Using Firebase Callable Cloud Functions with Angular and AngularFire 6

This tutorial will make a simple Angular app that connects to Firebase Cloud Functions.

This project uses Angular 14, AngularFire 6, and Firestore Web version 9 (modular). Why this tutorial doesn't use AngularFire 7 will be explained below.

I assume that you know the basics of Angular (nothing advanced is required). No CSS or other styling is used, to make the code easier to understand.

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
npm install firebase
ng add @angular/fire
```

Deselect `ng deploy -- hosting` and select `Firestore` and `Cloud Functions (callable)`.

It will ask you for your email address associated with your Firebase account. Then it will ask you to associate a Firebase project. Select `[CREATE NEW PROJECT]`. Call it `Firebase Functions Tutorial`.

## Create Firestore database and check Firebase credentials in `environments.ts`
Open your Firebase console and look for your project. If it wasn't made for you then make a new project with a Firestore database.

Open the Firestore [Get started](https://firebase.google.com/docs/firestore/quickstart) section.

In your [Firebase Console](https://console.firebase.google.com), under `Get started by adding Firebase to your app` select the web app `</>` icon. Register your app. You won't need `Firebase Hosting`, we'll just run the app locally.

Under `Add Firebase SDK` select `Use npm`.

If you're using an existing Firestore database go to the settings gear in the Firebase console.

Open `src/environments.ts`. You should see the credentials match the credentials in your Firebase console. If not, copy and paste the credenbtials from your Firebase console to `environments.ts`.

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

Check that your browser is still showing the demo app.

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

## Initialize Firestore
Open the official documentation for (Get started: write, test, and deploy your first functions)[https://firebase.google.com/docs/functions/get-started].

Stay in the project directory and install `firebase-tools`:

```bash
npm install -g firebase-tools
```

Install `firebase-functions` and `firebase-admin`.

```bash
npm install firebase-functions@latest firebase-admin@latest --save
```

Run `firebase login` to log in via the browser and authenticate the firebase tool:

```bash
firebase login
```

Initialize the Firestore database:

```bash
firebase init firestore
```

If you get a warning

```bash
Error: It looks like you haven't used Cloud Firestore in this project before.
```

then go to your Firebase console and click on `Firestore Database`. Open a new database.

## Initialize Functions

Then initialize Firebase Cloud Functions:

```bash
firebase init functions
```

Choose `Initialize`, not `Overwrite`. It will then ask you to name your functions. Just call it `functions`.

It will ask you to choose between `JavaScript` and `TypeScript`.

It will then ask if you want to use ESLint. I choose `no` because your functions won't deploy if ESLint finds anything to complain about, and ESLint can find a lot of unimportant things to complain about. I use Visual Studio Code to find my coding errors.

Then it asks if you want to install npm dependencies. Say `yes`.

### Fix `package.json`

If you chose TypeScript, open `functions/package.json` and change:

```js
"main": "lib/index.js",
```

to

```js
"main": "src/index.ts",
```

If you chose JavaScript skip this step.

## Initialize emulator

Firebase comes with an emulator. The emulator will simulate many Firebase services, including Firestore, Auth, etc. I've never found a need for the emulator with Firestore and Auth as these execute quickly in the cloud. 

Functions are different. Without the emulator developing code can be painfully slow. Deploying your code changes to the cloud takes about two minutes. Then I test my code changes and I have to wait for the console logs. This takes a few more minutes, with clicking various buttons in the console to get the logs to stream. I've seen a lag time between deploying functions to the cloud and the new version running so this can add a minute or two. All in all, waiting five minutes between writing new code and seeing the results feels painfully slow. With the emulator there's no waiting.

Another advantage of the emulator is that you screw up your code, such as writing an infinite loop, without affecting your Google Cloud Services bill. In other words, test your functions in the emulator before deploying them to the cloud.

Initiate the emulators:

```bash
firebase init emulators
npm run build
```

The latter command might ask you to update Java on your computer. 

In `src/environments.ts` add a property:

```ts
export const environment = {
  firebase: {
    projectId: '...,
    appId: '...,
    storageBucket: '...',
    locationId: 'us-central',
    apiKey: '...',
    authDomain: '...',
    messagingSenderId: '...',
  },
  production: false,
  useEmulators: true
};
```

Change `useEmulators` to `false` when you deploy to the cloud.

## Directory structure
Look at your directory and you should see, if you chose TypeScript:

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
           +- index.ts  # main source file for your Cloud Functions code
      |
      +- tsconfig.json  # if you chose TypeScript
      |
      +- package.json  # npm package file describing your Cloud Functions code
```

JavaScript will be simpler:

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
      +- index.js  # main source file for your Cloud Functions code
      |
      +- package.json  # npm package file describing your Cloud Functions code
```

## Setup `@NgModule` for the `AngularFireModule` and `AngularFireFunctionsModule`

Open the [AngularFire documentation](https://github.com/angular/angularfire/blob/master/docs/functions/functions.md) for this section.

Now we can start writing Angular. Open `/src/app/app.module.ts` and import modules.

### AngularFire 6 vs 7

Here we hit a stumbling block. The official AngularFire documentation shows how to use callable functions with AngularFire 6. AngularFire 7 seems to be available for functions but there's no documentation. I asked on Stack Overflow and no one answered my question.

The problem is that you can't mix AngularFire 6 and 7. I use AngularFire 7 for Firestore and Auth. This means that I can't use callable functions with Firestore or Auth. The workaround is to trigger background functions instead of calling functions directly. We'll get to triggered background functions later. First this tutorial will teach callable functions, which you can't really use in a real Angular app now. I believe that we're very close to using AngularFire 7 with callable apps and only a few lines of code will need to change so let's learn to use callable functions with Angular 6.

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { environment } from '../environments/environment';

// AngularFire 7 -- comment out these line, we won't be using AngularFire 7
// import { initializeApp, provideFirebaseApp } from '@angular/fire/app';
// import { provideFirestore, getFirestore } from '@angular/fire/firestore';
// import { provideFunctions, getFunctions, connectFunctionsEmulator } from '@angular/fire/functions';

// AngularFire 6
import { AngularFireModule } from '@angular/fire/compat';
import { AngularFireFunctionsModule } from '@angular/fire/compat/functions';
import { USE_EMULATOR as USE_FUNCTIONS_EMULATOR } from '@angular/fire/compat/functions';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,

    // AngularFire 7 -- comment out these line, we won't be using AngularFire 7
    // provideFirebaseApp(() => initializeApp(environment.firebase)),
    // provideFirestore(() => getFirestore()),
    // provideFunctions(() => getFunctions()),

    // AngularFire 6
    AngularFireModule.initializeApp(environment.firebase),
    AngularFireFunctionsModule
  ],
  providers: [
        { provide: USE_FUNCTIONS_EMULATOR, useValue: environment.useEmulators ? ['localhost', 5001] : undefined }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

You can comment out the Angular 7 lines now. 

## Make the HTML view

Open `app.component.html`. Replace the placeholder view with:

```html
<div>
    <button mat-raised-button color="basic" (click)='callMe()'>Call me!</button>
</div>

{{ data$ | async }}
```

We made a button that calls a handler function in theb controller. There's also a line to display data returned from the callable function.

## Make the component controller

In `app.component.ts` 

```ts
import { Component } from '@angular/core';

// AngularFire 7
// import { getApp } from '@angular/fire/app';
// import { provideFunctions, getFunctions, connectFunctionsEmulator, httpsCallable } from '@angular/fire/functions';
// import { Firestore, doc, getDoc, getDocs, collection, updateDoc } from '@angular/fire/firestore';

// AngularFire 6
import { AngularFireFunctions } from '@angular/fire/compat/functions';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent {
  data$: any;

  constructor(private functions: AngularFireFunctions) {
    const callable = this.functions.httpsCallable('executeOnPageLoad');
    this.data$ = callable({ name: 'Charles Babbage' });
  }

  callMe() {
    console.log("Calling...");
    const callable = this.functions.httpsCallable('callMe');
    this.data$ = callable({ name: 'Ada Lovelace' });
  };
}
```

Again, comment out the Angular 7 imports for now.

In `AppComponent` we make a single variable `data$: any`. This will handle the data returned from the callable functions and display it in the view.

In the `constructor` we make a local variable `functions` to alias `AngularFireFunctions`. This is AngularFire 6. `AngularFireFunctions` isn't on AngularFire 7. As soon as `AngularFireFunctions` (or an equivelant property) is available on AngularFire 7 we should be able to use AngularFire 7.

We have two lines of code that call a cloud function to execute on page load. These use the property `httpsCallable`. This take one parameter, the name of your cloud function.

To execute the callable function we use `this.data$` to handle the returned data and then call the function. Calling the function has a required parameter, which is an object holding the data you want to send to the cloud function.

Lastly, the component controller has a handler function for the button in the view. This executes similar code.

## Write your Firebase Cloud Functions

Open `functions/src/index.ts` or `functions/index.js`. Import two Firebase modules, initialize your app, and then write your callable functions.

```ts
// The Cloud Functions for Firebase SDK to create Cloud Functions and set up triggers.
const functions = require('firebase-functions');

// The Firebase Admin SDK to access Firestore.
const admin = require('firebase-admin');
admin.initializeApp();

// executes on page load
exports.executeOnPageLoad = functions.https.onCall((data, context) => {
    console.log("The page is loaded!")
    console.log(data);
    console.log(data.name);
    // console.log(context);
    return 22
});

// executes on user input
exports.callMe = functions.https.onCall((data, context) => {
    console.log("Thanks for calling!")
    console.log(data);
    console.log(data.name);
    // console.log(context);
    return 57
});
```

The function `executeOnPageLoad` executes when `ng serve` starts or restarts.

The functions `callMe` executes when you click the button in the view.

Each function is in the form 

```js
`functions.https.onCall((data, context) => {

});
```

`https.onCall` means that this functions can be called directly from Angular. `data` is the data sent from Angular. `context` is metadata about the function's execution. We won't be using this.

Each function sends a message to the console, then sends the data from Angular to the console. We've commented out displaying the `context` metadata in the console. This metadata goes on for pages and makes the logs hard to read. 

Finally, each callable function returns something. 

## Run emulator

Start the Firebase Emulator.

```bash
firebase emulators:start --only functions
```

Run the function `executeonPageLoad` by restarting `ng serve`. Run the function `callMe` by clicking the button in the view.

You should see the results in several places. In the view, you should see `22` as the result from `executeonPageLoad`. When you click the button this result changes to `57`. This is an observable so if the data changes in the cloud function it'll change in the view.

In the emulator logs, you should see the logs:

```
12:05:02  I function[us-central1-callMe]  Beginning execution of "callMe"
12:05:02  I function[us-central1-callMe]  Thanks for calling!
12:05:02  I function[us-central1-callMe]  { name: 'Ada Lovelace' }
12:05:02  I function[us-central1-callMe]  Ada Lovelace
12:05:02  I function[us-central1-callMe]  Finished "callMe" in 6.512202ms
```

You should see the same logs in the terminal emulator tab.

## TypeScript errors

If you chose to use TypeScript you may see some errors. The issue is the data type of the parameters in your functions:

```ts
exports.callMe = functions.https.onCall((data, context) => {}
```

This will throw any error `Parameter 'data' implicitly has an 'any' type.` This is because TypeScript is running in `strict` mode by default.

```ts
exports.callMe = functions.https.onCall((data: any, context: any) => {}
```

This will throw this error:

```
SyntaxError: Unexpected token ':'
```

The latter appears to be an issue with the transpiler. The transpiler apparently failed to remove the types when it transpiled TypeScript into JavaScript.

Both errors can be ignored. You can get rid of the former error by opening `functions/tsconfig.json` and either change `strict` to `false` or to add this line:

```js
"noImplicitAny": false,
```

It doesn't seem to matter if you set this to `true` or `false`.

## Deploy to Firebase


```bash
 firebase deploy --only functions
 ```
 
 In `src/environments.ts` change `useEmulators` to `false`:
 
 ```js
 useEmulators: false
 ```
 
Check the logs in your Firebase Console to see if your functions run.
 
## Calling functions via HTTP requests

Firebase cloud functions can also be [called via HTTP requests](https://firebase.google.com/docs/functions/http-events?hl=en&authuser=0). This is useful for Express apps but not for Angular apps.

## Triggering Firebase Cloud Functions from FireStore

You can trigger a Firebase Cloud Function by writing data to Firestore. This doesn't use AngularFire for functions, i.e., only uses AngularFire for Firestore, so it doesn't matter whether you use AngularFire 6 or 7 for functions.

Make a new cloud function:

```ts
exports.makeUppercase = functions.firestore.document('/triggers/upperCASE')
.onCreate((snap, context) => {
  const original = snap.data().original;
  console.log('Uppercasing', context.params.documentId, original);
  const uppercase = original.toUpperCase();
  return snap.ref.set({uppercase}, {merge: true});
});
```

This will trigger you write data to the document `upperCASE` in the collection `triggers`. The function receives a string and returns the string in UPPERCASE.

Add a form field to the HTML view:

```html
<div>
    <button mat-raised-button color="basic" (click)='callMe()'>Call me!</button>
</div>

{{ data$ | async }}

<form (ngSubmit)="upperCaseMe()">
    <input type="text" [(ngModel)]="message" name="message" placeholder="Message" required>
    <button type="submit" value="Submit">Submit</button>
</form>
```

In `app.module.ts` import Angular Forms:

```js
import { FormsModule } from '@angular/forms';
...
  imports: [
    BrowserModule,
    FormsModule,
    ...
    ]
```

Add the handler function to `app.component.ts`

```ts
  async upperCaseMe() {
    console.log(this.message);
      try {
        const docRef = await addDoc(collection(this.firestore, 'triggers'), {
          message: this.message,
        });
        console.log(docRef);
        this.message = null;
      } catch (error) {
        console.error(error);
      }
  }
```

Deploy to Firebase:

```bash
firebase deploy --only functions
```

### Switch from AngularFire 6 to 7

Functions and the emulator run in AngularFire 6. Firestore runs in AngularFire 7. You can't mix AngularFire 6 and 7. Comment out the AngularFire 6 code and comment in the AngularFire 7 code. You don't AngularFire Functions to trigger cloud functions (only for callable functions). 

