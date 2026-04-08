# Setting Up Swagger Docs

## Install package

Install with pnpm:
```terminal
pnpm add @nestjs/swagger swagger-ui-express
```

Install with npm:
```terminal
npm install --save @nestjs/swagger
```

## Update ```Main.ts```
Add Swagger doc config and create document factory
```ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
    .setTitle('Swagger API')
    .setDescription('Example Swagger Setup')
    .setVersion('1.0')
    .addTag('test')
    .build();
  const documentFactory = () => SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, documentFactory);
```

Config with JWT Authentication:
```ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
    .setTitle('Swagger API')
    .setDescription('Example Swagger Setup')
    .setVersion('1.0')
    .addTag('test')
    .addBearerAuth(
      {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT',
        description: 'Enter your JWT token',
      },
      'auth-name',
    )
    .build();
  const documentFactory = () => SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, documentFactory);
```