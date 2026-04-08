# Swagger Controller Annotations

## Tags
Define a specific tag for a controller:
```ts
@ApiTags('prompts')
```

## Controller Name
Define a specific name for a controller:
```ts
@Controller('prompts')
```

## API Auth

Define the authentication type:
```ts
@ApiBearerAuth('auth-name')
```

## Response Types
Specify the response types:
```ts
@ApiResponse({ status: 201, description: 'The record has been successfully created.'})
@ApiResponse({ status: 403, description: 'Forbidden.'})
```