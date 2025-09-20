# 🧪 Test-Driven Development (TDD) en Java

## Índice
1. [Conceptos Básicos](#conceptos-básicos)
   - [¿Qué es TDD?](#qué-es-tdd)
   - [Ciclo Red-Green-Refactor](#ciclo-red-green-refactor)
2. [Implementación](#implementación)
   - [Configuración](#configuración)
   - [Primeros Pasos](#primeros-pasos)
3. [Ejemplos Prácticos](#ejemplos-prácticos)
   - [Básico](#ejemplo-básico)
   - [Avanzado](#ejemplo-avanzado)
4. [Casos de Uso Reales](#casos-de-uso-reales)

## Conceptos Básicos

### ¿Qué es TDD?
TDD es una metodología de desarrollo donde primero se escriben las pruebas y luego el código que las hace pasar. El proceso sigue un ciclo conocido como Red-Green-Refactor.

### Ciclo Red-Green-Refactor
1. **Red**: Escribir una prueba que falle
2. **Green**: Escribir el código mínimo para que la prueba pase
3. **Refactor**: Mejorar el código manteniendo las pruebas en verde

## Implementación

### Configuración
```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.8.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>4.5.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Primeros Pasos
```java
// Primer test (RED)
@Test
void testCalculadoraSuma() {
    Calculadora calc = new Calculadora();
    assertEquals(4, calc.sumar(2, 2));
}

// Primera implementación (GREEN)
public class Calculadora {
    public int sumar(int a, int b) {
        return a + b;
    }
}

// Refactorización (REFACTOR)
public class Calculadora {
    public int sumar(int... numeros) {
        return Arrays.stream(numeros).sum();
    }
}
```

## Ejemplos Prácticos

### Ejemplo Básico
```java
// Test primero
public class ValidadorPasswordTest {
    @Test
    void passwordDebeSerMayorA8Caracteres() {
        ValidadorPassword validador = new ValidadorPassword();
        assertFalse(validador.esValido("corto"));
    }
    
    @Test
    void passwordDebeContenerNumeros() {
        ValidadorPassword validador = new ValidadorPassword();
        assertFalse(validador.esValido("sinNumeros"));
    }
}

// Implementación después
public class ValidadorPassword {
    public boolean esValido(String password) {
        if (password == null || password.length() < 8) {
            return false;
        }
        return password.matches(".*\\d.*");
    }
}
```

### Ejemplo Avanzado
```java
public class CarritoCompraTest {
    private CarritoCompra carrito;
    private ProductoService productoService;
    
    @BeforeEach
    void setUp() {
        productoService = mock(ProductoService.class);
        carrito = new CarritoCompra(productoService);
    }
    
    @Test
    void agregarProductoDebeIncrementarTotal() {
        // Arrange
        when(productoService.obtenerPrecio("PROD1")).thenReturn(10.0);
        
        // Act
        carrito.agregarProducto("PROD1", 2);
        
        // Assert
        assertEquals(20.0, carrito.getTotal());
    }
    
    @Test
    void descuentoAplicadoCorrectamente() {
        // Arrange
        when(productoService.obtenerPrecio("PROD1")).thenReturn(100.0);
        
        // Act
        carrito.agregarProducto("PROD1", 1);
        carrito.aplicarDescuento(10);
        
        // Assert
        assertEquals(90.0, carrito.getTotal());
    }
}

public class CarritoCompra {
    private final ProductoService productoService;
    private Map<String, Integer> productos = new HashMap<>();
    private double total = 0.0;
    
    public CarritoCompra(ProductoService productoService) {
        this.productoService = productoService;
    }
    
    public void agregarProducto(String codigo, int cantidad) {
        double precio = productoService.obtenerPrecio(codigo);
        productos.put(codigo, productos.getOrDefault(codigo, 0) + cantidad);
        total += precio * cantidad;
    }
    
    public void aplicarDescuento(double porcentaje) {
        total = total * (1 - porcentaje / 100);
    }
    
    public double getTotal() {
        return total;
    }
}
```

## Casos de Uso Reales

### Sistema de Reservas
```java
@SpringBootTest
public class SistemaReservasTest {
    @Mock
    private DisponibilidadService disponibilidadService;
    
    @Mock
    private NotificacionService notificacionService;
    
    @InjectMocks
    private SistemaReservas sistema;
    
    @Test
    void reservaExitosaDebeNotificarYActualizarDisponibilidad() {
        // Arrange
        LocalDate fecha = LocalDate.now().plusDays(1);
        Habitacion habitacion = new Habitacion("101", "Simple");
        when(disponibilidadService.verificarDisponibilidad(habitacion, fecha))
            .thenReturn(true);
        
        // Act
        ResultadoReserva resultado = sistema.realizarReserva(
            new Reserva(habitacion, fecha, "Juan Pérez"));
        
        // Assert
        assertTrue(resultado.esExitoso());
        verify(disponibilidadService).actualizarDisponibilidad(habitacion, fecha);
        verify(notificacionService).enviarConfirmacion(any(Reserva.class));
    }
    
    @Test
    void reservaEnFechaPasadaDebeFallar() {
        // Arrange
        LocalDate fechaPasada = LocalDate.now().minusDays(1);
        Habitacion habitacion = new Habitacion("101", "Simple");
        
        // Act & Assert
        assertThrows(ReservaInvalidaException.class, () -> 
            sistema.realizarReserva(new Reserva(habitacion, fechaPasada, "Juan Pérez")));
    }
}

public class SistemaReservas {
    private final DisponibilidadService disponibilidadService;
    private final NotificacionService notificacionService;
    
    public SistemaReservas(DisponibilidadService disponibilidadService,
                          NotificacionService notificacionService) {
        this.disponibilidadService = disponibilidadService;
        this.notificacionService = notificacionService;
    }
    
    public ResultadoReserva realizarReserva(Reserva reserva) {
        if (reserva.getFecha().isBefore(LocalDate.now())) {
            throw new ReservaInvalidaException("No se puede reservar en fechas pasadas");
        }
        
        if (disponibilidadService.verificarDisponibilidad(
                reserva.getHabitacion(), reserva.getFecha())) {
            disponibilidadService.actualizarDisponibilidad(
                reserva.getHabitacion(), reserva.getFecha());
            notificacionService.enviarConfirmacion(reserva);
            return new ResultadoReserva(true, "Reserva confirmada");
        }
        
        return new ResultadoReserva(false, "No hay disponibilidad");
    }
}
```

### Mejores Prácticas TDD

1. **Empezar Simple**
   - Escribir primero pruebas sencillas
   - Implementar la solución más simple que funcione

2. **Baby Steps**
   - Dar pasos pequeños y seguros
   - No intentar resolver todo de una vez

3. **Refactorizar Constantemente**
   - Mantener el código limpio
   - No duplicar código
   - Extraer métodos comunes

4. **Mantener las Pruebas Limpias**
   - Usar nombres descriptivos
   - Seguir el patrón Arrange-Act-Assert
   - Mantener las pruebas independientes

5. **Documentar con Pruebas**
   - Las pruebas son la mejor documentación
   - Usar nombres de pruebas descriptivos
   ```java
   @Test
   void dadoUsuarioNuevo_cuandoRegistra_entoncesCreaCuentaYEnviaEmail()
   ```