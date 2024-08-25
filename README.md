# -Angular-application-with-Okta-for-authentication
Integrating an Angular application with Okta for authentication involves setting up Okta as an identity provider, configuring your Angular app to use Okta, and implementing the necessary authentication logic. Below is a step-by-step guide to help you achieve this.

### Step 1: Create an Okta Developer Account

1. **Sign Up for Okta:**
   - Visit [Okta Developer](https://developer.okta.com/) and sign up for a free account.
   - Once you sign up, log in to your Okta dashboard.

2. **Create a New Application in Okta:**
   - In your Okta dashboard, navigate to **Applications** > **Applications**.
   - Click **Create App Integration**.
   - Choose **Single-Page App (SPA)** as the platform.
   - Click **Next**.

3. **Configure the App:**
   - **App Integration Name:** Give your app a name (e.g., "Angular Okta App").
   - **Sign-in redirect URIs:** Add `http://localhost:4200/callback`.
   - **Sign-out redirect URIs:** Add `http://localhost:4200`.
   - Click **Save**.

4. **Client ID and Issuer:**
   - Once the app is created, you'll see a screen with the **Client ID** and **Issuer**. Take note of these values, as you'll need them in your Angular app.

### Step 2: Set Up the Angular Application

1. **Create a New Angular App:**
   - If you don't already have an Angular project, create one using Angular CLI:
     ```bash
     ng new angular-okta-app
     cd angular-okta-app
     ```

2. **Install Okta Dependencies:**
   - Install the required Okta SDKs and Angular authentication modules:
     ```bash
     npm install @okta/okta-angular @okta/okta-auth-js
     ```

3. **Update `app.module.ts`:**
   - Import the `OktaAuthModule` and configure it with your Okta settings:
     ```typescript
     // src/app/app.module.ts
     import { NgModule } from '@angular/core';
     import { BrowserModule } from '@angular/platform-browser';
     import { AppRoutingModule } from './app-routing.module';
     import { AppComponent } from './app.component';
     import { OktaAuthModule, OKTA_CONFIG } from '@okta/okta-angular';
     import { OktaAuth } from '@okta/okta-auth-js';

     const oktaAuth = new OktaAuth({
       clientId: 'YOUR_CLIENT_ID',
       issuer: 'https://YOUR_OKTA_DOMAIN/oauth2/default',
       redirectUri: window.location.origin + '/callback',
     });

     @NgModule({
       declarations: [AppComponent],
       imports: [
         BrowserModule,
         AppRoutingModule,
         OktaAuthModule,
       ],
       providers: [{ provide: OKTA_CONFIG, useValue: { oktaAuth } }],
       bootstrap: [AppComponent],
     })
     export class AppModule {}
     ```

   Replace `'YOUR_CLIENT_ID'` and `'https://YOUR_OKTA_DOMAIN/oauth2/default'` with the values from your Okta application settings.

### Step 3: Implement Authentication Logic

1. **Add Routes and Guards:**
   - Create a callback component to handle the Okta redirect:
     ```bash
     ng generate component callback
     ```

   - In `app-routing.module.ts`, set up routes including the callback and a protected route:
     ```typescript
     // src/app/app-routing.module.ts
     import { NgModule } from '@angular/core';
     import { RouterModule, Routes } from '@angular/router';
     import { OktaCallbackComponent, OktaAuthGuard } from '@okta/okta-angular';
     import { HomeComponent } from './home/home.component';
     import { CallbackComponent } from './callback/callback.component';

     const routes: Routes = [
       { path: '', component: HomeComponent },
       { path: 'callback', component: CallbackComponent },
       { path: 'protected', component: HomeComponent, canActivate: [OktaAuthGuard] },
     ];

     @NgModule({
       imports: [RouterModule.forRoot(routes)],
       exports: [RouterModule],
     })
     export class AppRoutingModule {}
     ```

2. **Create a Login Button:**
   - Add a button in your `app.component.html` to allow users to log in:
     ```html
     <!-- src/app/app.component.html -->
     <button (click)="login()">Login with Okta</button>
     <button *ngIf="isAuthenticated" (click)="logout()">Logout</button>
     ```

   - Implement the login and logout methods in `app.component.ts`:
     ```typescript
     // src/app/app.component.ts
     import { Component, OnInit } from '@angular/core';
     import { OktaAuthStateService, OktaAuthService } from '@okta/okta-angular';

     @Component({
       selector: 'app-root',
       templateUrl: './app.component.html',
     })
     export class AppComponent implements OnInit {
       isAuthenticated: boolean = false;

       constructor(private oktaAuth: OktaAuthService, private oktaStateService: OktaAuthStateService) {}

       async ngOnInit() {
         this.isAuthenticated = await this.oktaStateService.isAuthenticated$().toPromise();
       }

       login() {
         this.oktaAuth.signInWithRedirect();
       }

       logout() {
         this.oktaAuth.signOut();
       }
     }
     ```

### Step 4: Test Your Application

1. **Run the Angular App:**
   - Start your Angular app:
     ```bash
     ng serve
     ```

2. **Access the App:**
   - Open your browser and navigate to `http://localhost:4200`.
   - Click the "Login with Okta" button. You should be redirected to the Okta sign-in page.
   - After successfully logging in, you will be redirected back to your Angular app.

### Step 5: Protect Routes (Optional)

To protect specific routes in your application, you can use the `OktaAuthGuard`. This guard ensures that only authenticated users can access certain routes.

```typescript
// src/app/app-routing.module.ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'callback', component: CallbackComponent },
  { path: 'protected', component: ProtectedComponent, canActivate: [OktaAuthGuard] }, // Protected route
];
```

### Conclusion

You have now integrated Okta into your Angular application, allowing users to authenticate using Okta. This integration leverages Okta's powerful identity management features, making it easy to secure your application and manage user authentication.

### Next Steps

You can further enhance this integration by:

- Handling access tokens and user roles.
- Integrating with other Okta services like MFA (Multi-Factor Authentication).
- Customizing the sign-in experience with Okta-hosted or self-hosted login pages.
- Using the `@okta/okta-angular` library to access user profile information and implement authorization logic based on roles or claims.
