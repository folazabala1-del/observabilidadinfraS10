# Laboratorio Observabilidad — Respuestas

**Alumno:** Olazabal Avila Fernando  
**Repositorio:** https://github.com/folazabala1-del/observabilidadinfraS10

---

## Respuestas a las preguntas

### 1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos /metrics?

Porque Prometheus solo nos muestra métricas como el uso de CPU, memoria o 
la cantidad de peticiones. En cambio, Loki guarda los logs de la aplicación. 
Entonces, si vemos que algo anda mal en las métricas, podemos revisar los 
logs para encontrar el error o ver qué fue lo que pasó.

### 2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén 
aprovisionadas como código y no creadas a mano?

La ventaja es que todo queda configurado automáticamente. Si se vuelve a 
levantar el proyecto, las fuentes de datos ya aparecen listas y no hay que 
configurarlas una por una. Además, todos los que trabajen con el proyecto 
tendrán la misma configuración.

### 3. El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores 
muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

Porque el CPU host muestra el consumo de toda la máquina, mientras que el 
CPU contenedor solo muestra el consumo del servicio que estamos monitoreando. 
Para una aplicación específica usaría el CPU contenedor, ya que me interesa 
saber cuánto está consumiendo esa aplicación y no todo el sistema.

### 4. ¿Qué diferencia hay entre el evaluation interval y el pending period?

El evaluation interval es cada cuánto tiempo Grafana revisa si se cumple la 
condición de la alerta. El pending period es el tiempo que esa condición debe 
mantenerse antes de que la alerta se active. Esto ayuda a que una alerta no 
se dispare por un pico de pocos segundos.

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