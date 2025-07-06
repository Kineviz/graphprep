# Project Setup Guide - KuzuDB ETL Tool

## 1. Project Structure

```
kuzu_etl/
├── frontend/                 # React TypeScript frontend
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── stores/
│   │   ├── services/
│   │   ├── types/
│   │   ├── utils/
│   │   └── pages/
│   ├── public/
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   └── README.md
├── backend/                  # FastAPI Python backend
│   ├── app/
│   │   ├── api/
│   │   ├── core/
│   │   ├── models/
│   │   ├── services/
│   │   ├── utils/
│   │   └── main.py
│   ├── tests/
│   ├── requirements.txt
│   ├── Dockerfile
│   └── README.md
├── docs/                     # Documentation
│   ├── requirements.md
│   ├── technical-architecture.md
│   └── api-documentation.md
├── docker-compose.yml        # Development environment
├── docker-compose.prod.yml   # Production environment
├── .github/                  # CI/CD workflows
│   └── workflows/
├── .gitignore
└── README.md
```

## 2. Frontend Setup

### 2.1 Initialize React Project

```bash
# Create frontend directory
mkdir frontend
cd frontend

# Initialize with Vite
npm create vite@latest . -- --template react-ts

# Install dependencies
npm install
```

### 2.2 Install Required Dependencies

```bash
# Core dependencies
npm install react react-dom react-router-dom
npm install @tanstack/react-query @tanstack/react-table
npm install zustand immer
npm install react-dropzone papaparse
npm install reactflow
npm install date-fns lodash

# UI dependencies
npm install tailwindcss @tailwindcss/forms @tailwindcss/typography
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu
npm install @radix-ui/react-select @radix-ui/react-tabs
npm install @radix-ui/react-toast @radix-ui/react-tooltip
npm install class-variance-authority clsx tailwind-merge
npm install lucide-react

# Development dependencies
npm install -D @types/node @types/lodash
npm install -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
npm install -D eslint eslint-plugin-react eslint-plugin-react-hooks
npm install -D prettier prettier-plugin-tailwindcss
npm install -D @testing-library/react @testing-library/jest-dom
npm install -D vitest jsdom
```

### 2.3 Configuration Files

#### `vite.config.ts`
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  },
})
```

#### `tailwind.config.js`
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  prefix: "",
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

#### `tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 2.4 Initial Component Structure

#### `src/types/index.ts`
```typescript
export interface FileData {
  id: string;
  name: string;
  size: number;
  type: string;
  encoding: string;
  delimiter: string;
  rowCount: number;
  columns: ColumnInfo[];
  createdAt: Date;
  updatedAt: Date;
}

export interface ColumnInfo {
  name: string;
  type: DataType;
  confidence: number;
  sampleValues: string[];
}

export type DataType = 'string' | 'integer' | 'float' | 'boolean' | 'date';

export interface SchemaData {
  fileId: string;
  columns: ColumnSchema[];
  inferredTypes: Record<string, DataType>;
  confidenceScores: Record<string, number>;
}

export interface ColumnSchema {
  name: string;
  type: DataType;
  nullable: boolean;
  constraints: Constraint[];
}

export interface Constraint {
  type: 'unique' | 'not_null' | 'range' | 'pattern';
  value?: any;
}

export interface Transformation {
  id: string;
  type: TransformationType;
  expression: string;
  targetColumn: string;
  parameters: Record<string, any>;
  previewData: any[];
  createdAt: Date;
}

export type TransformationType = 
  | 'cast' 
  | 'string_operation' 
  | 'date_operation' 
  | 'mathematical' 
  | 'conditional' 
  | 'concatenate';

export interface NodeSchema {
  name: string;
  sourceTable: string;
  keyColumn: string;
  properties: ColumnMapping[];
  constraints: Constraint[];
}

export interface EdgeSchema {
  name: string;
  sourceNode: string;
  targetNode: string;
  sourceKey: string;
  targetKey: string;
  properties: ColumnMapping[];
}

export interface ColumnMapping {
  sourceColumn: string;
  targetProperty: string;
  transformation?: Transformation;
}

export interface ValidationError {
  id: string;
  type: 'data_quality' | 'schema_compliance' | 'graph_integrity';
  severity: 'error' | 'warning' | 'info';
  message: string;
  rowIndex?: number;
  columnName?: string;
  details?: any;
}
```

