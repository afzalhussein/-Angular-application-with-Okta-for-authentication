# -Angular-application-with-Okta-for-authentication
Integrating an Angular application with Okta for authentication is a common scenario for securing your web application. Below, I'll provide a step-by-step example of how to integrate an Angular app with Okta using the Okta Angular SDK. This example will guide you through setting up a basic Angular app that uses Okta for user authentication.

**Step 1: Set Up an Okta Account**

If you don't have an Okta account, sign up for one at [Okta's website](https://developer.okta.com/). Once you have an account, log in to the Okta Developer Console.

**Step 2: Create an Okta Application**

- In the Okta Developer Console, navigate to "Applications" and click the "Create App Integration" button.
- Choose "Single Page App" as the application type.
- Configure the "Login redirect URIs" and "Logout redirect URIs" (e.g., `http://localhost:4200/login/callback` for development).
- Save the application settings and note the "Client ID."

**Step 3: Set Up Your Angular Application**

- Create a new Angular app or use an existing one. You can create one using Angular CLI:

```bash
ng new my-okta-app
cd my-okta-app
```

- Install the Okta Angular SDK:

```bash
npm install @okta/okta-angular --save
```

**Step 4: Configure Okta in Your Angular App**

- In your Angular app, import the `OktaAuthModule` and configure it with your Okta settings. Create a file called `app.module.ts` and add the following:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { OktaAuthModule } from '@okta/okta-angular';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    OktaAuthModule.initAuth({
      issuer: 'https://your-okta-domain.okta.com/oauth2/default',
      clientId: 'your-client-id',
      redirectUri: window.location.origin + '/login/callback',
      pkce: true,
    }),
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

- Replace `'https://your-okta-domain.okta.com/oauth2/default'` and `'your-client-id'` with your Okta domain and client ID obtained from Step 2.

**Step 5: Create Authentication Guard**

- Create an authentication guard to protect your application routes. Generate a new guard:

```bash
ng generate guard auth
```

- Modify the `auth.guard.ts` file to use Okta's `OktaAuthGuard`:

```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { OktaAuthService } from '@okta/okta-angular';

@Injectable({
  providedIn: 'root',
})
export class AuthGuard implements CanActivate {
  constructor(public oktaAuth: OktaAuthService, private router: Router) {}

  canActivate(): Promise<boolean> {
    return this.oktaAuth.isAuthenticated().then((authenticated) => {
      if (!authenticated) {
        this.router.navigate(['/login']);
        return false;
      }
      return true;
    });
  }
}
```

**Step 6: Create a Login Component**

- Create a login component for handling the login process:

```bash
ng generate component login
```

- In the `login.component.ts` file, add a login button that redirects to Okta's login page:

```typescript
import { Component } from '@angular/core';
import { OktaAuthService } from '@okta/okta-angular';

@Component({
  selector: 'app-login',
  template: '<button (click)="login()">Login</button>',
})
export class LoginComponent {
  constructor(public oktaAuth: OktaAuthService) {}

  login() {
    this.oktaAuth.loginRedirect('/');
  }
}
```

**Step 7: Create Routing and Protected Component**

- Set up routing in your app. Modify the `app-routing.module.ts` file to include your login and protected components:

```typescript
import { AuthGuard } from './auth.guard';
import { LoginComponent } from './login/login.component';
import { ProtectedComponent } from './protected/protected.component';

const routes: Routes = [
  { path: 'login', component: LoginComponent },
  { path: 'protected', component: ProtectedComponent, canActivate: [AuthGuard] },
  { path: '', redirectTo: 'protected', pathMatch: 'full' },
];
```

**Step 8: Create a Protected Component**

- Create a protected component that will be accessible only to authenticated users:

```bash
ng generate component protected
```

- In the `protected.component.html` file, add content that should be displayed only to authenticated users.

**Step 9: Run Your Application**

- Start your Angular app:

```bash
ng serve
```

- Visit `http://localhost:4200` in your browser. You should be redirected to the Okta login page when you try to access the protected route.

This example demonstrates a basic integration of Okta with an Angular app for authentication. You can further customize your app and explore Okta's documentation for more advanced features and configurations, such as adding user registration, using custom claims, and handling session management.
