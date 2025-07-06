🧾 ETL Tool with GUI for KuzuDB – Requirements Specification

1. Overview

A web-based ETL application that allows users to import structured data (initially CSVs, later Parquet and SQL query results), define transformation rules, and export the processed data in a format compatible with KuzuDB's node/edge schema. The tool should support visual editing, schema design, transformation, and validation with schema import/export capabilities. Inspired by Google Dataprep (Trifacta), the interface emphasizes spreadsheet-like previews, transformation history, and contextual suggestions.

2. Key Features

2.1 Data Ingestion

* Upload multiple CSV files (future: Parquet and SQL query integration)
* Preview a sample of each table in a spreadsheet-like view
* Drag-and-drop file upload interface

**Technical Implementation Details:**
- **File Upload**: Use `react-dropzone` for drag-and-drop with progress indicators
- **CSV Parsing**: Client-side parsing with `papaparse` for immediate preview, backend validation
- **File Size Limits**: 50MB per file for client-side processing, larger files require backend streaming
- **Encoding Detection**: Auto-detect UTF-8, UTF-16, ISO-8859-1 using `jschardet`
- **Delimiter Detection**: Auto-detect comma, tab, semicolon, pipe using frequency analysis
- **Memory Management**: Stream large files in chunks, implement virtual scrolling for preview

**Potential Challenges:**
- Large file handling (>100MB) requires backend streaming
- Mixed encodings in single file
- Malformed CSV with inconsistent delimiters
- Memory constraints in browser for large datasets

2.2 Type Inference and Editing

* Auto-detect data type for each column (string, int, float, date, boolean)
* Allow manual override of inferred types
* Contextual suggestions based on column data (like Trifacta)

**Technical Implementation Details:**
- **Type Detection Algorithm**:
  - Sample first 1000 rows for type inference
  - Boolean: Check for true/false, yes/no, 1/0 patterns
  - Integer: All values parse as integers, no decimals
  - Float: Contains decimal points, scientific notation
  - Date: Multiple date format detection (ISO, MM/DD/YYYY, DD/MM/YYYY, etc.)
  - String: Default fallback
- **Confidence Scoring**: Assign confidence levels (0-100%) for each type inference
- **Pattern Recognition**: Use regex patterns for email, phone, URL detection
- **Contextual Suggestions**: ML-based suggestions using column name patterns and data distribution

**Potential Challenges:**
- Ambiguous data types (e.g., "123" could be string ID or integer)
- Mixed data types in single column
- Date format variations across regions
- Performance with large datasets during inference

2.3 Data Transformation

* Force type conversion (e.g. cast float to string)
* Date parsing and formatting (e.g., from "MM/DD/YYYY" to ISO 8601)
* String operations (e.g., trimming, concatenation, case normalization)
* Combine two or more columns into one (e.g., full name = first + last)
* Generate a new UID for each row (UUIDv4 or auto-increment)
* View a transformation history with undo/redo support

**Technical Implementation Details:**
- **Transformation Engine**: Expression-based transformation using a custom DSL
- **Supported Operations**:
  - Type casting: `CAST(column, type)`
  - String operations: `TRIM()`, `UPPER()`, `LOWER()`, `CONCAT()`, `SUBSTRING()`
  - Date operations: `PARSE_DATE()`, `FORMAT_DATE()`, `EXTRACT_YEAR()`
  - Mathematical: `ROUND()`, `CEIL()`, `FLOOR()`, `ABS()`
  - Conditional: `IF()`, `CASE()`, `COALESCE()`
- **Transformation History**: Immutable operation log with command pattern
- **Preview System**: Real-time preview of transformations on sample data
- **Performance**: Lazy evaluation, caching of intermediate results

**Potential Challenges:**
- Complex transformation chains affecting performance
- Error handling for invalid transformations
- Memory management for large transformation histories
- Real-time preview with complex operations

2.4 Graph Schema Mapping

* Visual interface to declare Nodes and Edges
* Select columns as node properties
* Select pairs of tables/columns for edges
* For each node/edge:
  * Designate a merge key column
  * Allow creation of a new column by concatenating existing ones for the key
* Optional visual graph editor using React Flow

**Technical Implementation Details:**
- **Schema Model**:
  ```typescript
  interface NodeSchema {
    name: string;
    sourceTable: string;
    keyColumn: string;
    properties: ColumnMapping[];
    constraints: Constraint[];
  }
  
  interface EdgeSchema {
    name: string;
    sourceNode: string;
    targetNode: string;
    sourceKey: string;
    targetKey: string;
    properties: ColumnMapping[];
  }
  ```
