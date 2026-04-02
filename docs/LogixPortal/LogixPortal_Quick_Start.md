# LogixPortal - Guía Rápida de Inicio
## ¡Comienza tu proyecto en 1 hora!

---

## 📋 Pre-requisitos

Antes de comenzar, asegúrate de tener instalado:

```bash
# Dart SDK 3.x
dart --version

# Flutter 3.x
flutter --version

# PostgreSQL 15+
psql --version

# Git
git --version

# Docker (opcional pero recomendado)
docker --version
```

---

## 🚀 Setup en 10 Pasos (60 minutos)

### Paso 1: Instalar Serverpod CLI (5 min)

```bash
dart pub global activate serverpod_cli
serverpod --version
```

### Paso 2: Crear Proyecto Serverpod (5 min)

```bash
serverpod create logixportal
cd logixportal
```

Estructura generada:
```
logixportal/
├── logixportal_server/      # Backend Dart
├── logixportal_client/      # Cliente generado
├── logixportal_flutter/     # Frontend Flutter
└── docker-compose.yaml
```

### Paso 3: Configurar PostgreSQL con PostGIS (10 min)

Editar `docker-compose.yaml`:

```yaml
services:
  postgres:
    image: postgis/postgis:15-3.3
    ports:
      - '5432:5432'
    environment:
      POSTGRES_USER: postgres
      POSTGRES_DB: logixportal
      POSTGRES_PASSWORD: your_password_here
    volumes:
      - logixportal_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'

volumes:
  logixportal_data:
```

Iniciar servicios:

```bash
docker-compose up -d
```

### Paso 4: Configurar Extensiones PostgreSQL (5 min)

```bash
# Conectar a PostgreSQL
docker exec -it logixportal-postgres-1 psql -U postgres -d logixportal

# Dentro de psql:
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "postgis";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

# Verificar
\dx

# Salir
\q
```

### Paso 5: Definir Primera Entidad en serverpod.yaml (10 min)

Editar `logixportal_server/lib/src/protocol/company.spy.yaml`:

```yaml
class: Company
table: companies
fields:
  id:
    type: UuidValue
    database: uuid PRIMARY KEY DEFAULT uuid_generate_v4()
  companyCode:
    type: String
    database: VARCHAR(20) UNIQUE NOT NULL
  businessName:
    type: String
    database: VARCHAR(300) NOT NULL
  taxId:
    type: String
    database: VARCHAR(20) UNIQUE NOT NULL
  subscriptionTier:
    type: String
    database: VARCHAR(20) DEFAULT 'BASIC'
  activeModules:
    type: List<String>
    database: JSONB DEFAULT '[]'::jsonb
  isActive:
    type: bool
    database: BOOLEAN DEFAULT true
  createdAt:
    type: DateTime
    database: TIMESTAMP DEFAULT NOW()
  updatedAt:
    type: DateTime
    database: TIMESTAMP DEFAULT NOW()
indexes:
  - name: idx_companies_tax_id
    fields: [taxId]
    unique: true
```

### Paso 6: Generar Código y Migraciones (5 min)

```bash
cd logixportal_server
serverpod generate
serverpod create-migration
```

Esto genera:
- Clases Dart en `lib/src/generated/`
- SQL de migración en `migrations/`

### Paso 7: Aplicar Migración (2 min)

```bash
# Editar config/development.yaml con tus credenciales de PostgreSQL
# Luego:
dart bin/main.dart --apply-migrations
```

### Paso 8: Crear Primer Endpoint (10 min)

Crear `logixportal_server/lib/src/endpoints/company_endpoint.dart`:

```dart
import 'package:serverpod/serverpod.dart';
import '../generated/protocol.dart';

class CompanyEndpoint extends Endpoint {
  
  Future<Company> createCompany(
    Session session,
    String companyCode,
    String businessName,
    String taxId,
  ) async {
    final company = Company(
      companyCode: companyCode,
      businessName: businessName,
      taxId: taxId,
      subscriptionTier: 'BASIC',
      activeModules: [],
      isActive: true,
      createdAt: DateTime.now(),
      updatedAt: DateTime.now(),
    );

    return await Company.db.insertRow(session, company);
  }

  Future<List<Company>> getAllCompanies(Session session) async {
    return await Company.db.find(
      session,
      where: (t) => t.isActive.equals(true),
    );
  }

  Future<Company?> getCompanyById(Session session, UuidValue id) async {
    return await Company.db.findById(session, id);
  }
}
```

