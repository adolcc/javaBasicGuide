# Test-Driven Development (TDD) y Assertions Modernos en Java

## Índice
1. [Test-Driven Development](#test-driven-development)
   - [Concepto](#concepto-tdd)
   - [Ciclo TDD](#ciclo-tdd)
2. [AssertAll y Assertions Modernos](#assertions-modernos)
   - [JUnit 5 Assertions](#junit-5-assertions)
   - [Agrupación de Assertions](#agrupación-de-assertions)
3. [Ejemplos Prácticos](#ejemplos-prácticos)
   - [Básicos](#ejemplos-básicos)
   - [Avanzados](#ejemplos-avanzados)
4. [Casos de Uso Reales](#casos-de-uso-reales)

## Test-Driven Development

### Concepto TDD
TDD es una práctica de desarrollo donde escribes las pruebas antes que el código de producción, siguiendo el ciclo Red-Green-Refactor.

### Ciclo TDD
1. **Red**: Escribe una prueba que falle
2. **Green**: Escribe el código mínimo para que la prueba pase
3. **Refactor**: Mejora el código manteniendo las pruebas en verde

```java
// Paso 1: Red - Escribimos la prueba primero
@Test
void testCalcularPrecioTotal() {
    CarritoCompra carrito = new CarritoCompra();
    carrito.agregarProducto(new Producto("Libro", 29.99));
    assertEquals(29.99, carrito.calcularTotal());
}

// Paso 2: Green - Implementación mínima
public class CarritoCompra {
    private List<Producto> productos = new ArrayList<>();
    
    public void agregarProducto(Producto producto) {
        productos.add(producto);
    }
    
    public double calcularTotal() {
        return productos.stream()
            .mapToDouble(Producto::getPrecio)
            .sum();
    }
}

// Paso 3: Refactor - Mejoramos el código
public class CarritoCompra {
    private final List<Producto> productos = new ArrayList<>();
    
    public void agregarProducto(Producto producto) {
        Objects.requireNonNull(producto, "El producto no puede ser nulo");
        productos.add(producto);
    }
    
    public double calcularTotal() {
        return productos.stream()
            .mapToDouble(Producto::getPrecio)
            .sum();
    }
}
```

## Assertions Modernos

### JUnit 5 Assertions
```java
@Test
void testAssertionsModernos() {
    // Assertions básicos
    assertEquals(4, 2 + 2);
    assertTrue(true);
    assertFalse(false);
    assertNull(null);
    assertNotNull("Hello");
    
    // Assertions con mensajes
    assertEquals(4, 2 + 2, "La suma debería ser 4");
    
    // Assertions con lambdas
    assertThrows(ArithmeticException.class, () -> {
        int resultado = 1 / 0;
    });
}
```

### Agrupación de Assertions
```java
@Test
void testAssertAll() {
    Usuario usuario = new Usuario("Juan", "juan@email.com", 25);
    
    assertAll("Usuario",
        () -> assertEquals("Juan", usuario.getNombre(), "El nombre debe coincidir"),
        () -> assertEquals("juan@email.com", usuario.getEmail(), "El email debe coincidir"),
        () -> assertTrue(usuario.getEdad() >= 18, "El usuario debe ser mayor de edad"),
        () -> assertNotNull(usuario.getFechaRegistro(), "La fecha de registro no puede ser nula")
    );
}
```

## Ejemplos Prácticos

### Ejemplos Básicos

#### TDD para Validador de Contraseñas
```java
// Primero, escribimos las pruebas
@Test
void testValidadorPassword() {
    ValidadorPassword validador = new ValidadorPassword();
    
    assertAll("Validación de Contraseña",
        () -> assertFalse(validador.esValida("corta"), 
            "Contraseña muy corta"),
        () -> assertFalse(validador.esValida("sinNumeros"), 
            "Contraseña sin números"),
        () -> assertFalse(validador.esValida("sin-mayusculas123"), 
            "Contraseña sin mayúsculas"),
        () -> assertTrue(validador.esValida("Contraseña123"), 
            "Contraseña válida")
    );
}

// Luego, implementamos la clase
public class ValidadorPassword {
    public boolean esValida(String password) {
        return password.length() >= 8 &&
               password.matches(".*\\d.*") &&
               password.matches(".*[A-Z].*");
    }
}
```

### Ejemplos Avanzados

#### Sistema de Reservas con TDD
```java
@Test
void testSistemaReservas() {
    SistemaReservas sistema = new SistemaReservas();
    LocalDate fecha = LocalDate.now().plusDays(1);
    
    assertAll("Sistema de Reservas",
        // Verificar disponibilidad
        () -> assertTrue(sistema.verificarDisponibilidad(fecha, "Sala A")),
        
        // Realizar reserva
        () -> {
            Reserva reserva = sistema.realizarReserva(fecha, "Sala A", "Juan");
            assertAll("Detalles de Reserva",
                () -> assertEquals(fecha, reserva.getFecha()),
                () -> assertEquals("Sala A", reserva.getSala()),
                () -> assertEquals("Juan", reserva.getUsuario()),
                () -> assertTrue(reserva.isConfirmada())
            );
        },
        
        // Verificar que la sala ya no está disponible
        () -> assertFalse(sistema.verificarDisponibilidad(fecha, "Sala A")),
        
        // Verificar que lanza excepción al intentar reservar sala ocupada
        () -> assertThrows(SalaNoDisponibleException.class, () -> 
            sistema.realizarReserva(fecha, "Sala A", "Pedro"))
    );
}
```

## Casos de Uso Reales

### Sistema de Gestión de Pedidos
```java
@Test
void testProcesarPedido() {
    // Arrange
    GestorPedidos gestor = new GestorPedidos();
    Pedido pedido = new Pedido("P001", Arrays.asList(
        new ItemPedido("Producto1", 2, 10.0),
        new ItemPedido("Producto2", 1, 15.0)
    ));
    
    // Act
    ResultadoProcesamiento resultado = gestor.procesarPedido(pedido);
    
    // Assert
    assertAll("Procesamiento de Pedido",
        // Verificar resultado general
        () -> assertTrue(resultado.isExitoso()),
        () -> assertNotNull(resultado.getIdTransaccion()),
        
        // Verificar detalles del pedido
        () -> assertAll("Detalles del Pedido",
            () -> assertEquals(35.0, pedido.getTotal()),
            () -> assertEquals(3, pedido.getCantidadTotal()),
            () -> assertTrue(pedido.getFechaProcesamiento().isBefore(
                LocalDateTime.now()))
        ),
        
        // Verificar estado del inventario
        () -> assertAll("Estado del Inventario",
            () -> assertTrue(gestor.verificarStock("Producto1") >= 0),
            () -> assertTrue(gestor.verificarStock("Producto2") >= 0)
        ),
        
        // Verificar notificaciones
        () -> assertAll("Notificaciones",
            () -> assertTrue(resultado.isNotificacionEnviada()),
            () -> assertNotNull(resultado.getFechaNotificacion())
        )
    );
}

// Implementación del sistema
public class GestorPedidos {
    public ResultadoProcesamiento procesarPedido(Pedido pedido) {
        // Validar pedido
        if (!validarPedido(pedido)) {
            throw new PedidoInvalidoException("Pedido inválido");
        }
        
        // Verificar stock
        if (!verificarStockDisponible(pedido)) {
            throw new StockInsuficienteException("Stock insuficiente");
        }
        
        // Procesar pago
        String idTransaccion = procesarPago(pedido);
        
        // Actualizar inventario
        actualizarInventario(pedido);
        
        // Enviar notificación
        enviarNotificacion(pedido);
        
        return new ResultadoProcesamiento(true, idTransaccion);
    }
    
    // Implementación de los métodos auxiliares...
}
```

En este ejemplo completo, hemos cubierto:
- Principios básicos de TDD
- Uso de assertAll para pruebas más organizadas
- Ejemplos prácticos progresivos
- Un caso de uso real con múltiples validaciones
- Pruebas que validan diferentes aspectos del sistema

La combinación de TDD con assertions modernos nos permite:
1. Desarrollar código más robusto
2. Tener mejor documentación a través de las pruebas
3. Detectar errores temprano en el ciclo de desarrollo
4. Mantener una base de código más mantenible