- **Visual Editor**: React Flow integration with custom node types
- **Key Generation**: Support for composite keys, UUID generation, hash-based keys
- **Validation**: Real-time validation of schema relationships
- **Auto-suggestions**: Suggest relationships based on column name patterns

**Potential Challenges:**
- Complex many-to-many relationship mapping
- Circular reference detection
- Performance with large schema graphs
- Schema versioning and migration

2.4.1 Data Modeling Guidelines

**Node Modeling:**
- **Entity Identification**: Clear identification of what constitutes a node entity
- **Property Selection**: Choose relevant properties that add value to the graph
- **Key Strategy**: Use stable, unique identifiers as node keys
- **Labeling**: Use descriptive labels for node types (e.g., Person, Company, Product)

**Edge Modeling:**
- **Relationship Types**: Define meaningful relationship types with descriptive names
- **Directionality**: Consider whether relationships are directional or bidirectional
- **Properties**: Add properties to edges when they describe the relationship itself
- **Cardinality**: Handle one-to-one, one-to-many, and many-to-many relationships

**Best Practices:**
- **Normalization**: Avoid redundant data in node properties
- **Consistency**: Use consistent naming conventions across the graph
- **Performance**: Consider query patterns when designing the schema
- **Scalability**: Design for future growth and data volume

2.4.2 Advanced Mapping Features

**Smart Relationship Detection:**
- **Foreign Key Inference**: Automatically detect potential foreign key relationships
- **Pattern Matching**: Use column name patterns to suggest relationships
- **Data Analysis**: Analyze data overlap to suggest connections
- **Confidence Scoring**: Provide confidence levels for suggested relationships

**Composite Key Support:**
- **Multi-column Keys**: Support for keys composed of multiple columns
- **Key Generation**: Automatic generation of composite keys
- **Key Validation**: Ensure uniqueness and stability of composite keys

**Relationship Properties:**
- **Edge Attributes**: Map columns to edge properties
- **Temporal Properties**: Handle time-based relationship properties
- **Weighted Relationships**: Support for relationship weights and scores

2.5 Schema Persistence

* Export ETL schema as JSON
* Re-import ETL schema from JSON
* Autosave working schema in local storage

**Technical Implementation Details:**
- **Schema Format**: JSON Schema for validation
- **Versioning**: Include schema version for backward compatibility
- **Autosave**: Debounced saves to localStorage with conflict resolution
- **Export Formats**: JSON, YAML, and KuzuDB DDL
- **Import Validation**: Validate imported schemas against current data

**Potential Challenges:**
- Schema evolution and backward compatibility
- Large schema serialization performance
- Cross-browser localStorage limitations
- Schema sharing and collaboration

2.6 Validation

* Run data validation checks before export:
  * Missing values in merge keys
  * Type mismatch
  * Invalid date formats
  * Unresolved references in edge definitions
* Present errors in a side panel with inline row highlighting

**Technical Implementation Details:**
- **Validation Engine**: Rule-based validation with custom validators
- **Validation Types**:
  - Data quality: missing values, duplicates, outliers
  - Schema compliance: type mismatches, constraint violations
  - Graph integrity: orphaned nodes, broken relationships
- **Error Reporting**: Categorized errors with severity levels
- **Performance**: Incremental validation, background processing
- **Visual Feedback**: Inline highlighting, error tooltips, summary dashboard

**Potential Challenges:**
- Performance with large datasets
- Complex validation rule combinations
- User-friendly error messages
- Validation rule customization

2.7 Import Optimization

**Index and Constraint Management:**
- **Automatic Indexing**: Suggest optimal indexes based on query patterns
- **Constraint Validation**: Ensure data integrity constraints are met
- **Performance Tuning**: Optimize import performance for target databases
- **Batch Processing**: Support for large-scale data imports

**Data Quality Assurance:**
- **Duplicate Detection**: Identify and handle duplicate records
- **Data Cleansing**: Automatic data cleaning and normalization
- **Outlier Detection**: Identify and flag data outliers
- **Completeness Checks**: Ensure required fields are populated

**Import Monitoring:**
- **Progress Tracking**: Real-time progress indicators during import
- **Error Handling**: Graceful handling of import errors
- **Rollback Capability**: Ability to rollback failed imports
- **Performance Metrics**: Track import performance and optimization opportunities

3. Non-Functional Requirements