Registrar endpoint en `logixportal_server/lib/src/server.dart`:

```dart
import 'endpoints/company_endpoint.dart';

// En el método de configuración:
endpoints.register(CompanyEndpoint());
```

### Paso 9: Iniciar Servidor (2 min)

```bash
cd logixportal_server
dart bin/main.dart
```

Output esperado:
```
Serverpod server starting...
Listening on http://localhost:8080
```

### Paso 10: Probar con Flutter (6 min)

Editar `logixportal_flutter/lib/main.dart`:

```dart
import 'package:flutter/material.dart';
import 'package:logixportal_client/logixportal_client.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final client = Client('http://localhost:8080/');

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'LogixPortal',
      home: CompanyListScreen(client: client),
    );
  }
}

class CompanyListScreen extends StatefulWidget {
  final Client client;
  
  CompanyListScreen({required this.client});

  @override
  _CompanyListScreenState createState() => _CompanyListScreenState();
}

class _CompanyListScreenState extends State<CompanyListScreen> {
  List<Company>? companies;
  bool loading = true;

  @override
  void initState() {
    super.initState();
    _loadCompanies();
  }

  Future<void> _loadCompanies() async {
    try {
      final result = await widget.client.company.getAllCompanies();
      setState(() {
        companies = result;
        loading = false;
      });
    } catch (e) {
      print('Error loading companies: $e');
      setState(() {
        loading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('LogixPortal - Empresas'),
      ),
      body: loading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              itemCount: companies?.length ?? 0,
              itemBuilder: (context, index) {
                final company = companies![index];
                return ListTile(
                  title: Text(company.businessName),
                  subtitle: Text('RUC: ${company.taxId}'),
                  trailing: Text(company.subscriptionTier),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // Navegar a pantalla de crear empresa
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

Ejecutar Flutter:

```bash
cd logixportal_flutter
flutter run -d chrome
```

---

## ✅ Verificación

Si todo funcionó correctamente, deberías ver:

1. ✅ Servidor Serverpod corriendo en http://localhost:8080
2. ✅ PostgreSQL con PostGIS activo
3. ✅ Tabla `companies` creada
4. ✅ Flutter app mostrando lista (vacía inicialmente)

---

## 🎯 Próximos Pasos

Ahora que tienes el setup básico:

1. **Semana 1**: Implementar autenticación (User + Role entities)
2. **Semana 2**: Agregar entidad Product
3. **Semana 3**: Implementar CRUD de productos en Flutter
4. **Semana 4**: Agregar Warehouse con PostGIS para geolocalización

Consulta el **Prompt Maestro Profesional** para el roadmap completo.

---

## 🆘 Troubleshooting

### Error: "Could not connect to database"

```bash
# Verificar que PostgreSQL esté corriendo
docker ps | grep postgres

# Ver logs
docker logs logixportal-postgres-1
```

### Error: "Extension postgis not found"

```bash
# Reinstalar imagen PostGIS
docker-compose down
docker-compose up -d
```

### Error de migración

```bash
# Resetear base de datos (⚠️ solo en desarrollo)
docker exec -it logixportal-postgres-1 psql -U postgres -c "DROP DATABASE logixportal; CREATE DATABASE logixportal;"
dart bin/main.dart --apply-migrations
```

---

## 📚 Recursos Adicionales

- **Serverpod Docs**: https://docs.serverpod.dev/
- **Flutter Docs**: https://docs.flutter.dev/
- **PostGIS Tutorial**: https://postgis.net/workshops/postgis-intro/
- **Riverpod**: https://riverpod.dev/

---

¡Felicidades! Ya tienes LogixPortal corriendo localmente. 🎉
