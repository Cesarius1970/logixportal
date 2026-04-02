# Arquitectura Técnica Detallada - LogixPortal
## Stack: Serverpod 3 + Flutter Web + PostgreSQL

---

## DECISIONES ARQUITECTÓNICAS CLAVE

### ¿Por qué Serverpod 3?

**Ventajas:**
- ✅ **Full-stack Dart**: Un solo lenguaje frontend/backend
- ✅ **Type-safety extremo**: Models compartidos entre cliente y servidor
- ✅ **ORM integrado**: Generación automática de queries
- ✅ **WebSockets nativos**: Real-time sin configuración compleja
- ✅ **Caché integrado**: Redis out-of-the-box
- ✅ **Auth incluido**: OAuth, JWT, sessions
- ✅ **Migraciones de BD**: Versionadas y automáticas

**Desventajas a considerar:**
- ⚠️ Ecosistema más pequeño que Node.js/Python
- ⚠️ Menos recursos/tutoriales disponibles
- ⚠️ Equipo necesita aprender Serverpod

**Decisión:** Vale la pena por productividad a largo plazo

---

## ARQUITECTURA DE ALTO NIVEL

```
┌───────────────────────────────────────────────────────┐
│                   USUARIO FINAL                        │
│              (Navegador Web Chrome/Edge)              │
└───────────────────┬───────────────────────────────────┘
                    │ HTTPS
┌───────────────────▼───────────────────────────────────┐
│              FLUTTER WEB APPLICATION                   │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Presentation │  │    Domain    │  │    Data    │ │
│  │   (Widgets)  │  │  (Entities)  │  │(Repository)│ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
│                                                        │
│  State Management: Riverpod 3                         │
│  HTTP Client: Serverpod Client SDK                    │
└───────────────────┬───────────────────────────────────┘
                    │ HTTPS/WSS
                    │ (Serverpod Protocol)
┌───────────────────▼───────────────────────────────────┐
│                 NGINX REVERSE PROXY                    │
│         SSL Termination + Load Balancing              │
└───────────────────┬───────────────────────────────────┘
                    │
┌───────────────────▼───────────────────────────────────┐
│              SERVERPOD 3 BACKEND                       │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │  Endpoints   │  │   Services   │  │    Models  │ │
│  │  (API Layer) │  │ (Business)   │  │(Shared DTO)│ │
│  └──────────────┘  └──────────────┘  └────────────┘ │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐                  │
│  │ Repositories │  │  Middleware  │                  │
│  │  (DB Access) │  │ (Auth/Audit) │                  │
│  └──────────────┘  └──────────────┘                  │
└───────┬────────────────┬──────────────┬──────────────┘
        │                │              │
┌───────▼────┐  ┌───────▼─────┐  ┌────▼────────────┐
│ PostgreSQL │  │    Redis    │  │   RabbitMQ      │
│  (Primary) │  │   (Cache)   │  │ (Message Queue) │
└────────────┘  └─────────────┘  └─────────────────┘
```

---

## ESTRUCTURA DE PROYECTO COMPLETA

### Monorepo Structure

