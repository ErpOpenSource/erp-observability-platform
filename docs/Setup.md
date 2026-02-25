# Setup — Integración de microservicios ERP con la plataforma de observabilidad

Este documento explica cómo integrar cualquier microservicio del ERP con la plataforma de observabilidad compartida:

- Métricas → Prometheus
- Trazas → OpenTelemetry → Tempo
- Logs → stdout JSON + (opcional) Loki/Promtail

---

# 1️⃣ Requisitos previos

- El microservicio debe estar conectado a la red Docker `erp-platform`.
- Debe exponer:
  - `/actuator/prometheus` para métricas
  - OTLP hacia `otel-collector:4317` (gRPC) o `otel-collector:4318` (HTTP)

---

# 2️⃣ Métricas (Spring Boot + Micrometer + Prometheus)

## 2.1 Añadir dependencia

### Maven
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
Gradle
implementation("io.micrometer:micrometer-registry-prometheus")
2.2 Exponer endpoint Prometheus

application.yml

management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    prometheus:
      enabled: true
2.3 Añadir el servicio a Prometheus

Editar config/prometheus.yml en el repo de observabilidad:

- job_name: "erp-sales-service"
  metrics_path: "/actuator/prometheus"
  static_configs:
    - targets: ["erp-sales-service:8080"]

⚠️ El nombre del target debe coincidir exactamente con el nombre del servicio en Docker Compose.

Después:

docker compose restart prometheus
3️⃣ Trazas (Spring Boot → OpenTelemetry → Tempo)
3.1 Añadir dependencias
Maven
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>

<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
Gradle
implementation("io.micrometer:micrometer-tracing-bridge-otel")
implementation("io.opentelemetry:opentelemetry-exporter-otlp")
3.2 Configurar sampling
Entorno DEV
management:
  tracing:
    sampling:
      probability: 1.0
Entorno PROD (recomendado)
management:
  tracing:
    sampling:
      probability: 0.1
3.3 Variables de entorno en Docker

En el docker-compose.yml del microservicio:

environment:
  - OTEL_SERVICE_NAME=erp-api-gateway
  - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
  - OTEL_EXPORTER_OTLP_PROTOCOL=grpc

Si usas HTTP:

environment:
  - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4318
  - OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
4️⃣ Logs (stdout + JSON estructurado)
4.1 Recomendación enterprise

Los microservicios deben:

Loggear a stdout

En formato JSON estructurado

Nunca enviar logs directamente a Loki desde el servicio.

4.2 Dependencia Logstash encoder
Maven
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>7.4</version>
</dependency>
4.3 Configuración básica logback

logback-spring.xml

<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
  </appender>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
4.4 Activar Loki (opcional)

Si quieres logs centralizados:

docker compose --profile logs up -d

Promtail recogerá automáticamente los logs de Docker.

5️⃣ Escenarios de red
Escenario A — Todo en el mismo docker-compose

Todos los servicios comparten:

networks:
  - erp-platform
Escenario B — Gateway y servicios en otro compose (recomendado)

En el compose del gateway:

networks:
  erp-platform:
    external: true
    name: erp-platform

Y en el servicio:

services:
  erp-api-gateway:
    networks:
      - erp-platform
6️⃣ Checklist de validación por servicio
Métricas

http://servicio:8080/actuator/prometheus responde

Prometheus → Targets → UP

Trazas

El servicio exporta a otel-collector

Aparecen trazas en Tempo (Grafana → Explore)

Logs (si activado)

Logs aparecen en Loki

Se pueden consultar desde Grafana


---

# ✅ Cómo usar el repo correctamente (README principal)

Añade esta sección a tu `README.md` raíz:

```md
# Uso del repositorio

## 1️⃣ Clonar el repositorio

```bash
git clone <repo-url>
cd erp-observability-platform
2️⃣ Configurar variables de entorno

Entrar en carpeta docker:

cd docker

Copiar el ejemplo:

cp .env.example .env

Editar .env si quieres cambiar:

Puertos

Usuario/contraseña Grafana

Retención Prometheus

3️⃣ Puertos usados por defecto
Servicio	Puerto
Grafana	3000
Prometheus	9090
Tempo	3200
OTLP gRPC	4317
OTLP HTTP	4318
Loki (opcional)	3100
4️⃣ Levantar la plataforma

Desde carpeta docker:

docker compose --env-file .env up -d

Ver estado:

docker compose ps
5️⃣ Activar logs (opcional)
docker compose --profile logs up -d
6️⃣ Accesos

Grafana:

http://localhost:3000

Prometheus:

http://localhost:9090

Tempo:

http://localhost:3200
7️⃣ Parar la plataforma
docker compose down

Eliminar volúmenes (reset completo):

docker compose down -v
8️⃣ Red compartida para microservicios

Crear red (solo primera vez):

docker network create erp-platform

Los microservicios deben unirse a esa red para que Prometheus y OTEL funcionen.