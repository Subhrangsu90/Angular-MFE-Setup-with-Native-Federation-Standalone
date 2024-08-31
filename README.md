# **Angular Micro Frontend (MFE) Setup with Native Federation (Standalone)**

This guide outlines the steps to set up an Angular Micro Frontend (MFE) project using native federation with standalone modules.

## **Prerequisites**

Ensure that you have the following installed:

- **Node.js**
- **Angular CLI**

## **Step 1: Create a New Angular Workspace**

Start by creating a new Angular workspace without creating an application initially.

```bash
ng new [project-name] --create-application=false
cd [project-name]
```

## **Step 2: Generate the Applications**

Generate a shell application and two micro frontend applications within the workspace.

```bash
ng g application shell --routing --style=scss
ng g application [application-name1] --routing --style=scss
ng g application [application-name2] --routing --style=scss
```

## **Step 3: Add Native Federation to Each Application**

Install and configure the `@angular-architects/native-federation` package for each application.

```bash
ng add @angular-architects/native-federation --project [application-name1] --port 4201 --type remote
ng add @angular-architects/native-federation --project [application-name2] --port 4202 --type remote
ng add @angular-architects/native-federation --project shell --port 4200 --type dynamic-host
```

## **Step 4: Update `package.json` for Concurrent Builds**

Modify the `package.json` file to include scripts for serving and building the applications.

```json
{
  "name": "mfe-test1",
  "version": "0.0.0",
  "scripts": {
    "ng": "ng",
    "shell": "ng serve shell --configuration development --port 4200 -o",
    "mfe1": "ng serve mfe1 --configuration development --port 4201 -o",
    "mfe2": "ng serve mfe2 --configuration development --port 4202 -o",
    "start": "concurrently \"ng run mfe1:serve\" \"ng run shell:serve\" \"ng run mfe2:serve\"",
    "build": "ng run shell:build && ng run mfe1:build && ng run mfe1:build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test"
  },
  "private": true,
  "dependencies": {
    "@angular-architects/native-federation": "^17.1.6",
    "@angular/animations": "^17.2.1",
    "@angular/common": "^17.2.1",
    "@angular/compiler": "^17.2.1",
    "@angular/core": "^17.2.1",
    "@angular/forms": "^17.2.1",
    "@angular/platform-browser": "^17.2.1",
    "@angular/platform-browser-dynamic": "^17.2.1",
    "@angular/router": "^17.2.1",
    "es-module-shims": "^1.5.12",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.14.2"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^17.2.0",
    "@angular/cli": "~17.2.0",
    "@angular/compiler-cli": "^17.2.1",
    "@types/jasmine": "~4.3.0",
    "concurrently": "^8.2.1",
    "jasmine-core": "~4.6.0",
    "karma": "~6.4.0",
    "karma-chrome-launcher": "~3.2.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.1.0",
    "ng-packagr": "^17.0.2",
    "typescript": "~5.2.2"
  }
}
```

## **Step 5: Configure Routing in the Shell Application**

Update the `routes` in the shell application to lazy-load the components from the micro frontends.

```typescript
import { Routes } from "@angular/router";
import { loadRemoteModule } from "@angular-architects/native-federation";
import { HomeComponent } from "./home/home.component";

export const routes: Routes = [
  {
    path: "",
    component: HomeComponent,
    pathMatch: "full",
  },
  {
    path: "page1",
    loadComponent: () => loadRemoteModule("mfe1", "./Component").then((m) => m.AppComponent),
  },
  {
    path: "page2",
    loadComponent: () => loadRemoteModule("mfe2", "./Component").then((m) => m.AppComponent),
  },
];
```

### **Step 6: Configure the `federation.manifest.json` File**

In the `shell` application, create or update the `federation.manifest.json` file located in `projects/shell/src/assets/`. This file should contain the URLs for the remote entry points of your micro frontends.

```json
{
  "mfe1": "http://localhost:4201/remoteEntry.json",
  "mfe2": "http://localhost:4202/remoteEntry.json"
}
```

### **Explanation:**

- **mfe1**: Points to the remote entry file of the first micro frontend application, which is served at `http://localhost:4201/remoteEntry.json`.
- **mfe2**: Points to the remote entry file of the second micro frontend application, which is served at `http://localhost:4202/remoteEntry.json`.

### **Usage:**

The `federation.manifest.json` file is used by the shell application to dynamically load the micro frontends at runtime. This enables seamless integration of different modules within the shell application.

### **Final Steps:**

After adding this file, ensure that your applications are running and correctly serving their respective remote entries. You can verify this by accessing the URLs provided in the `federation.manifest.json` file from your browser.

## **Step 7: Running the Applications**

You can start the applications individually or concurrently using the following commands:

```bash
# Serve the shell and MFEs concurrently
npm run start

# Serve the shell application only
npm run shell

# Serve mfe1 only
npm run mfe1

# Serve mfe2 only
npm run mfe2
```

## **References**

- [Micro Frontends with Modern Angular](https://www.angulararchitects.io/en/blog/micro-frontends-with-modern-angular-part-1-standalone-and-esbuild/) - Article by Manfred Steyer

- [Code Repository](https://github.com/Subhrangsu90/Angular-MFE-Setup-with-Native-Federation-Standalone.git) - GitHub repository for this setup.

---

This step concludes the setup of your Angular MFE project using native federation. Now, when you navigate to the corresponding routes in your shell application, it should dynamically load the respective micro frontend applications.