* Platform: Web-based (React + TypeScript frontend, Python backend)
* UI Libraries: TailwindCSS + shadcn/ui; Table: TanStack Table or AG Grid
* Performance: Handle ~10K rows interactively; offload large jobs to backend
* Security: No sensitive data stored permanently; all in-browser or local server
* Extensibility: Plugin system for new data formats or transformations

**Technical Architecture:**
- **Frontend**: React 18+ with TypeScript, Vite for build tooling
- **Backend**: FastAPI with Python 3.11+, async/await for performance
- **Database**: SQLite for local storage, PostgreSQL for production
- **State Management**: Zustand for client state, React Query for server state
- **Testing**: Jest + React Testing Library, Pytest for backend
- **Deployment**: Docker containers, nginx reverse proxy

**Performance Targets:**
- Initial load: <2 seconds
- File upload: <5 seconds for 10MB files
- Type inference: <1 second for 10K rows
- Transformation preview: <500ms
- Schema validation: <2 seconds for 10K rows

**Security Considerations:**
- Client-side data processing (no server storage of sensitive data)
- CORS configuration for local development
- Input sanitization and validation
- Rate limiting for API endpoints
- Secure file upload validation

4. Future Extensions

* Connect to live databases (via SQL)
* Handle streaming data or incremental updates
* Support for KuzuDB schema introspection or DDL auto-generation
* Export in formats compatible with other graph databases (e.g., Neo4j CSV)

**Technical Roadmap:**
- **Phase 2**: Database connectivity with connection pooling
- **Phase 3**: Real-time streaming with WebSockets
- **Phase 4**: Advanced analytics and ML features
- **Phase 5**: Multi-user collaboration and version control

4.1 Intended Output Formats

The GraphPrep platform is designed to support multiple output formats for maximum compatibility and flexibility:

**Primary Target:**
- **KuzuDB**: Native format with optimized schema and data files

**Extended Graph Database Support:**
- **GraphXR**: Export format optimized for GraphXR visualization platform
- **Neo4j**: CSV format compatible with Neo4j's LOAD CSV command and Neo4j Data Importer
- **UltipaGraph**: UltipaGraph format

**Technical Implementation Details:**
- **Format Adapters**: Plugin-based architecture for output format generation
- **Schema Translation**: Automatic mapping between different graph database schemas
- **Data Validation**: Format-specific validation rules
- **Performance Optimization**: Streaming export for large datasets
- **Metadata Preservation**: Maintain data lineage and transformation history

**Export Features:**
- **Batch Export**: Generate multiple format outputs simultaneously
- **Custom Mappings**: User-defined field mappings for specific databases
- **Template System**: Reusable export templates for common use cases
- **Incremental Export**: Support for delta updates and data synchronization

**Neo4j-Specific Export Features:**
- **LOAD CSV Compatibility**: Generate CSV files compatible with Neo4j's LOAD CSV command
- **Data Importer Format**: Export in format compatible with Neo4j Data Importer UI
- **Cypher Script Generation**: Generate Cypher scripts for data import
- **Index and Constraint Scripts**: Generate DDL scripts for indexes and constraints
- **Bulk Import Optimization**: Optimize for Neo4j's bulk import tools
- **Relationship Direction**: Handle directional and bidirectional relationships
- **Property Types**: Map data types to Neo4j property types
- **Label Management**: Generate appropriate node labels and relationship types

**Potential Challenges:**
- Schema compatibility across different graph database paradigms
- Performance optimization for large-scale exports
- Maintaining data integrity during format conversion
- Handling database-specific features and constraints

5. User Workflow (Example)

6. Upload a CSV

7. Auto-infer schema → Review/override column types

8. Transform columns (dates, merge fields)

9. Define node tables and their merge keys

10. Define edge tables from relationships

11. Validate schema and data

12. Export schema + data for ingestion into KuzuDB

**Detailed Workflow Implementation:**
- **Step Navigation**: URL-based routing with state persistence
- **Progress Tracking**: Visual progress indicators with validation gates
- **Error Recovery**: Graceful error handling with recovery suggestions
- **Keyboard Shortcuts**: Power user shortcuts for common operations
- **Tutorial System**: Interactive onboarding for new users

5.1 Enhanced User Workflow

**Step 1: Data Provision**
- **File Upload**: Drag-and-drop interface with file validation
- **Data Preview**: Immediate preview of uploaded data with sample rows
- **Encoding Detection**: Automatic detection and handling of file encodings
- **Delimiter Detection**: Smart detection of CSV delimiters
- **File Validation**: Check for common file format issues