```
logixportal/
├── logixportal_server/              # Serverpod Backend
│   ├── lib/
│   │   ├── src/
│   │   │   ├── endpoints/           # API Endpoints
│   │   │   │   ├── auth_endpoint.dart
│   │   │   │   ├── procurement_endpoint.dart
│   │   │   │   ├── inventory_endpoint.dart
│   │   │   │   └── ...
│   │   │   ├── services/            # Business Logic
│   │   │   │   ├── procurement_service.dart
│   │   │   │   ├── sunat_service.dart
│   │   │   │   └── ...
│   │   │   ├── repositories/        # Data Access
│   │   │   │   ├── base_repository.dart
│   │   │   │   ├── product_repository.dart
│   │   │   │   └── ...
│   │   │   ├── middleware/          # Request Interceptors
│   │   │   │   ├── auth_middleware.dart
│   │   │   │   ├── tenant_middleware.dart
│   │   │   │   └── audit_middleware.dart
│   │   │   ├── utils/               # Utilities
│   │   │   │   ├── validators.dart
│   │   │   │   ├── encryption.dart
│   │   │   │   └── pdf_generator.dart
│   │   │   └── generated/           # Serverpod Generated Code
│   │   │       ├── protocol.dart
│   │   │       └── endpoints.dart
│   │   └── server.dart              # Server Entry Point
│   ├── config/
│   │   ├── development.yaml
│   │   ├── production.yaml
│   │   └── passwords.yaml (git-ignored)
│   ├── migrations/                  # DB Migrations
│   │   ├── 001_initial_schema.sql
│   │   ├── 002_add_imports.sql
│   │   └── ...
│   ├── test/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   └── pubspec.yaml
│
├── logixportal_client/              # Serverpod Client SDK (Generated)
│   ├── lib/
│   │   └── src/
│   │       ├── protocol/
│   │       │   ├── product.dart
│   │       │   ├── purchase_order.dart
│   │       │   └── ...
│   │       └── client.dart
│   └── pubspec.yaml
│
├── logixportal_flutter/             # Flutter Web App
│   ├── lib/
│   │   ├── main.dart
│   │   ├── core/
│   │   │   ├── config/
│   │   │   │   ├── app_config.dart
│   │   │   │   ├── environment.dart
│   │   │   │   └── routes.dart
│   │   │   ├── theme/
│   │   │   │   ├── app_theme.dart
│   │   │   │   ├── color_schemes.dart
│   │   │   │   └── typography.dart
│   │   │   ├── network/
│   │   │   │   ├── serverpod_client.dart
│   │   │   │   └── error_handler.dart
│   │   │   └── utils/
│   │   │       ├── formatters.dart
│   │   │       ├── validators.dart
│   │   │       └── extensions.dart
│   │   ├── features/
│   │   │   ├── auth/
│   │   │   │   ├── data/
│   │   │   │   │   ├── repositories/
│   │   │   │   │   │   └── auth_repository.dart
│   │   │   │   │   └── models/
│   │   │   │   ├── domain/
│   │   │   │   │   ├── entities/
│   │   │   │   │   └── usecases/
│   │   │   │   └── presentation/
│   │   │   │       ├── providers/
│   │   │   │       │   └── auth_provider.dart
│   │   │   │       ├── screens/
│   │   │   │       │   ├── login_screen.dart
│   │   │   │       │   └── register_screen.dart
│   │   │   │       └── widgets/
│   │   │   ├── procurement/
│   │   │   │   ├── data/
│   │   │   │   ├── domain/
│   │   │   │   └── presentation/
│   │   │   ├── inventory/
│   │   │   ├── sales/
│   │   │   └── ...
│   │   └── shared/
│   │       ├── widgets/
│   │       │   ├── app_bar.dart
│   │       │   ├── drawer.dart
│   │       │   ├── data_table.dart
│   │       │   └── ...
│   │       └── providers/
│   ├── web/
│   │   ├── index.html
│   │   ├── manifest.json
│   │   └── favicon.png
│   ├── test/
│   └── pubspec.yaml
│
├── docker/
│   ├── Dockerfile.server
│   ├── Dockerfile.flutter
│   ├── docker-compose.yml
│   └── docker-compose.prod.yml
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
└── README.md
```

---

## SERVERPOD 3: CONCEPTOS CLAVE

### 1. Protocol Definition (Shared Models)

```dart
// logixportal_server/lib/src/protocol/product.yaml
class: Product
table: products
fields:
  id: int?, database, primary
  tenantId: int, database
  productCode: String, database
  name: String, database
  description: String?
  categoryId: int?
  unitOfMeasureId: int, database
  standardCost: double?
  salePrice: double?
  active: bool, database
  createdAt: DateTime, database
  updatedAt: DateTime, database
```

**Esto genera automáticamente:**
- `Product` class en servidor
- `Product` class en cliente (mismo código)
- Database queries
- Serialization/deserialization

### 2. Endpoints (API Layer)

