# Nest.js GCP Logger



### Module Parameters

| Property              | Default | Description |
| ---                   | ---     | --- |
| `gcpErrorReporting`   | `false` | If set to `true` all error messages are recognized by GCP Error Reporting by wrapping the provided message in a stack trace |



### How to use

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggingModule } from '@pzwik/nestjs-gcp-logger'; // <-- Import the module
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    LoggingModule.forRoot({ // <-- Initialize the module
      gcpErrorReporting: false // default is 'false'
    })
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule { }
```


```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { LoggingService } from '@pzwik/nestjs-gcp-logger'; // <-- Import here
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { bufferLogs: true });
  app.useLogger(app.get(LoggingService));

  await app.listen(3000);
}
bootstrap();
```

```typescript
import { Injectable, Logger } from '@nestjs/common'; // <-- Import Logger from Nest.js

@Injectable()
export class AppService {
  private readonly logger = new Logger('AppService');

  getHello(): string {
    this.logger.error('Some Error happened!'); // <-- Calls the logger
    // {"severity":"ERROR","message":"Some Error happened!"}
    return 'Hello World!';
  }
}
````

### How to mock in tests

```typescript
  // logger.mock.ts
  export class LoggerMock {
  log() {
    return;
  }
  error() {
    return;
  }
  warn() {
    return;
  }
  debug() {
    return;
  }
  verbose() {
    return;
  }
}
```

```typescript
  // test.spec.ts
  const moduleFixture = await Test.createTestingModule({...})
    .overrideProvider(LoggingService)
    .useClass(LoggerMock)
    .compile();

  const app = moduleFixture.createNestApplication();

  // Do not forget to set the logger, otherwise nestjs default logger
  app.useLogger(app.get(LoggingService));
  
  await app.init();
```

### Migrate 1.X -> 2.X

1. Update module creation
  * from 
  ```ts
  LoggingModule.register()
  ```
  * to 
  ```ts
  LoggingModule.forRoot()
  ```
2. Update name of param
  * from 
  ```ts
  GCP_ERROR_REPORTING
  ```
  * to 
  ```ts
  gcpErrorReporting
  ```
3. Update how logger passed to app 
  * from 
  ```ts
  const app = await NestFactory.create(AppModule, { logger: new LoggingService() });
  ```
  * to 
  ```ts
  const app = await NestFactory.create(AppModule, { bufferLogs: true });
  app.useLogger(app.get(LoggingService))
  ```
4. Update logger usage in services
  * from
  ```ts
  import { Injectable, Logger } from '@nestjs/common';

  @Injectable()
  export class AppService {
    getHello(): string {
      Logger.error('Some Error happened!');
      return 'Hello World!';
    }
  }
  ```
  * to
  ```ts
  import { Injectable, Logger } from '@nestjs/common';

  @Injectable()
  export class AppService {
    private readonly logger = new Logger('AppService');

    getHello(): string {
      this.logger.error('Some Error happened!');
      return 'Hello World!';
    }
  }
  ```