**Step 2: Data Modeling**
- **Entity Identification**: Help users identify what constitutes nodes in their data
- **Relationship Discovery**: Suggest potential relationships between entities
- **Schema Visualization**: Visual representation of the proposed graph schema
- **Best Practices Guidance**: Provide tips for effective graph modeling
- **Template Library**: Pre-built templates for common use cases

**Step 3: Data Mapping**
- **Column Mapping**: Map source columns to node/edge properties
- **Key Selection**: Choose appropriate keys for nodes and relationships
- **Type Mapping**: Map data types to target database types
- **Transformation Rules**: Apply data transformations during mapping
- **Validation Preview**: Preview mapped data before import

**Step 4: Schema Validation**
- **Data Quality Checks**: Validate data integrity and completeness
- **Relationship Validation**: Ensure all relationships are properly defined
- **Constraint Validation**: Check for constraint violations
- **Performance Analysis**: Analyze potential performance issues
- **Error Resolution**: Provide guidance for fixing validation errors

**Step 5: Import Preparation**
- **Target Selection**: Choose target database and format
- **Import Strategy**: Select appropriate import method (bulk, streaming, etc.)
- **Performance Optimization**: Optimize import settings for target database
- **Script Generation**: Generate import scripts and DDL
- **Documentation**: Generate documentation for the import process

**Step 6: Import Execution**
- **Progress Monitoring**: Real-time progress tracking during import
- **Error Handling**: Graceful handling of import errors
- **Rollback Capability**: Ability to rollback failed imports
- **Performance Metrics**: Track import performance and optimization
- **Success Validation**: Verify successful import and data integrity

**Workflow Enhancements:**
- **Guided Tours**: Interactive tutorials for new users
- **Contextual Help**: Inline help and tooltips throughout the process
- **Error Recovery**: Smart suggestions for fixing common issues
- **Performance Insights**: Recommendations for optimizing import performance
- **Collaboration Features**: Share and collaborate on import configurations

13. UI Wireframe Overview

* **Top Navigation Bar**: Logo, Project Name, Save Status, Export Button
* **Left Sidebar (Multi-step Navigation)**:
  * Step 1: Upload
  * Step 2: Schema & Type Inference
  * Step 3: Transform
  * Step 4: Graph Schema
  * Step 5: Validation
  * Step 6: Export
* **Main Canvas Area** (changes by step):
  * Upload: Drag-and-drop + file list
  * Schema: Editable table with data types
  * Transform: Spreadsheet view with transform suggestions and preview
  * Graph: Visual map + merge key definition (optionally via React Flow)
  * Validation: Issues list + table row highlights
  * Export: Format selector + schema JSON view
* **Bottom Panel** (optional): Transformation History or Console Log

**UI/UX Specifications:**
- **Responsive Design**: Mobile-first approach with breakpoints
- **Accessibility**: WCAG 2.1 AA compliance
- **Theme Support**: Light/dark mode with system preference detection
- **Internationalization**: i18n support for multiple languages
- **Component Library**: Custom design system built on shadcn/ui

14. Implementation Phases

**Phase 1: Core Foundation (Weeks 1-4)**
- Project setup and architecture
- Basic file upload and CSV parsing
- Simple type inference
- Basic UI framework

**Phase 2: Data Processing (Weeks 5-8)**
- Transformation engine
- Schema mapping interface
- Basic validation
- Export functionality

**Phase 3: Advanced Features (Weeks 9-12)**
- Visual graph editor
- Advanced transformations
- Performance optimizations
- Error handling improvements

**Phase 4: Polish & Testing (Weeks 13-16)**
- UI/UX refinements
- Comprehensive testing
- Documentation
- Deployment preparation

15. Risk Assessment

**High Risk:**
- Large file performance issues
- Complex transformation performance
- Browser memory limitations
- Schema validation complexity

**Medium Risk:**
- Type inference accuracy
- Visual graph editor complexity
- Cross-browser compatibility
- State management complexity

**Low Risk:**
- Basic UI implementation
- File upload functionality
- Simple transformations
- Export functionality

16. Success Metrics

- **Performance**: <2s initial load, <500ms for interactions
- **Usability**: <5 minutes to complete basic ETL workflow
- **Reliability**: 99.9% uptime, <1% error rate
- **Adoption**: User retention >80% after first use
- **Quality**: <5 critical bugs in production