#### `src/stores/appStore.ts`
```typescript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';
import { FileData, SchemaData, Transformation, NodeSchema, EdgeSchema, ValidationError } from '@/types';

interface AppState {
  // File management
  files: FileData[];
  currentFile: string | null;
  
  // Schema and transformations
  schemas: SchemaData[];
  transformations: Transformation[];
  transformationHistory: Transformation[];
  
  // Graph schema
  nodes: NodeSchema[];
  edges: EdgeSchema[];
  
  // UI state
  currentStep: number;
  validationErrors: ValidationError[];
  isLoading: boolean;
  error: string | null;
}

interface AppActions {
  // File actions
  uploadFile: (file: File) => Promise<void>;
  setCurrentFile: (fileId: string | null) => void;
  
  // Schema actions
  updateSchema: (schema: SchemaData) => void;
  setInferredTypes: (fileId: string, types: Record<string, string>) => void;
  
  // Transformation actions
  addTransformation: (transformation: Transformation) => void;
  removeTransformation: (id: string) => void;
  undoTransformation: () => void;
  redoTransformation: () => void;
  
  // Graph schema actions
  addNode: (node: NodeSchema) => void;
  updateNode: (id: string, node: Partial<NodeSchema>) => void;
  removeNode: (id: string) => void;
  addEdge: (edge: EdgeSchema) => void;
  updateEdge: (id: string, edge: Partial<EdgeSchema>) => void;
  removeEdge: (id: string) => void;
  
  // UI actions
  setCurrentStep: (step: number) => void;
  setValidationErrors: (errors: ValidationError[]) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  
  // Utility actions
  reset: () => void;
}

type AppStore = AppState & AppActions;

const initialState: AppState = {
  files: [],
  currentFile: null,
  schemas: [],
  transformations: [],
  transformationHistory: [],
  nodes: [],
  edges: [],
  currentStep: 0,
  validationErrors: [],
  isLoading: false,
  error: null,
};

export const useAppStore = create<AppStore>()(
  immer((set, get) => ({
    ...initialState,
    
    uploadFile: async (file: File) => {
      set((state) => {
        state.isLoading = true;
        state.error = null;
      });
      
      try {
        // TODO: Implement file upload logic
        const fileData: FileData = {
          id: crypto.randomUUID(),
          name: file.name,
          size: file.size,
          type: file.type,
          encoding: 'UTF-8',
          delimiter: ',',
          rowCount: 0,
          columns: [],
          createdAt: new Date(),
          updatedAt: new Date(),
        };
        
        set((state) => {
          state.files.push(fileData);
          state.currentFile = fileData.id;
          state.isLoading = false;
        });
      } catch (error) {
        set((state) => {
          state.error = error instanceof Error ? error.message : 'Upload failed';
          state.isLoading = false;
        });
      }
    },
    
    setCurrentFile: (fileId: string | null) => {
      set((state) => {
        state.currentFile = fileId;
      });
    },
    
    updateSchema: (schema: SchemaData) => {
      set((state) => {
        const index = state.schemas.findIndex(s => s.fileId === schema.fileId);
        if (index >= 0) {
          state.schemas[index] = schema;
        } else {
          state.schemas.push(schema);
        }
      });
    },
    
    setInferredTypes: (fileId: string, types: Record<string, string>) => {
      set((state) => {
        const schema = state.schemas.find(s => s.fileId === fileId);
        if (schema) {
          schema.inferredTypes = types;
        }
      });
    },
    
    addTransformation: (transformation: Transformation) => {
      set((state) => {
        state.transformations.push(transformation);
        state.transformationHistory.push(transformation);
      });
    },
    
    removeTransformation: (id: string) => {
      set((state) => {
        state.transformations = state.transformations.filter(t => t.id !== id);
      });
    },
    
    undoTransformation: () => {
      set((state) => {
        if (state.transformationHistory.length > 0) {
          state.transformationHistory.pop();
          // TODO: Implement undo logic
        }
      });
    },
    
    redoTransformation: () => {
      // TODO: Implement redo logic
    },
    
    addNode: (node: NodeSchema) => {
      set((state) => {
        state.nodes.push(node);
      });
    },
    
    updateNode: (id: string, node: Partial<NodeSchema>) => {
      set((state) => {
        const index = state.nodes.findIndex(n => n.name === id);
        if (index >= 0) {
          state.nodes[index] = { ...state.nodes[index], ...node };
        }
      });
    },
    
    removeNode: (id: string) => {
      set((state) => {
        state.nodes = state.nodes.filter(n => n.name !== id);
        // Remove related edges
        state.edges = state.edges.filter(e => e.sourceNode !== id && e.targetNode !== id);
      });
    },
    
    addEdge: (edge: EdgeSchema) => {
      set((state) => {
        state.edges.push(edge);
      });
    },
    
    updateEdge: (id: string, edge: Partial<EdgeSchema>) => {
      set((state) => {
        const index = state.edges.findIndex(e => e.name === id);
        if (index >= 0) {
          state.edges[index] = { ...state.edges[index], ...edge };
        }
      });
    },
    
    removeEdge: (id: string) => {
      set((state) => {
        state.edges = state.edges.filter(e => e.name !== id);
      });
    },
    
    setCurrentStep: (step: number) => {
      set((state) => {
        state.currentStep = step;
      });
    },
    
    setValidationErrors: (errors: ValidationError[]) => {
      set((state) => {
        state.validationErrors = errors;
      });
    },
    
    setLoading: (loading: boolean) => {
      set((state) => {
        state.isLoading = loading;
      });
    },
    
    setError: (error: string | null) => {
      set((state) => {
        state.error = error;
      });
    },
    
    reset: () => {
      set(initialState);
    },
  }))
);
```

