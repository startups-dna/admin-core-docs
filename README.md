# Admin Packages Integration

## Concept

In order to integrate your App CMS with the Admin system, your app should work with base url `/app-cms`:

```
https://admin.acme.com
- /company
- /app-tools
- /app-cms <-- Your App CMS
```

## Setup

### Configure StartupsDNA npm registry

Create `.npmrc` file in the root of your project and add the following lines:

```
@startupsdna-tools:registry=https://europe-npm.pkg.dev/startupsdna-tools/npm-public/
```

### Install packages

Backend:
    
```bash
npm install @startupsdna-tools/admin-core
```

Frontend:

```bash
npm install @startupsdna-tools/admin-core-ui
```

## Backend Integration

### Add cookie parser

Add cookie parser package to your backend:

```bash
npm install cookie-parser
npm install -D @types/cookie-parser
```

Add cookie parser to your app in `main.ts` file:

```typescript
import cookieParser from 'cookie-parser';

const app = await NestFactory.create(AppModule);
app.use(cookieParser());
```

### Include AdminCoreModule

In your main module:

```typescript
import { AdminCoreModule } from '@startupsdna-tools/admin-core';

@Module({
  imports: [
    AdminCoreModule.forRoot({
      auth: {
        dev: process.env.NODE_ENV === 'development',
        projectId: process.env.ADMIN_AUTH_PROJECT_ID || '',
      },
    }),
  ],
})
```

## Frontend Integration

### Set base url

For Vite, add `base` property to `vite.config.ts` and use it in `BrowserRouter`:

```typescript
import { defineConfig } from 'vite';

export default defineConfig({
  base: '/app-cms',
});
```

```tsx
<BrowserRouter basename={import.meta.env.BASE_URL}>
  {/* Your code */}
</BrowserRouter>
```

### Include Admin

Replace `Admin` component from `react-admin` to `@startupsdna-tools/admin-core-ui`:

```tsx
import { Admin } from '@startupsdna-tools/admin-core-ui';

const App = () => (
  <Admin
    dev={import.meta.env.MODE === 'development'}
    title="CMS"
    {/* ... your other props */}
  >
    {/* ... your other components */}
  </Admin>
);
```

## Permissions

### Declaring permissions in backend

Create a file `permissions.ts` with list of available permissions:

```typescript
import { AdminAuthPermissionSection } from '@startupsdna-tools/admin-core';

export enum PERMISSIONS {
  POSTS = 'app-cms/posts',
  COMMENTS = 'app-cms/comments',
}

export const permissionsConfig: AdminAuthPermissionSection[] = [
  {
    id: PERMISSIONS.POSTS,
    name: 'Posts',
    actions: [
      {
        id: 'view',
        name: 'View',
      },
      {
        id: 'manage',
        name: 'Manage',
      },
    ],
  },
  {
    id: PERMISSIONS.COMMENTS,
    name: 'Comments',
    actions: [
      {
        id: 'view',
        name: 'View',
      },
      {
        id: 'manage',
        name: 'Manage',
      },
    ],
  },
];
```

Add `permissionsConfig` to `AdminCoreModule`:

```typescript
import { AdminCoreModule } from '@startupsdna-tools/admin-core';
import { permissionsConfig } from './permissions';

@Module({
  imports: [
    AdminCoreModule.forRoot({
      auth: {
        /* ... */
        permissions: permissionsConfig,
      },
    }),
  ],
})
```

### Protecting API endpoints

Use `AdminAuth` decorator to protect your endpoints:

```typescript
import { AdminAuth, AdminPermissions } from '@startupsdna-tools/admin-core';
import { PERMISSIONS } from '../permissions';

@Controller('posts')
@AdminAuth(PERMISSIONS.POSTS) // all endpoints in this controller will require any permission from 'app-cms/posts' section
export class PostsController {

  @Get() // will require any permission from 'app-cms/posts' (controller level)
  async find() {
    return {
      data: this.posts,
      total: this.posts.length,
    };
  }

  @Post()
  @AdminPermissions(`${PERMISSIONS.POSTS}:manage`) // this endpoint will require 'app-cms/posts:manage' permission
  async create() {
    // Your code
  }

  @Patch(':id')
  @AdminPermissions(`${PERMISSIONS.POSTS}:manage`) // this endpoint will require 'app-cms/posts:manage' permission
  async update() {
    // Your code
  }
}
```

### Protecting in frontend
Replace `Resource` component from `react-admin` to `@startupsdna-tools/admin-core-ui`:

```tsx
import { Resource } from '@startupsdna-tools/admin-core-ui';

/** Example for users resource */
<Resource
  name="users"
  list={UserList}
  edit={UserEdit}
  create={UserCreate}
  icon={UserIcon}
  options={{ 
    label: 'Users',
    permissions: [
      'app-cms/users' // will require any permission from 'app-cms/posts' section
    ],
  }}
/>
```

To protect any block in component:
    
```tsx
import { IfHasPermission } from '@startupsdna-tools/admin-core-ui';

<IfHasPermission permission="app-cms/posts:manage">
    <Button>Create post</Button>
</IfHasPermission>
```
