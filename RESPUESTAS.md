# Laboratorio Observabilidad — Respuestas

**Alumno:** Olazabal Avila Fernando  
**Repositorio:** https://github.com/folazabala1-del/observabilidadinfraS10

---

## Respuestas a las preguntas

### 1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?

Porque Prometheus solo guarda números, como el porcentaje de CPU o cuántas 
peticiones llegaron. Pero eso no te dice qué pasó exactamente ni por qué 
falló algo. Loki guarda los logs, que son los mensajes de texto que genera 
la aplicación, como errores o eventos. Los dos se complementan: con 
Prometheus ves que algo salió mal (la CPU subió mucho), y con Loki puedes 
buscar en los logs qué error ocurrió exactamente en ese momento.

### 2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén 
aprovisionadas como código y no creadas a mano?

La principal ventaja es que si el entorno se cae o alguien hace 
`docker compose down -v`, al volver a levantar todo con 
`docker compose up` las fuentes de datos ya están configuradas 
automáticamente. No depende de que alguien recuerde hacer los clics 
correctos en Grafana. Además si trabajas en equipo todos tienen 
exactamente la misma configuración porque está en el repositorio.

### 3. ¿Por qué el panel "CPU contenedor" y "CPU host" pueden mostrar 
valores muy distintos? ¿Cuál usarías para alertar sobre una aplicación?

El host mide el uso total de la máquina, incluyendo todos los contenedores 
que están corriendo más el sistema operativo. El contenedor solo mide lo 
que consume ese servicio específico. Por ejemplo el host puede estar al 
40% pero el backend solo estar usando el 3% de eso. Para alertar sobre 
una aplicación concreta usaría el panel por contenedor, porque así sé 
exactamente que ES ese servicio el que está consumiendo de más, y no otro 
proceso del sistema.

### 4. ¿Qué diferencia hay entre el evaluation interval y el pending period?

El evaluation interval es cada cuánto Grafana revisa si la condición se 
cumple, en nuestro caso cada 10 segundos. El pending period es cuánto 
tiempo tiene que mantenerse esa condición antes de disparar la alarma, 
que pusimos en 30 segundos. La diferencia práctica es que sin el pending 
period cualquier pico corto de CPU dispararía la alarma aunque dure solo 
unos segundos. Con 30s de pending nos aseguramos de que el problema es 
real y no un pico momentáneo.

---

## Cómo validar el laboratorio

### Requisitos
- Docker Desktop corriendo
- Puertos libres: `3000`, `3001`, `3100`, `8080`, `8081`, `9090`, `9100`, `12345`

### Levantar el stack
```bash
git clone https://github.com/folazabala1-del/observabilidadinfraS10.git
cd observabilidadinfraS10
docker compose up -d --build
docker compose ps
```

### Servicios disponibles

| Servicio | URL |
|---|---|
| Frontend | http://localhost:8080 |
| Backend métricas | http://localhost:3001/metrics |
| Grafana | http://localhost:3000 (admin123/admin123) |
| Prometheus | http://localhost:9090 |

### Verificar dashboard
Grafana → Dashboards → **Observabilidad — Olazabal Avila Fernando**  
Debe mostrar 4 paneles: CPU contenedor, CPU host, logs de aplicación 
y logs de infraestructura.

### Verificar alarma
Grafana → Alerting → Alert rules → regla **CPU backend > 50%**  
Para dispararla:
```bash
curl "http://localhost:3001/load?seconds=60"
```
Esperar a que pase de Normal → Pending → Firing.

### Reset
```bash
docker compose down -v
```