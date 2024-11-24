# Chapter 12: Development Integration

## 12.1 Introduction

SWARM is designed for developers first, making integration straightforward while maintaining powerful capabilities. This chapter covers the various ways developers can build on and interact with the SWARM platform.

### 12.1.1 Integration Options

1. **Direct Integration**
   - Native SDKs
   - RESTful APIs
   - WebSocket connections
   - Command-line interface

2. **Framework Integration**
   - Popular framework plugins
   - Middleware components
   - Build tool integration
   - IDE extensions

3. **Container Integration**
   - Docker support
   - Kubernetes operators
   - Compose files
   - CI/CD plugins

## 12.2 Getting Started

### 12.2.1 Quick Start Example

Basic integration in multiple languages:

```python
# Python SDK
from swarm import Client

client = Client(
    token="your_token",
    region="us-east"
)

# Deploy a simple web service
deployment = await client.deploy(
    name="my-web-app",
    image="nginx:latest",
    replicas=3,
    resources={
        "cpu": "1",
        "memory": "1Gi"
    },
    ports={
        80: "http"
    }
)
```

```typescript
// TypeScript/JavaScript SDK
import { SWARMClient } from '@swarm/sdk';

const client = new SWARMClient({
    token: 'your_token',
    region: 'us-east'
});

// Deploy a serverless function
const function = await client.functions.deploy({
    name: 'process-image',
    runtime: 'nodejs18',
    handler: 'index.handler',
    memory: '512MB',
    timeout: '30s',
    environment: {
        BUCKET: 'images'
    }
});
```

```go
// Go SDK
import "github.com/swarm/sdk-go"

client, err := swarm.NewClient(swarm.Config{
    Token:  "your_token",
    Region: "us-east",
})

// Create a storage bucket
bucket, err := client.Storage.CreateBucket(context.Background(), &swarm.BucketConfig{
    Name:       "user-uploads",
    Encryption: true,
    Lifecycle: &swarm.Lifecycle{
        ExpirationDays: 30,
    },
})
```

## 12.3 API Integration

### 12.3.1 REST API

The SWARM REST API follows OpenAPI 3.0 specifications and uses standard HTTP methods:

```yaml
openapi: 3.0.0
info:
  title: SWARM API
  version: '1.0'
paths:
  /compute/instances:
    post:
      summary: Create compute instance
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                size:
                  type: string
                image:
                  type: string
      responses:
        '200':
          description: Instance created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Instance'
```

Example usage:

```typescript
class APIClient {
    private baseUrl: string;
    private token: string;

    async createInstance(config: InstanceConfig): Promise<Instance> {
        const response = await fetch(`${this.baseUrl}/compute/instances`, {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${this.token}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(config)
        });

        if (!response.ok) {
            throw await this.handleError(response);
        }

        return response.json();
    }

    private async handleError(response: Response): Promise<Error> {
        const error = await response.json();
        return new SWARMError(error.message, {
            code: error.code,
            requestId: response.headers.get('x-request-id')
        });
    }
}
```

### 12.3.2 WebSocket API

Real-time updates and streaming data:

```typescript
class WebSocketClient {
    private ws: WebSocket;
    private subscriptions: Map<string, Callback>;

    connect(url: string): Promise<void> {
        return new Promise((resolve, reject) => {
            this.ws = new WebSocket(url);
            
            this.ws.onopen = () => {
                this.startHeartbeat();
                resolve();
            };

            this.ws.onmessage = (event) => {
                const message = JSON.parse(event.data);
                this.handleMessage(message);
            };

            this.ws.onerror = (error) => {
                reject(error);
            };
        });
    }

    subscribe(topic: string, callback: Callback): void {
        this.subscriptions.set(topic, callback);
        this.ws.send(JSON.stringify({
            type: 'subscribe',
            topic
        }));
    }

    private handleMessage(message: WebSocketMessage): void {
        const callback = this.subscriptions.get(message.topic);
        if (callback) {
            callback(message.data);
        }
    }
}
```

## 12.4 Framework Integration

### 12.4.1 Express.js Middleware