```dart
// logixportal_server/lib/src/endpoints/procurement_endpoint.dart
import 'package:serverpod/serverpod.dart';

class ProcurementEndpoint extends Endpoint {
  
  /// Create a new quotation request
  Future<QuotationRequest> createQuotationRequest(
    Session session,
    QuotationRequestInput input,
  ) async {
    // 1. Validar autenticación
    if (!session.isUserSignedIn) {
      throw UnauthorizedException('Usuario no autenticado');
    }
    
    // 2. Validar tenant (multi-tenancy)
    final tenantId = session.auth.tenantId;
    
    // 3. Validar permisos
    if (!await hasPermission(session, 'procurement.create.quotation_request')) {
      throw ForbiddenException('Sin permisos');
    }
    
    // 4. Validar datos de entrada
    _validateQuotationRequestInput(input);
    
    // 5. Crear entidad
    final qr = QuotationRequest(
      tenantId: tenantId,
      requestNumber: await _generateRequestNumber(session),
      requestDate: DateTime.now(),
      requestedBy: session.auth.userId,
      priority: input.priority,
      status: 'DRAFT',
    );
    
    // 6. Guardar en BD (transaccional)
    await session.db.transaction((transaction) async {
      // Insertar quotation request
      final inserted = await QuotationRequest.db.insertRow(
        session,
        qr,
      );
      
      // Insertar items
      for (var item in input.items) {
        await QuotationRequestItem.db.insertRow(
          session,
          QuotationRequestItem(
            quotationRequestId: inserted.id!,
            lineNumber: item.lineNumber,
            productId: item.productId,
            quantity: item.quantity,
          ),
        );
      }
      
      // Auditoría
      await _auditLog(session, 'CREATE_QR', inserted.id);
    });
    
    return qr;
  }
  
  /// List quotation requests with filters
  Future<List<QuotationRequest>> listQuotationRequests(
    Session session, {
    String? status,
    int? requestedBy,
    DateTime? dateFrom,
    DateTime? dateTo,
    int offset = 0,
    int limit = 50,
  }) async {
    final tenantId = session.auth.tenantId;
    
    // Build query
    var query = QuotationRequest.db.find(
      where: (t) => t.tenantId.equals(tenantId),
    );
    
    // Apply filters
    if (status != null) {
      query = query.where((t) => t.status.equals(status));
    }
    
    if (requestedBy != null) {
      query = query.where((t) => t.requestedBy.equals(requestedBy));
    }
    
    if (dateFrom != null) {
      query = query.where((t) => t.requestDate >= dateFrom);
    }
    
    return await query
        .orderBy((t) => t.requestDate, orderDescending: true)
        .offset(offset)
        .limit(limit)
        .toList();
  }
}
```

### 3. Services (Business Logic)

```dart
// logixportal_server/lib/src/services/procurement_service.dart
class ProcurementService {
  
  /// Convert approved quotation to purchase order
  Future<PurchaseOrder> convertQuotationToPO(
    Session session,
    int quotationId,
  ) async {
    // 1. Obtener cotización
    final quotation = await Quotation.db.findById(session, quotationId);
    if (quotation == null) {
      throw NotFoundException('Cotización no encontrada');
    }
    
    // 2. Validar estado
    if (quotation.status != 'APPROVED') {
      throw ValidationException('Solo se pueden convertir cotizaciones aprobadas');
    }
    
    // 3. Obtener items de la cotización
    final quotationItems = await QuotationItem.db.find(
      where: (t) => t.quotationId.equals(quotationId),
    );
    
    // 4. Crear PO
    final po = PurchaseOrder(
      tenantId: quotation.tenantId,
      poNumber: await _generatePONumber(session),
      quotationId: quotationId,
      supplierId: quotation.supplierId,
      currencyCode: quotation.currencyCode,
      incoterm: quotation.incoterm,
      status: 'DRAFT',
      subtotal: quotation.subtotal,
      totalAmount: quotation.totalAmount,
      createdBy: session.auth.userId,
    );
    
    // 5. Guardar en transacción
    return await session.db.transaction((tx) async {
      final insertedPO = await PurchaseOrder.db.insertRow(session, po);
      
      // Crear items de PO desde items de cotización
      for (var qItem in quotationItems) {
        await PurchaseOrderItem.db.insertRow(
          session,
          PurchaseOrderItem(
            poId: insertedPO.id!,
            lineNumber: qItem.lineNumber,
            productId: qItem.productId,
            quantity: qItem.quantity,
            unitPrice: qItem.unitPrice,
          ),
        );
      }
      
      // Actualizar estado de cotización
      quotation.status = 'CONVERTED';
      await Quotation.db.updateRow(session, quotation);
      
      // Evento: PO creada
      await session.messages.postMessage(
        'purchase_order.created',
        {'poId': insertedPO.id},
      );
      
      return insertedPO;
    });
  }
  
  /// Calculate landed cost for import
  Future<LandedCost> calculateLandedCost(
    Session session,
    int poId,
  ) async {
    final po = await PurchaseOrder.db.findById(session, poId);
    if (po == null) throw NotFoundException('PO no encontrada');
    
    final shipment = await Shipment.db.findFirst(
      where: (t) => t.poId.equals(poId),
    );
    
    final customsDecl = shipment != null
        ? await CustomsDeclaration.db.findFirst(
            where: (t) => t.shipmentId.equals(shipment.id!),
          )
        : null;
    
    // Calcular costos
    final fob = po.subtotal;
    final freight = po.freight ?? 0.0;
    final insurance = po.insurance ?? 0.0;
    final cif = fob + freight + insurance;
    
    final tariff = customsDecl?.tariffAmount ?? 0.0;
    final igv = customsDecl?.igvAmount ?? 0.0;
    
    final total = cif + tariff + igv;
    
    final landedCost = LandedCost(
      tenantId: po.tenantId,
      poId: poId,
      shipmentId: shipment?.id,
      customsDeclarationId: customsDecl?.id,
      fobAmount: fob,
      freightAmount: freight,
      insuranceAmount: insurance,
      tariffAmount: tariff,
      igvAmount: igv,
      totalLandedCost: total,
    );
    
    return await LandedCost.db.insertRow(session, landedCost);
  }
}
```