## 3. Backend Setup

### 3.1 Initialize Python Project

```bash
# Create backend directory
mkdir backend
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 3.2 Requirements File

#### `requirements.txt`
```
# FastAPI and web framework
fastapi==0.104.1
uvicorn[standard]==0.24.0
python-multipart==0.0.6

# Database
sqlalchemy==2.0.23
alembic==1.12.1
psycopg2-binary==2.9.9

# Data processing
pandas==2.1.3
numpy==1.25.2
python-dateutil==2.8.2

# Validation and serialization
pydantic==2.5.0
pydantic-settings==2.1.0

# File processing
python-magic==0.4.27
chardet==5.2.0

# Utilities
python-dotenv==1.0.0
structlog==23.2.0
httpx==0.25.2

# Development and testing
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0
black==23.11.0
isort==5.12.0
flake8==6.1.0
mypy==1.7.1
```

### 3.3 Project Structure

#### `app/main.py`
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from contextlib import asynccontextmanager
import structlog

from app.core.config import settings
from app.api.v1.api import api_router
from app.core.database import engine, Base

logger = structlog.get_logger()

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    logger.info("Starting up ETL application")
    
    # Create database tables
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    yield
    
    # Shutdown
    logger.info("Shutting down ETL application")

app = FastAPI(
    title="KuzuDB ETL Tool",
    description="A web-based ETL application for KuzuDB",
    version="1.0.0",
    lifespan=lifespan,
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include API router
app.include_router(api_router, prefix="/api/v1")

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "version": "1.0.0",
        "timestamp": "2024-01-01T00:00:00Z"
    }

@app.exception_handler(Exception)
async def global_exception_handler(request, exc):
    logger.error("Unhandled exception", exc_info=exc)
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"}
    )

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### `app/core/config.py`
```python
from pydantic_settings import BaseSettings
from typing import List
import os

