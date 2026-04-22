---
name: mock-service-generator
description: Create mock backend services when real infrastructure (Docker, databases) isn't available in container environments
tags: [mock, backend, nodejs, container, development]
---

# Mock Service Generator

Create lightweight mock services when real infrastructure isn't available (e.g., in Docker containers without Docker-in-Docker).

## Use Case

When developing in a containerized environment where:
- Docker is not available
- Databases can't be started
- External services are unreachable

## Pattern: Express Mock Server

```javascript
const express = require('express');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

// Mock data
const mockData = { /* ... */ };

// Endpoints matching real API
app.get('/api/endpoint', (req, res) => {
  res.json(mockData);
});

app.listen(PORT, () => {
  console.log(`Mock server running on port ${PORT}`);
});
```

## Implementation

### 1. Create mock service directory
```bash
mkdir mock-backend
cd mock-backend
```

### 2. Create package.json
```json
{
  "name": "mock-backend",
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  }
}
```

### 3. Create server.js with mock endpoints
- Match real API contract
- Return realistic sample data
- Include health check endpoint

### 4. Start service
```bash
npm install
node server.js &
```

## Key Learnings

1. **Match API contract** - same endpoints, same response format
2. **Realistic data** - use plausible sample values
3. **Health endpoint** - `/health` for service verification
4. **CORS enabled** - allow frontend access
5. **Lightweight** - minimal dependencies (express + cors only)

## Pitfalls

- Mock data won't reflect real-world variations
- No persistence - data resets on restart
- Limited error simulation
- Not suitable for production testing