### 4. Repositories (Optional, para lógica compleja)

```dart
// logixportal_server/lib/src/repositories/product_repository.dart
class ProductRepository {
  
  Future<List<Product>> searchProducts(
    Session session,
    String searchTerm, {
    int? categoryId,
    bool activeOnly = true,
  }) async {
    final tenantId = session.auth.tenantId;
    
    // Query con full-text search
    final results = await session.db.query(
      '''
      SELECT * FROM products
      WHERE tenant_id = @tenantId
        AND active = @active
        AND (
          product_code ILIKE @search
          OR name ILIKE @search
        )
        AND (@categoryId IS NULL OR category_id = @categoryId)
      ORDER BY name
      LIMIT 50
      ''',
      {
        'tenantId': tenantId,
        'active': activeOnly,
        'search': '%$searchTerm%',
        'categoryId': categoryId,
      },
    );
    
    return results.map((row) => Product.fromJson(row.toColumnMap())).toList();
  }
}
```

---

## FLUTTER WEB: ARQUITECTURA FRONTEND

### 1. Riverpod 3 Providers

```dart
// logixportal_flutter/lib/features/procurement/presentation/providers/quotation_request_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:logixportal_client/logixportal_client.dart';

part 'quotation_request_provider.g.dart';

/// Provider del cliente Serverpod
@riverpod
Client serverpodClient(ServerpodClientRef ref) {
  return Client('https://api.logixportal.com');
}

/// Provider de lista de quotation requests
@riverpod
class QuotationRequestList extends _$QuotationRequestList {
  @override
  Future<List<QuotationRequest>> build({
    String? status,
    int? requestedBy,
  }) async {
    final client = ref.watch(serverpodClientProvider);
    
    return await client.procurement.listQuotationRequests(
      status: status,
      requestedBy: requestedBy,
    );
  }
  
  /// Create new quotation request
  Future<void> create(QuotationRequestInput input) async {
    state = const AsyncValue.loading();
    
    state = await AsyncValue.guard(() async {
      final client = ref.read(serverpodClientProvider);
      final newQR = await client.procurement.createQuotationRequest(input);
      
      // Actualizar lista
      final currentList = state.value ?? [];
      return [newQR, ...currentList];
    });
  }
}

/// Provider de filtro de quotation requests
@riverpod
class QuotationRequestFilter extends _$QuotationRequestFilter {
  @override
  QuotationRequestFilterState build() {
    return const QuotationRequestFilterState(
      status: null,
      requestedBy: null,
    );
  }
  
  void setStatus(String? status) {
    state = state.copyWith(status: status);
  }
  
  void setRequestedBy(int? userId) {
    state = state.copyWith(requestedBy: userId);
  }
}

/// Provider de quotation requests filtrados
@riverpod
Future<List<QuotationRequest>> filteredQuotationRequests(
  FilteredQuotationRequestsRef ref,
) async {
  final filter = ref.watch(quotationRequestFilterProvider);
  
  return ref.watch(
    quotationRequestListProvider(
      status: filter.status,
      requestedBy: filter.requestedBy,
    ).future,
  );
}
```