```typescript
import express from 'express';
import { swarmMiddleware } from '@swarm/express';

const app = express();

// Add SWARM middleware
app.use(swarmMiddleware({
    auth: true,
    monitoring: true,
    rateLimit: {
        windowMs: 15 * 60 * 1000, // 15 minutes
        max: 100 // limit each IP to 100 requests per windowMs
    }
}));

// Use SWARM services in routes
app.post('/upload', async (req, res) => {
    const storage = req.swarm.storage;
    const file = req.files.upload;
    
    const result = await storage.upload({
        bucket: 'uploads',
        key: file.name,
        body: file.data,
        metadata: {
            contentType: file.mimetype
        }
    });
    
    res.json(result);
});
```

### 12.4.2 Django Integration

```python
# settings.py
INSTALLED_APPS = [
    ...
    'swarm.django',
]

SWARM_CONFIG = {
    'token': 'your_token',
    'region': 'us-east',
    'features': {
        'storage': True,
        'functions': True,
        'ai': True
    }
}

# views.py
from swarm.django.decorators import swarm_authenticated
from swarm.django.storage import swarm_storage

@swarm_authenticated
def process_image(request):
    if request.method == 'POST':
        image = request.FILES['image']
        
        # Upload to SWARM storage
        key = await swarm_storage.upload(
            bucket='images',
            file=image,
            metadata={
                'user': request.user.id
            }
        )
        
        # Process with SWARM AI
        result = await request.swarm.ai.process_image(key, {
            'operations': ['resize', 'optimize']
        })
        
        return JsonResponse(result)
```

## 12.5 Development Tools

### 12.5.1 CLI Integration

```typescript
class SWARMDevTools {
    private config: DevConfig;
    private environment: DevEnvironment;

    async setupLocal(): Promise<void> {
        // Initialize local development environment
        await this.environment.init({
            services: ['compute', 'storage', 'ai'],
            emulators: true,
            hotReload: true
        });

        // Set up local endpoints
        const endpoints = await this.environment.getEndpoints();
        
        // Configure local development
        await this.configureLocal(endpoints);
    }

    async runTests(): Promise<TestResults> {
        // Run test suite
        const runner = new TestRunner(this.config);
        
        // Execute tests against local environment
        const results = await runner.run({
            coverage: true,
            parallel: true,
            watch: true
        });
        
        return results;
    }
}
```

### 12.5.2 Build Integration

```typescript
// webpack.config.js
const SWARMWebpackPlugin = require('@swarm/webpack-plugin');

module.exports = {
    // ... other webpack config
    plugins: [
        new SWARMWebpackPlugin({
            mode: 'development',
            emulators: true,
            endpoints: {
                api: 'http://localhost:3000',
                storage: 'http://localhost:3001'
            },
            features: ['storage', 'functions']
        })
    ]
};
```

## 12.6 Testing and Debugging

### 12.6.1 Testing Utilities

```typescript
class SWARMTestKit {
    private emulators: ServiceEmulators;
    private testData: TestData;

    async setupTestEnvironment(): Promise<TestEnvironment> {
        // Start local emulators
        await this.emulators.start({
            services: ['compute', 'storage', 'functions']
        });

        // Load test data
        await this.testData.load();

        // Configure test client
        const client = await this.createTestClient();

        return { client, emulators: this.emulators };
    }

    async runIntegrationTests(): Promise<void> {
        const env = await this.setupTestEnvironment();

        try {
            // Run tests
            await this.runTests(env);
        } finally {
            // Cleanup
            await env.emulators.stop();
        }
    }
}
```

### 12.6.2 Debugging Tools

```typescript
class SWARMDebugger {
    private tracer: Tracer;
    private logger: Logger;

    async attachDebugger(service: string): Promise<void> {
        // Connect to service
        await this.connect(service);

        // Set up logging
        await this.configureLogging({
            level: 'debug',
            format: 'json',
            destination: 'console'
        });

        // Initialize tracing
        await this.initializeTracing({
            sampling: 1.0,
            exporters: ['console', 'jaeger']
        });

        // Start debugging session
        await this.startDebugging();
    }

    async inspectRequest(requestId: string): Promise<RequestInfo> {
        // Get request trace
        const trace = await this.tracer.getTrace(requestId);

        // Get related logs
        const logs = await this.logger.getLogs({
            requestId,
            timeRange: '1h'
        });

        return {
            trace,
            logs,
            timeline: this.buildTimeline(trace, logs)
        };
    }
}
```

This chapter provides comprehensive coverage of the various ways developers can integrate with and build on the SWARM platform, with practical examples and code snippets for common integration patterns.