This wireframe and feature set balance simplicity for new users and power for data engineers, using Google Dataprep as the UX benchmark.

## 17. Technical Implementation Requirements

### 17.1 Frontend Technology Stack

**Core Framework:**
- **React 18+** with TypeScript for type safety and modern React features
- **Vite** for fast development and optimized builds
- **React Router v6** for client-side routing with nested routes

**State Management:**
- **Zustand** for client-side state management (inspired by Google DataPrep's approach)
- **React Query (TanStack Query)** for server state management and caching
- **Immer** for immutable state updates

**UI Framework:**
- **TailwindCSS** for utility-first styling
- **shadcn/ui** for pre-built, accessible components
- **Radix UI** for headless component primitives
- **Lucide React** for consistent iconography

**Data Processing & Visualization:**
- **TanStack Table (React Table v8)** for spreadsheet-like data display
- **React Flow** for visual graph schema editor
- **D3.js** for custom data visualizations and charts
- **react-dropzone** for drag-and-drop file uploads
- **papaparse** for client-side CSV parsing

**Form Handling & Validation:**
- **React Hook Form** for performant form management
- **Zod** for schema validation (compatible with React Hook Form)
- **react-hot-toast** for user notifications

**Development Tools:**
- **ESLint** with TypeScript rules
- **Prettier** for code formatting
- **Husky** for git hooks
- **Vitest** for unit testing
- **Playwright** for E2E testing

### 17.2 Backend Technology Stack

**Web Framework:**
- **FastAPI** for high-performance async API development
- **Pydantic** for data validation and serialization
- **Uvicorn** as ASGI server

**Database & ORM:**
- **SQLAlchemy 2.0** with async support
- **Alembic** for database migrations
- **PostgreSQL** for production (SQLite for development)
- **Redis** for caching and session storage

**Data Processing:**
- **Pandas** for data manipulation and analysis
- **NumPy** for numerical operations
- **Polars** for high-performance data processing (alternative to Pandas)
- **Apache Arrow** for efficient data serialization

**File Processing:**
- **python-magic** for file type detection
- **chardet** for encoding detection
- **aiofiles** for async file operations
- **python-multipart** for file upload handling

**Validation & Parsing:**
- **jsonschema** for JSON schema validation
- **python-dateutil** for date parsing
- **email-validator** for email validation
- **phonenumbers** for phone number validation

**Background Processing:**
- **Celery** with Redis for background tasks
- **RQ (Redis Queue)** as lightweight alternative
- **Dramatiq** for modern async task processing

**Monitoring & Logging:**
- **Structlog** for structured logging
- **Prometheus** for metrics collection
- **Sentry** for error tracking
- **OpenTelemetry** for distributed tracing

### 17.3 Data Processing Libraries

**Type Inference & Analysis:**
```python
# Type detection libraries
import pandas as pd
import numpy as np
from dateutil import parser
import re
from typing import Dict, List, Any, Optional

class TypeInferenceEngine:
    def __init__(self):
        self.patterns = {
            'email': r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
            'phone': r'^\+?[\d\s\-\(\)]+$',
            'url': r'^https?://[^\s/$.?#].[^\s]*$',
            'uuid': r'^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$'
        }
    
    def infer_type(self, series: pd.Series) -> Dict[str, Any]:
        # Implementation for type inference
        pass
```

**Transformation Engine:**
```python
# Transformation libraries
import pandas as pd
import numpy as np
from datetime import datetime
import uuid
import hashlib

class TransformationEngine:
    def __init__(self):
        self.functions = {
            'cast': self._cast_type,
            'trim': self._trim_string,
            'upper': self._to_upper,
            'lower': self._to_lower,
            'concat': self._concatenate,
            'parse_date': self._parse_date,
            'generate_uuid': self._generate_uuid,
            'hash': self._hash_value
        }
    
    def apply_transformation(self, data: pd.DataFrame, transformation: Dict) -> pd.DataFrame:
        # Implementation for applying transformations
        pass
```

### 17.4 Graph Database Integration

**Neo4j Integration:**
```python
# Neo4j libraries
from neo4j import GraphDatabase
import pandas as pd
from typing import List, Dict

class Neo4jExporter:
    def __init__(self, uri: str, user: str, password: str):
        self.driver = GraphDatabase.driver(uri, auth=(user, password))
    
    def export_nodes(self, nodes_df: pd.DataFrame, label: str, properties: List[str]):
        # Implementation for Neo4j node export
        pass
    
    def export_relationships(self, edges_df: pd.DataFrame, relationship_type: str):
        # Implementation for Neo4j relationship export
        pass
    
    def generate_cypher_script(self, schema: Dict) -> str:
        # Implementation for Cypher script generation
        pass
```

**KuzuDB Integration:**
```python
# KuzuDB libraries
import kuzu
from typing import Dict, List

class KuzuDBExporter:
    def __init__(self, db_path: str):
        self.db = kuzu.Database(db_path)
    
    def create_schema(self, schema: Dict):
        # Implementation for KuzuDB schema creation
        pass
    
    def import_data(self, nodes: Dict, edges: Dict):
        # Implementation for KuzuDB data import
        pass
```

### 17.5 Performance Optimization Libraries

**Frontend Performance:**
```typescript
// Performance optimization libraries
import { useMemo, useCallback, memo } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';
import { debounce } from 'lodash-es';

// Virtual scrolling for large datasets
const VirtualizedTable = memo(({ data }: { data: any[] }) => {
  const rowVirtualizer = useVirtualizer({
    count: data.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 35,
  });
  
  return (
    // Implementation for virtualized table
  );
});
```

**Backend Performance:**
```python
# Performance optimization libraries
import asyncio
import aiofiles
from concurrent.futures import ThreadPoolExecutor
import multiprocessing as mp

class AsyncDataProcessor:
    def __init__(self):
        self.executor = ThreadPoolExecutor(max_workers=mp.cpu_count())
    
    async def process_large_file(self, file_path: str, chunk_size: int = 10000):
        # Implementation for async file processing
        pass
    
    async def parallel_transformation(self, data: pd.DataFrame, transformations: List):
        # Implementation for parallel transformations
        pass
```

### 17.6 Testing Framework

**Frontend Testing:**
```typescript
// Testing libraries
import { render, screen, fireEvent } from '@testing-library/react';
import { vi } from 'vitest';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

// Test setup
const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

const renderWithProviders = (component: React.ReactElement) => {
  const queryClient = createTestQueryClient();
  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};
```

**Backend Testing:**
```python
# Testing libraries
import pytest
import pytest-asyncio
from httpx import AsyncClient
from unittest.mock import Mock, patch

# Test fixtures
@pytest.fixture
async def async_client():
    from app.main import app
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.fixture
def mock_file_service():
    with patch('app.services.file_service.FileService') as mock:
        yield mock
```

### 17.7 Development & Deployment Tools

**Development Environment:**
- **Docker** for containerization
- **Docker Compose** for local development
- **Make** for build automation
- **Pre-commit** for code quality hooks

**CI/CD Pipeline:**
```yaml
# GitHub Actions workflow
name: CI/CD Pipeline
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          cd frontend && npm ci
          cd ../backend && pip install -r requirements.txt
      - name: Run tests
        run: |
          cd frontend && npm test
          cd ../backend && pytest
```

**Monitoring & Observability:**
```python
# Monitoring setup
import structlog
from prometheus_client import Counter, Histogram
import sentry_sdk

# Structured logging
logger = structlog.get_logger()

# Metrics
file_upload_counter = Counter('file_uploads_total', 'Total file uploads')
transformation_duration = Histogram('transformation_duration_seconds', 'Transformation duration')

# Error tracking
sentry_sdk.init(dsn="your-sentry-dsn")
```

### 17.8 Security Libraries

**Frontend Security:**
```typescript
// Security libraries
import { sanitize } from 'dompurify';
import { z } from 'zod';

// Input sanitization
const sanitizeInput = (input: string): string => {
  return sanitize(input, { ALLOWED_TAGS: [] });
};

// Schema validation
const FileUploadSchema = z.object({
  name: z.string().min(1).max(255),
  size: z.number().max(100 * 1024 * 1024), // 100MB limit
  type: z.string().regex(/^text\/csv|application\/csv$/),
});
```

**Backend Security:**
```python
# Security libraries
from fastapi import HTTPException, Depends
from fastapi.security import HTTPBearer
import hashlib
import secrets

# Rate limiting
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

# File validation
def validate_file(file: UploadFile) -> bool:
    # Implementation for file validation
    pass

# Input sanitization
def sanitize_input(input_str: str) -> str:
    # Implementation for input sanitization
    pass
```

This comprehensive technical implementation guide provides specific libraries and frameworks based on proven approaches from Google DataPrep and Neo4j Data Importer, ensuring a robust, scalable, and maintainable GraphPrep platform.
