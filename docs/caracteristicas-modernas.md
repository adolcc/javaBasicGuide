# Características Modernas en Java

## Índice
1. [Lambdas](#lambdas)
   - [Concepto](#concepto-de-lambdas)
   - [Sintaxis Básica](#sintaxis-básica)
   - [Ejemplos Prácticos](#ejemplos-prácticos-de-lambdas)
2. [Anotaciones](#anotaciones)
   - [Concepto](#concepto-de-anotaciones)
   - [Tipos Comunes](#tipos-comunes-de-anotaciones)
   - [Ejemplos Prácticos](#ejemplos-prácticos-de-anotaciones)

## Lambdas

### Concepto de Lambdas
Las expresiones lambda son funciones anónimas que permiten tratar la funcionalidad como un argumento, es decir, pasar funciones como argumentos. Son una característica fundamental de la programación funcional en Java.

### Sintaxis Básica
```java
// Sintaxis básica de lambda
(parámetros) -> expresión
(parámetros) -> { sentencias; }

// Ejemplos de sintaxis
() -> System.out.println("Hola")  // Sin parámetros
x -> x * 2                        // Un parámetro
(x, y) -> x + y                   // Múltiples parámetros
(String x, String y) -> x.length() + y.length()  // Con tipos explícitos
```

### Ejemplos Prácticos de Lambdas

#### Ejemplo Básico: Ordenamiento de Lista
```java
public class EjemploLambdaBasico {
    public static void main(String[] args) {
        List<String> nombres = Arrays.asList("Juan", "Ana", "Pedro", "María");
        
        // Ordenamiento tradicional
        Collections.sort(nombres, new Comparator<String>() {
            @Override
            public int compare(String a, String b) {
                return a.compareTo(b);
            }
        });

        // Con lambda
        Collections.sort(nombres, (a, b) -> a.compareTo(b));
        
        // Usando método de referencia (aún más conciso)
        Collections.sort(nombres, String::compareTo);
    }
}
```

#### Ejemplo Avanzado: Procesamiento de Streams
```java
public class ProcesadorDatos {
    private List<Empleado> empleados;

    public List<String> obtenerNombresEmpleadosFiltrados() {
        return empleados.stream()
            .filter(e -> e.getSalario() > 50000)
            .map(Empleado::getNombre)
            .sorted()
            .collect(Collectors.toList());
    }

    public Map<String, Double> calcularBonusAnual() {
        return empleados.stream()
            .collect(Collectors.toMap(
                Empleado::getNombre,
                e -> e.getSalario() * 0.1
            ));
    }
}
```

### Caso de Uso Real: Sistema de Notificaciones
```java
public class SistemaNotificaciones {
    private Map<TipoNotificacion, Consumer<Mensaje>> manejadores = new HashMap<>();

    public SistemaNotificaciones() {
        // Registrar manejadores usando lambdas
        manejadores.put(TipoNotificacion.EMAIL, mensaje -> 
            enviarEmail(mensaje.getDestinatario(), mensaje.getContenido()));
        
        manejadores.put(TipoNotificacion.SMS, mensaje -> 
            enviarSMS(mensaje.getDestinatario(), mensaje.getContenido()));
        
        manejadores.put(TipoNotificacion.PUSH, mensaje -> {
            if (mensaje.esUrgente()) {
                enviarNotificacionUrgente(mensaje);
            } else {
                enviarNotificacionNormal(mensaje);
            }
        });
    }

    public void enviarNotificacion(Mensaje mensaje) {
        Consumer<Mensaje> manejador = manejadores.get(mensaje.getTipo());
        if (manejador != null) {
            manejador.accept(mensaje);
        }
    }
}
```

## Anotaciones

### Concepto de Anotaciones
Las anotaciones son una forma de añadir metadatos al código fuente Java. Proporcionan información adicional sobre el programa que puede ser utilizada por el compilador, herramientas de desarrollo o en tiempo de ejecución.

### Tipos Comunes de Anotaciones
```java
// Anotaciones básicas de Java
@Override           // Indica que un método sobrescribe uno de la superclase
@Deprecated         // Marca un elemento como obsoleto
@SuppressWarnings  // Suprime advertencias del compilador
@FunctionalInterface // Indica que la interfaz es funcional (un solo método abstracto)

// Anotaciones comunes en frameworks
@Test              // JUnit - Marca un método como prueba
@Autowired         // Spring - Inyección de dependencias
@Entity            // JPA - Marca una clase como entidad de base de datos
```

### Ejemplos Prácticos de Anotaciones

#### Ejemplo Básico: Validación de Datos
```java
public class ValidacionUsuario {
    @NotNull
    private String nombre;

    @Email
    private String email;

    @Min(18)
    private int edad;

    @Pattern(regexp = "^[0-9]{8}[A-Z]$")
    private String dni;
}
```

#### Ejemplo Avanzado: Anotación Personalizada
```java
// Definición de una anotación personalizada
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Auditoria {
    String modulo() default "";
    boolean requiereAutorizacion() default false;
}

// Uso de la anotación personalizada
public class ServicioUsuarios {
    @Auditoria(modulo = "usuarios", requiereAutorizacion = true)
    public void crearUsuario(Usuario usuario) {
        // Lógica para crear usuario
    }
}

// Procesador de la anotación
public class ProcesadorAuditoria {
    public static void procesarMetodo(Method metodo) {
        if (metodo.isAnnotationPresent(Auditoria.class)) {
            Auditoria anotacion = metodo.getAnnotation(Auditoria.class);
            registrarAuditoria(anotacion.modulo(), anotacion.requiereAutorizacion());
        }
    }
}
```

### Caso de Uso Real: API REST con Spring Boot
```java
@RestController
@RequestMapping("/api/productos")
public class ProductoController {
    
    @Autowired
    private ProductoServicio productoServicio;

    @GetMapping("/{id}")
    @ApiOperation(value = "Obtener producto por ID")
    public ResponseEntity<Producto> obtenerProducto(
        @PathVariable @Min(1) Long id) {
        return productoServicio.buscarPorId(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    @Secured("ROLE_ADMIN")
    public ResponseEntity<Producto> crearProducto(
        @Valid @RequestBody ProductoDTO productoDTO) {
        Producto nuevoProducto = productoServicio.crear(productoDTO);
        return ResponseEntity
            .created(URI.create("/api/productos/" + nuevoProducto.getId()))
            .body(nuevoProducto);
    }
}
```