# Installing and Using Angular CLI

The error occurs because the Angular CLI (`ng`) is not installed or not accessible from your command line. Here's how to fix it:

1. **Install Angular CLI Globally**

   ```bash
   npm install -g @angular/cli
   ```

2. **Verify Installation**

   ```bash
   ng version
   ```

3. **Generate User Component**
   After installation, run:

   ```bash
   ng generate component user
   # or shorter
   ng g c user
   ```

## If Issues Persist

### Check Node.js Installation

```bash
node --version
npm --version
```

### Alternative Component Generation Methods

1. **Using NPX**

   ```bash
   npx ng g c user
   ```

2. **Using Local Project CLI**

   ```bash
   npm run ng g c user
   ```

## Manual Component Creation

If CLI still doesn't work, create files manually:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-user',
  standalone: true,
  imports: [],
  templateUrl: './user.component.html',
  styleUrl: './user.component.css'
})
export class UserComponent {
}
```

```html
// filepath: d:\Coding\Angular\01-starting-project\src\app\user\user.component.html
<p>user works!</p>
```

```css
// filepath: d:\Coding\Angular\01-starting-project\src\app\user\user.component.css
/* Add styles here */
```

### Note

- Close and reopen your terminal after installing Angular CLI
- Make sure you're in the project directory when running commands
- Windows users might need to update their PATH environment variable

Similar code found with 1 license type
