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
- **Neo4j**: CSV format compatible with Neo4j's LOAD CSV command
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
