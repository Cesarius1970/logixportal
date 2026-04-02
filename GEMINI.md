# LogixPortal - Project Overview & Instructions

LogixPortal is an incremental ERP system specialized in Peruvian foreign trade (imports/exports), built with a modern Dart-centric stack.

## 🏗 Project Architecture

The project is structured as a Dart workspace containing three main packages:

1.  **`logixportal_server`**: The backend server built with Serverpod 3. It handles business logic, database interactions, and API endpoints.
2.  **`logixportal_client`**: Automatically generated client-side SDK. It provides type-safe communication between the server and the frontend.
3.  **`logixportal_flutter`**: The frontend application built with Flutter Web, following Material Design 3 and using Riverpod for state management.

### Key Technologies
- **Language**: Dart (Full-stack)
- **Framework**: [Serverpod 3.4.5](https://serverpod.dev)
- **Frontend**: Flutter Web (Targeting Chrome/Edge)
- **Database**: PostgreSQL (Primary) + Redis (Cache)
- **State Management**: Riverpod 3
- **Infrastructure**: Docker for local development

## 🚀 Getting Started

### Prerequisites
- Dart SDK (^3.11.0)
- Flutter SDK (^3.32.0)
- Serverpod CLI (`dart pub global activate serverpod_cli`)
- Docker & Docker Compose

### Development Workflow
1.  **Database**: Start the local database using Docker.
    ```bash
    cd logixportal_server
    docker-compose up -d
    ```
2.  **Code Generation**: Always run generate after modifying protocol files (`.yaml`) in the server.
    ```bash
    # From root or logixportal_server
    serverpod generate
    ```
3.  **Run Server**:
    ```bash
    cd logixportal_server
    dart bin/main.dart --apply-migrations
    ```
4.  **Run Frontend**:
    ```bash
    cd logixportal_flutter
    flutter run -d chrome
    ```

## 🛠 Building & Deployment

- **Build Flutter Web**:
  The server package includes a script to build and deploy the Flutter app into the server's static directory.
  ```bash
  cd logixportal_server
  # On Linux/macOS
  dart pub run serverpod:flutter_build posix
  ```

## 📜 Development Conventions

- **Incremental Growth**: The project follows an 8-phase plan (Fase 0 to Fase 7). Current focus is **Fase 0 (Walking Skeleton)**.
- **Clean Architecture**: Maintain a clear separation between Data, Domain, and Presentation layers in both frontend and backend.
- **Multi-tenancy**: Most database entities must include a `tenantId` to support multiple organizations.
- **Type Safety**: Leverage Serverpod's generated models to ensure type safety across the entire stack.
- **Documentation**: Use ADRs (Architecture Decision Records) for significant changes. Refer to `docs/LogixPortal/` for detailed blueprints.

## 📂 Key Directories
- `logixportal_server/lib/src/endpoints/`: API definitions.
- `logixportal_server/lib/src/protocol/`: Data model definitions (`.yaml`).
- `logixportal_flutter/lib/features/`: Feature-based frontend modules.
- `docs/LogixPortal/`: Comprehensive design and planning documents.

## 🧪 Testing
- **Server**: Use `serverpod_test` for integration tests in `logixportal_server/test/`.
- **Flutter**: Use standard `flutter_test` in `logixportal_flutter/test/`.
