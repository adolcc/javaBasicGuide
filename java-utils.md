# Java Utils

## Índice
1. [Colecciones de Utilidades](#colecciones-de-utilidades)
   - [Arrays](#arrays)
   - [Collections](#collections)
   - [Objects](#objects)
2. [Fecha y Tiempo](#fecha-y-tiempo)
   - [LocalDate](#localdate)
   - [LocalTime](#localtime)
   - [LocalDateTime](#localdatetime)
3. [Utilidades de String](#utilidades-de-string)
4. [Ejemplos Prácticos](#ejemplos-prácticos)

## Colecciones de Utilidades

### Arrays
```java
// Operaciones comunes con Arrays
int[] numeros = {3, 1, 4, 1, 5, 9};
Arrays.sort(numeros);                     // Ordenar
int indice = Arrays.binarySearch(numeros, 4); // Búsqueda binaria
int[] copia = Arrays.copyOf(numeros, 6);  // Copiar array
Arrays.fill(numeros, 0);                  // Rellenar array
boolean sonIguales = Arrays.equals(numeros, copia); // Comparar arrays
```

### Collections
```java
// Utilidades para colecciones
List<String> lista = new ArrayList<>();
Collections.addAll(lista, "uno", "dos", "tres");
Collections.sort(lista);                  // Ordenar
Collections.reverse(lista);               // Invertir orden
Collections.shuffle(lista);               // Mezclar elementos
Collections.frequency(lista, "uno");      // Contar ocurrencias
Collections.max(lista);                   // Obtener máximo
Collections.min(lista);                   // Obtener mínimo
```

### Objects
```java
// Utilidades para objetos
public class Persona {
    private String nombre;
    
    // Constructor y getters/setters
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Persona persona = (Persona) obj;
        return Objects.equals(nombre, persona.nombre);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(nombre);
    }
}
```

## Fecha y Tiempo

### LocalDate
```java
// Manejo de fechas
LocalDate hoy = LocalDate.now();
LocalDate ayer = hoy.minusDays(1);
LocalDate mañana = hoy.plusDays(1);

// Formateo de fechas
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String fechaFormateada = hoy.format(formatter);

// Comparaciones
boolean esAntes = ayer.isBefore(hoy);
boolean esDespues = mañana.isAfter(hoy);
```

### LocalTime
```java
// Manejo de tiempo
LocalTime ahora = LocalTime.now();
LocalTime dosHorasDespues = ahora.plusHours(2);
LocalTime treintaMinutosAntes = ahora.minusMinutes(30);

// Parseando tiempo
LocalTime tiempo = LocalTime.parse("15:30");
```

### LocalDateTime
```java
// Fecha y tiempo combinados
LocalDateTime fechaHora = LocalDateTime.now();
LocalDateTime futuro = fechaHora.plusWeeks(1)
                               .plusHours(2)
                               .plusMinutes(30);

// Zona horaria
ZonedDateTime zonaHoraria = ZonedDateTime.now(ZoneId.of("Europe/Madrid"));
```

## Utilidades de String
```java
// Operaciones con String
String texto = "  Hola Mundo  ";
String procesado = texto.trim()           // Eliminar espacios
                       .toLowerCase()      // Convertir a minúsculas
                       .replace("o", "0"); // Reemplazar caracteres

// String.format
String formato = String.format("Hola %s, tienes %d años", "Juan", 25);

// StringBuilder para concatenación eficiente
StringBuilder sb = new StringBuilder();
sb.append("Hola")
  .append(" ")
  .append("Mundo");
String resultado = sb.toString();
```

## Ejemplos Prácticos

### Ejemplo de Utilidades Combinadas
```java
public class GestorTareas {
    private List<Tarea> tareas = new ArrayList<>();
    
    public void agregarTarea(String descripcion) {
        Tarea tarea = new Tarea(descripcion, LocalDateTime.now());
        tareas.add(tarea);
        // Ordenar por fecha
        Collections.sort(tareas, Comparator.comparing(Tarea::getFechaCreacion));
    }
    
    public List<Tarea> obtenerTareasDelDia(LocalDate fecha) {
        return tareas.stream()
                    .filter(t -> t.getFechaCreacion().toLocalDate().equals(fecha))
                    .collect(Collectors.toList());
    }
    
    public String generarReporte() {
        StringBuilder reporte = new StringBuilder();
        reporte.append("Reporte de Tareas\n");
        reporte.append("Fecha: ").append(LocalDate.now().format(
            DateTimeFormatter.ofPattern("dd/MM/yyyy"))).append("\n");
        
        for (Tarea tarea : tareas) {
            reporte.append(String.format("- %s (Creada: %s)\n",
                tarea.getDescripcion(),
                tarea.getFechaCreacion().format(
                    DateTimeFormatter.ofPattern("HH:mm:ss"))));
        }
        
        return reporte.toString();
    }
}

class Tarea {
    private String descripcion;
    private LocalDateTime fechaCreacion;
    
    // Constructor y getters
    public Tarea(String descripcion, LocalDateTime fechaCreacion) {
        this.descripcion = Objects.requireNonNull(descripcion, "La descripción no puede ser nula");
        this.fechaCreacion = Objects.requireNonNull(fechaCreacion, "La fecha no puede ser nula");
    }
    
    // Getters...
}
```

### Caso de Uso Real: Sistema de Registro
```java
public class SistemaRegistro {
    private static final int MAX_INTENTOS = 3;
    private Map<String, List<IntentosAcceso>> registros;
    
    public SistemaRegistro() {
        this.registros = new HashMap<>();
    }
    
    public void registrarIntento(String usuario, boolean exitoso) {
        IntentosAcceso intento = new IntentosAcceso(
            LocalDateTime.now(),
            exitoso
        );
        
        registros.computeIfAbsent(usuario, k -> new ArrayList<>())
                .add(intento);
                
        // Limpiar intentos antiguos
        limpiarIntentosAntiguos(usuario);
        
        // Verificar bloqueo
        if (debeBloquear(usuario)) {
            throw new SeguridadException("Usuario bloqueado por múltiples intentos fallidos");
        }
    }
    
    private void limpiarIntentosAntiguos(String usuario) {
        LocalDateTime limite = LocalDateTime.now().minusHours(1);
        List<IntentosAcceso> intentos = registros.get(usuario);
        
        if (intentos != null) {
            intentos.removeIf(intento -> 
                intento.getFecha().isBefore(limite));
        }
    }
    
    private boolean debeBloquear(String usuario) {
        List<IntentosAcceso> intentos = registros.get(usuario);
        if (intentos == null) return false;
        
        long intentosFallidos = intentos.stream()
            .filter(i -> !i.isExitoso())
            .count();
            
        return intentosFallidos >= MAX_INTENTOS;
    }
    
    public String generarReporteSeguridad() {
        return registros.entrySet().stream()
            .map(entry -> String.format(
                "Usuario: %s, Intentos: %d, Último intento: %s",
                entry.getKey(),
                entry.getValue().size(),
                entry.getValue().isEmpty() ? "N/A" : 
                    entry.getValue().get(entry.getValue().size() - 1)
                        .getFecha().format(DateTimeFormatter.ISO_LOCAL_DATE_TIME)
            ))
            .collect(Collectors.joining("\n"));
    }
}

class IntentosAcceso {
    private final LocalDateTime fecha;
    private final boolean exitoso;
    
    // Constructor y getters...
}

class SeguridadException extends RuntimeException {
    public SeguridadException(String mensaje) {
        super(mensaje);
    }
}
```

Este ejemplo muestra:
- Uso de Collections y Arrays
- Manejo de fechas y tiempo
- Manipulación de Strings
- Utilidades de Objects
- Casos de uso prácticos y reales