class Settings(BaseSettings):
    # Application
    APP_NAME: str = "KuzuDB ETL Tool"
    DEBUG: bool = False
    VERSION: str = "1.0.0"
    
    # Server
    HOST: str = "0.0.0.0"
    PORT: int = 8000
    
    # CORS
    ALLOWED_ORIGINS: List[str] = [
        "http://localhost:3000",
        "http://127.0.0.1:3000",
    ]
    
    # Database
    DATABASE_URL: str = "sqlite:///./etl.db"
    
    # File upload
    MAX_FILE_SIZE: int = 100 * 1024 * 1024  # 100MB
    UPLOAD_DIR: str = "./uploads"
    ALLOWED_FILE_TYPES: List[str] = [".csv", ".tsv", ".txt"]
    
    # Processing
    MAX_ROWS_FOR_PREVIEW: int = 1000
    MAX_ROWS_FOR_INFERENCE: int = 10000
    CHUNK_SIZE: int = 10000
    
    # Security
    SECRET_KEY: str = "your-secret-key-here"
    
    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()

# Ensure upload directory exists
os.makedirs(settings.UPLOAD_DIR, exist_ok=True)
```

#### `app/core/database.py`
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import MetaData
from app.core.config import settings

# Create async engine
engine = create_async_engine(
    settings.DATABASE_URL.replace("sqlite:///", "sqlite+aiosqlite:///"),
    echo=settings.DEBUG,
    future=True,
)

# Create async session factory
AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

# Create base class for models
Base = declarative_base()

# Dependency to get database session
async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

### 3.4 API Structure

#### `app/api/v1/api.py`
```python
from fastapi import APIRouter
from app.api.v1.endpoints import files, schema, transform, graph, export

api_router = APIRouter()

api_router.include_router(files.router, prefix="/files", tags=["files"])
api_router.include_router(schema.router, prefix="/schema", tags=["schema"])
api_router.include_router(transform.router, prefix="/transform", tags=["transform"])
api_router.include_router(graph.router, prefix="/graph", tags=["graph"])
api_router.include_router(export.router, prefix="/export", tags=["export"])
```

#### `app/api/v1/endpoints/files.py`
```python
from fastapi import APIRouter, UploadFile, File, HTTPException, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import get_db
from app.services.file_service import FileService
from app.models.file import FileData
from app.schemas.file import FileResponse
import structlog

logger = structlog.get_logger()
router = APIRouter()

@router.post("/upload", response_model=FileResponse)
async def upload_file(
    file: UploadFile = File(...),
    db: AsyncSession = Depends(get_db)
):
    """Upload a CSV file for processing"""
    try:
        file_service = FileService(db)
        file_data = await file_service.upload_file(file)
        logger.info("File uploaded successfully", file_id=file_data.id, file_name=file.name)
        return FileResponse.from_orm(file_data)
    except Exception as e:
        logger.error("File upload failed", error=str(e))
        raise HTTPException(status_code=400, detail=str(e))

@router.get("/{file_id}", response_model=FileResponse)
async def get_file(
    file_id: str,
    db: AsyncSession = Depends(get_db)
):
    """Get file information"""
    try:
        file_service = FileService(db)
        file_data = await file_service.get_file(file_id)
        if not file_data:
            raise HTTPException(status_code=404, detail="File not found")
        return FileResponse.from_orm(file_data)
    except HTTPException:
        raise
    except Exception as e:
        logger.error("Failed to get file", file_id=file_id, error=str(e))
        raise HTTPException(status_code=500, detail="Internal server error")