### 2. Clean Architecture Screens

```dart
// logixportal_flutter/lib/features/procurement/presentation/screens/quotation_request_list_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class QuotationRequestListScreen extends ConsumerWidget {
  const QuotationRequestListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final quotationRequestsAsync = ref.watch(filteredQuotationRequestsProvider);
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('Solicitudes de Cotización'),
        actions: [
          IconButton(
            icon: const Icon(Icons.filter_list),
            onPressed: () => _showFilterDialog(context, ref),
          ),
        ],
      ),
      body: quotationRequestsAsync.when(
        data: (quotationRequests) {
          if (quotationRequests.isEmpty) {
            return const Center(
              child: Text('No hay solicitudes de cotización'),
            );
          }
          
          return ListView.builder(
            itemCount: quotationRequests.length,
            itemBuilder: (context, index) {
              final qr = quotationRequests[index];
              return QuotationRequestCard(quotationRequest: qr);
            },
          );
        },
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => Center(
          child: Text('Error: $error'),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _navigateToCreateScreen(context),
        child: const Icon(Icons.add),
      ),
    );
  }
  
  void _showFilterDialog(BuildContext context, WidgetRef ref) {
    // Implementar diálogo de filtros
  }
  
  void _navigateToCreateScreen(BuildContext context) {
    // Navegar a pantalla de creación
  }
}
```

---

## AUTENTICACIÓN Y AUTORIZACIÓN

### Backend (Serverpod)

```dart
// Middleware de autenticación
class AuthenticationMiddleware extends Middleware {
  @override
  Future<void> run(
    Session session,
    String method,
    Map<String, dynamic> arguments,
  ) async {
    // Verificar token JWT
    if (!session.isUserSignedIn) {
      throw UnauthorizedException('No autenticado');
    }
    
    // Extraer tenant_id del token
    final tenantId = session.auth.tenantId;
    
    // Validar tenant activo
    final tenant = await Tenant.db.findById(session, tenantId);
    if (tenant == null || !tenant.active) {
      throw ForbiddenException('Tenant inválido o inactivo');
    }
    
    // Continuar
    await next(session, method, arguments);
  }
}
```

### Frontend (Flutter)

```dart
// Auth Provider
@riverpod
class Auth extends _$Auth {
  @override
  Future<User?> build() async {
    // Intentar restaurar sesión desde SharedPreferences
    final prefs = await SharedPreferences.getInstance();
    final token = prefs.getString('auth_token');
    
    if (token != null) {
      try {
        final client = ref.read(serverpodClientProvider);
        // Validar token con servidor
        final user = await client.auth.validateToken(token);
        return user;
      } catch (e) {
        // Token inválido, limpiar
        await prefs.remove('auth_token');
        return null;
      }
    }
    
    return null;
  }
  
  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    
    state = await AsyncValue.guard(() async {
      final client = ref.read(serverpodClientProvider);
      final authResponse = await client.auth.login(email, password);
      
      // Guardar token
      final prefs = await SharedPreferences.getInstance();
      await prefs.setString('auth_token', authResponse.token);
      
      return authResponse.user;
    });
  }
  
  Future<void> logout() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove('auth_token');
    state = const AsyncValue.data(null);
  }
}
```

---

## DEPLOYMENT

### Docker Compose (Development)

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: logixportal_dev
      POSTGRES_USER: logixportal
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  server:
    build:
      context: .
      dockerfile: docker/Dockerfile.server
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgresql://logixportal:dev_password@postgres:5432/logixportal_dev
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    volumes:
      - ./logixportal_server:/app

volumes:
  postgres_data:
```

### Dockerfile (Backend)

```dockerfile
# docker/Dockerfile.server
FROM dart:stable AS build

WORKDIR /app
COPY logixportal_server/pubspec.* ./
RUN dart pub get

COPY logixportal_server/ .
RUN dart compile exe bin/main.dart -o bin/server

FROM scratch
COPY --from=build /runtime/ /
COPY --from=build /app/bin/server /app/bin/
COPY --from=build /app/config/ /app/config/

EXPOSE 8080
CMD ["/app/bin/server"]
```

---

**Este documento es la base técnica completa. Próximos documentos cubrirán:**
- Manual de desarrollo paso a paso
- Plan de testing y QA
- Estrategia de deployment en producción
