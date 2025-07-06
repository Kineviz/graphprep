# Technical Architecture - KuzuDB ETL Tool

## 1. System Overview

The KuzuDB ETL Tool is a web-based application with a client-server architecture designed for data transformation and graph schema mapping. The system prioritizes performance, scalability, and user experience while maintaining data security through client-side processing.

## 2. Architecture Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Storage       │
│   (React/TS)    │◄──►│   (FastAPI)     │◄──►│   (SQLite/      │
│                 │    │                 │    │    PostgreSQL)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Browser       │    │   File System   │    │   Export        │
│   Storage       │    │   (Temp)        │    │   (KuzuDB)      │
│   (localStorage)│    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 3. Frontend Architecture

### 3.1 Technology Stack
- **Framework**: React 18+ with TypeScript
- **Build Tool**: Vite
- **State Management**: Zustand (client state) + React Query (server state)
- **UI Library**: TailwindCSS + shadcn/ui
- **Table Component**: TanStack Table
- **Graph Editor**: React Flow
- **File Upload**: react-dropzone
- **CSV Parsing**: papaparse

### 3.2 Component Architecture

```
src/
├── components/
│   ├── ui/                    # shadcn/ui components
│   ├── layout/               # Layout components
│   ├── upload/               # File upload components
│   ├── schema/               # Schema inference components
│   ├── transform/            # Transformation components
│   ├── graph/                # Graph schema components
│   ├── validation/           # Validation components
│   └── export/               # Export components
├── hooks/                    # Custom React hooks
├── stores/                   # Zustand stores
├── services/                 # API services
├── types/                    # TypeScript type definitions
├── utils/                    # Utility functions
└── pages/                    # Page components
```

### 3.3 State Management

```typescript
// Main application store
interface AppState {
  // File management
  files: FileData[];
  currentFile: string | null;
  
  // Schema and transformations
  schemas: SchemaData[];
  transformations: Transformation[];
  transformationHistory: HistoryEntry[];
  
  // Graph schema
  nodes: NodeSchema[];
  edges: EdgeSchema[];
  
  // UI state
  currentStep: number;
  validationErrors: ValidationError[];
  
  // Actions
  actions: {
    uploadFile: (file: File) => Promise<void>;
    updateSchema: (schema: SchemaData) => void;
    addTransformation: (transformation: Transformation) => void;
    // ... other actions
  };
}
```

## 4. Backend Architecture

### 4.1 Technology Stack
- **Framework**: FastAPI (Python 3.11+)
- **Database**: SQLite (development), PostgreSQL (production)
- **ORM**: SQLAlchemy with async support
- **Validation**: Pydantic
- **Testing**: Pytest
- **Documentation**: OpenAPI/Swagger

### 4.2 API Design

```python
# API Endpoints
/api/v1/
├── files/
│   ├── upload/              # POST - Upload file
│   ├── {file_id}/           # GET - Get file info
│   ├── {file_id}/preview/   # GET - Get file preview
│   └── {file_id}/validate/  # POST - Validate file
├── schema/
│   ├── infer/               # POST - Infer schema
│   ├── validate/            # POST - Validate schema
│   └── export/              # POST - Export schema
├── transform/
│   ├── preview/             # POST - Preview transformation
│   ├── apply/               # POST - Apply transformation
│   └── history/             # GET - Get transformation history
├── graph/
│   ├── nodes/               # CRUD operations for nodes
│   ├── edges/               # CRUD operations for edges
│   └── validate/            # POST - Validate graph schema
└── export/
    ├── kuzudb/              # POST - Export to KuzuDB format
    ├── json/                # GET - Export schema as JSON
    └── ddl/                 # GET - Generate DDL
```

### 4.3 Data Models

```python
# Pydantic models for API
class FileData(BaseModel):
    id: str
    name: str
    size: int
    type: str
    encoding: str
    delimiter: str
    row_count: int
    columns: List[ColumnInfo]
    created_at: datetime
    updated_at: datetime

class SchemaData(BaseModel):
    file_id: str
    columns: List[ColumnSchema]
    inferred_types: Dict[str, DataType]
    confidence_scores: Dict[str, float]

class Transformation(BaseModel):
    id: str
    type: TransformationType
    expression: str
    target_column: str
    parameters: Dict[str, Any]
    preview_data: List[Any]
    created_at: datetime

class NodeSchema(BaseModel):
    name: str
    source_table: str
    key_column: str
    properties: List[ColumnMapping]
    constraints: List[Constraint]

class EdgeSchema(BaseModel):
    name: str
    source_node: str
    target_node: str
    source_key: str
    target_key: str
    properties: List[ColumnMapping]
```

## 5. Data Flow

### 5.1 File Upload Flow
1. User drags file to upload area
2. Frontend validates file type and size
3. File uploaded to backend with progress tracking
4. Backend processes file (encoding detection, parsing)
5. Preview data returned to frontend
6. Schema inference triggered automatically

### 5.2 Schema Inference Flow
1. Backend samples first 1000 rows
2. Type detection algorithm runs on sample
3. Confidence scores calculated for each column
4. Results cached and returned to frontend
5. User can override inferred types
6. Schema validation runs in background