@router.get("/{file_id}/preview")
async def get_file_preview(
    file_id: str,
    rows: int = 100,
    db: AsyncSession = Depends(get_db)
):
    """Get a preview of the file data"""
    try:
        file_service = FileService(db)
        preview_data = await file_service.get_preview(file_id, rows)
        return preview_data
    except Exception as e:
        logger.error("Failed to get file preview", file_id=file_id, error=str(e))
        raise HTTPException(status_code=500, detail="Internal server error")
```

## 4. Docker Setup

### 4.1 Development Docker Compose

#### `docker-compose.yml`
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - VITE_API_URL=http://localhost:8000
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
      - ./uploads:/app/uploads
    environment:
      - DATABASE_URL=sqlite:///./etl.db
      - DEBUG=true
      - ALLOWED_ORIGINS=http://localhost:3000
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  database:
    image: postgres:15
    environment:
      - POSTGRES_DB=etl_dev
      - POSTGRES_USER=etl_user
      - POSTGRES_PASSWORD=etl_pass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 4.2 Frontend Dockerfile

#### `frontend/Dockerfile.dev`
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start development server
CMD ["npm", "run", "dev"]
```

### 4.3 Backend Dockerfile

#### `backend/Dockerfile.dev`
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code
COPY . .

# Expose port
EXPOSE 8000

# Start development server
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

## 5. Development Workflow

### 5.1 Getting Started

```bash
# Clone repository
git clone <repository-url>
cd kuzu_etl

# Start development environment
docker-compose up -d

# Access applications
# Frontend: http://localhost:3000
# Backend: http://localhost:8000
# API Docs: http://localhost:8000/docs
```

### 5.2 Development Commands

```bash
# Frontend development
cd frontend
npm run dev          # Start development server
npm run build        # Build for production
npm run test         # Run tests
npm run lint         # Run linter
npm run format       # Format code

# Backend development
cd backend
uvicorn app.main:app --reload  # Start development server
pytest                          # Run tests
black .                        # Format code
isort .                        # Sort imports
flake8                         # Run linter
mypy .                         # Type checking
```

### 5.3 Environment Variables

#### `.env.example`
```bash
# Application
DEBUG=true
SECRET_KEY=your-secret-key-here

# Database
DATABASE_URL=sqlite:///./etl.db

# File upload
MAX_FILE_SIZE=104857600
UPLOAD_DIR=./uploads

# CORS
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
```

## 6. Testing Setup

### 6.1 Frontend Testing

#### `frontend/src/test/setup.ts`
```typescript
import '@testing-library/jest-dom';
import { vi } from 'vitest';

// Mock ResizeObserver
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));

// Mock IntersectionObserver
global.IntersectionObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));
```

### 6.2 Backend Testing

#### `backend/tests/conftest.py`
```python
import pytest
import asyncio
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from app.core.database import Base
from app.core.config import settings

@pytest.fixture(scope="session")
def event_loop():
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture
async def test_db():
    # Create test database
    test_engine = create_async_engine(
        "sqlite+aiosqlite:///./test.db",
        echo=False,
    )
    
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    TestSessionLocal = sessionmaker(
        test_engine, class_=AsyncSession, expire_on_commit=False
    )
    
    async with TestSessionLocal() as session:
        yield session
    
    # Cleanup
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
```

## 7. Deployment Preparation

### 7.1 Production Docker Compose

#### `docker-compose.prod.yml`
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    ports:
      - "80:80"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://etl_user:etl_pass@database:5432/etl_prod
      - DEBUG=false
    depends_on:
      - database

  database:
    image: postgres:15
    environment:
      - POSTGRES_DB=etl_prod
      - POSTGRES_USER=etl_user
      - POSTGRES_PASSWORD=etl_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 7.2 Production Dockerfiles

#### `frontend/Dockerfile.prod`
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### `backend/Dockerfile.prod`
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy source code
COPY . .

# Create non-root user
RUN useradd -m -u 1000 app && chown -R app:app /app
USER app

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

This setup guide provides a comprehensive foundation for developing the KuzuDB ETL tool with modern best practices, proper separation of concerns, and scalable architecture. 