### 5.3 Transformation Flow
1. User defines transformation expression
2. Frontend validates expression syntax
3. Preview request sent to backend
4. Backend applies transformation to sample data
5. Results returned with performance metrics
6. User confirms transformation
7. Transformation added to history

### 5.4 Graph Schema Flow
1. User defines node schemas from tables
2. User defines edge schemas between nodes
3. Real-time validation of relationships
4. Schema exported in KuzuDB format
5. DDL generation for database creation

## 6. Performance Considerations

### 6.1 Frontend Performance
- **Virtual Scrolling**: For large datasets in tables
- **Lazy Loading**: Components loaded on demand
- **Memoization**: React.memo and useMemo for expensive operations
- **Debouncing**: User input and API calls
- **Caching**: React Query for API responses

### 6.2 Backend Performance
- **Async Processing**: All I/O operations async
- **Connection Pooling**: Database connection management
- **Caching**: Redis for frequently accessed data
- **Streaming**: Large file processing in chunks
- **Background Tasks**: Celery for long-running operations

### 6.3 Memory Management
- **Streaming**: Process large files in chunks
- **Garbage Collection**: Explicit cleanup of large objects
- **Memory Limits**: Configurable limits for file processing
- **Worker Processes**: Separate processes for heavy computation

## 7. Security Considerations

### 7.1 Data Security
- **Client-side Processing**: Sensitive data never stored on server
- **Temporary Storage**: Files stored temporarily with auto-cleanup
- **Input Validation**: All user inputs validated and sanitized
- **File Type Validation**: Strict file type checking

### 7.2 API Security
- **Rate Limiting**: Prevent abuse of API endpoints
- **CORS Configuration**: Proper cross-origin settings
- **Input Sanitization**: SQL injection and XSS prevention
- **Authentication**: JWT tokens for user sessions (future)

## 8. Error Handling

### 8.1 Frontend Error Handling
```typescript
// Error boundary for React components
class ErrorBoundary extends React.Component {
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log error and show user-friendly message
  }
}

// API error handling
const useApiError = () => {
  return useQuery({
    queryFn: fetchData,
    onError: (error) => {
      // Handle different error types
      if (error.status === 413) {
        showError('File too large');
      } else if (error.status === 422) {
        showError('Invalid file format');
      }
    }
  });
};
```

### 8.2 Backend Error Handling
```python
# FastAPI error handlers
@app.exception_handler(ValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(
        status_code=422,
        content={"detail": "Validation error", "errors": exc.errors()}
    )

@app.exception_handler(FileTooLargeError)
async def file_too_large_handler(request, exc):
    return JSONResponse(
        status_code=413,
        content={"detail": "File size exceeds limit"}
    )
```

## 9. Testing Strategy

### 9.1 Frontend Testing
- **Unit Tests**: Jest + React Testing Library
- **Integration Tests**: Component interaction testing
- **E2E Tests**: Playwright for critical user flows
- **Visual Regression**: Storybook for component testing

### 9.2 Backend Testing
- **Unit Tests**: Pytest for individual functions
- **Integration Tests**: API endpoint testing
- **Performance Tests**: Load testing with large files
- **Security Tests**: Input validation and injection testing

## 10. Deployment Architecture

### 10.1 Development Environment
```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
    environment:
      - DATABASE_URL=sqlite:///./dev.db
      
  database:
    image: postgres:15
    environment:
      - POSTGRES_DB=etl_dev
      - POSTGRES_USER=etl_user
      - POSTGRES_PASSWORD=etl_pass
    ports:
      - "5432:5432"
```

### 10.2 Production Environment
- **Container Orchestration**: Kubernetes
- **Load Balancer**: nginx
- **Database**: Managed PostgreSQL service
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK stack
- **CI/CD**: GitHub Actions

## 11. Monitoring and Observability

### 11.1 Metrics
- **Performance**: Response times, throughput
- **Errors**: Error rates, error types
- **Usage**: File uploads, transformations, exports
- **Resources**: CPU, memory, disk usage

### 11.2 Logging
```python
# Structured logging
import structlog

logger = structlog.get_logger()

logger.info(
    "file_uploaded",
    file_id=file_id,
    file_size=file_size,
    file_type=file_type,
    user_id=user_id
)
```

### 11.3 Health Checks
```python
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow(),
        "version": "1.0.0",
        "database": await check_database_connection()
    }
```

## 12. Scalability Considerations

### 12.1 Horizontal Scaling
- **Stateless Backend**: Easy horizontal scaling
- **Database Sharding**: For large datasets
- **CDN**: For static assets
- **Caching Layer**: Redis cluster

### 12.2 Vertical Scaling
- **Resource Limits**: Configurable CPU/memory limits
- **Connection Pooling**: Database connection management
- **Background Processing**: Async task queues

## 13. Future Enhancements

### 13.1 Technical Debt
- **Code Quality**: Automated code reviews
- **Documentation**: API documentation with OpenAPI
- **Testing Coverage**: Maintain >90% test coverage
- **Performance Monitoring**: Continuous performance tracking

### 13.2 Feature Extensions
- **Real-time Collaboration**: WebSocket-based collaboration
- **Plugin System**: Extensible transformation plugins
- **Advanced Analytics**: ML-based data quality insights
- **Multi-tenant Support**: User management and isolation

This technical architecture provides a solid foundation for building a scalable, performant, and maintainable ETL tool for KuzuDB. 