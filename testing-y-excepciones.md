# 🛠 Testing y Desarrollo en Java

## Índice
1. [Excepciones](#excepciones)
   - [Tipos de Excepciones](#tipos-de-excepciones)
   - [Try-Catch-Finally](#try-catch-finally)
   - [Throws y Throw](#throws-y-throw)
2. [Testing Básico](#testing-básico)
   - [JUnit](#junit)
   - [Assertions](#assertions)
3. [Ejemplos Prácticos](#ejemplos-prácticos)
   - [Básicos](#ejemplos-básicos)
   - [Avanzados](#ejemplos-avanzados)
4. [Casos de Uso Reales](#casos-de-uso-reales)

## Excepciones

### Tipos de Excepciones
```java
// Checked Exception (debe ser declarada o manejada)
public class MiExcepcion extends Exception {
    public MiExcepcion(String mensaje) {
        super(mensaje);
    }
}

// Runtime Exception (no requiere declaración)
public class MiRuntimeException extends RuntimeException {
    public MiRuntimeException(String mensaje) {
        super(mensaje);
    }
}
```

### Try-Catch-Finally
```java
public void ejemploManejoCombinado() {
    FileReader archivo = null;
    try {
        archivo = new FileReader("archivo.txt");
        // Código que puede lanzar excepción
    } catch (FileNotFoundException e) {
        System.err.println("Archivo no encontrado: " + e.getMessage());
    } catch (IOException e) {
        System.err.println("Error de lectura: " + e.getMessage());
    } finally {
        if (archivo != null) {
            try {
                archivo.close();
            } catch (IOException e) {
                System.err.println("Error al cerrar el archivo");
            }
        }
    }
}
```

### Try-with-Resources
```java
public void ejemploTryWithResources() {
    try (FileReader fr = new FileReader("archivo.txt");
         BufferedReader br = new BufferedReader(fr)) {
        String linea;
        while ((linea = br.readLine()) != null) {
            System.out.println(linea);
        }
    } catch (IOException e) {
        System.err.println("Error: " + e.getMessage());
    }
}
```

## Testing Básico

### JUnit
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {
    @Test
    void testSuma() {
        Calculadora calc = new Calculadora();
        assertEquals(4, calc.suma(2, 2), "2 + 2 debe ser igual a 4");
    }
    
    @Test
    void testDivision() {
        Calculadora calc = new Calculadora();
        assertThrows(ArithmeticException.class, () -> {
            calc.dividir(1, 0);
        });
    }
}
```

### Assertions
```java
@Test
void ejemploAssertions() {
    // Verificaciones básicas
    assertTrue(condicion);
    assertFalse(condicion);
    assertNull(objeto);
    assertNotNull(objeto);
    
    // Comparaciones
    assertEquals(esperado, actual);
    assertNotEquals(esperado, actual);
    
    // Arrays
    assertArrayEquals(esperado, actual);
    
    // Excepciones
    assertThrows(Exception.class, () -> { /* código */ });
}
```

## Ejemplos Prácticos

### Ejemplo Básico: Validación de Edad
```java
public class ValidadorEdad {
    public void validarEdad(int edad) throws EdadInvalidaException {
        if (edad < 0) {
            throw new EdadInvalidaException("La edad no puede ser negativa");
        }
        if (edad > 120) {
            throw new EdadInvalidaException("La edad es demasiado alta");
        }
    }
}

@Test
void testValidadorEdad() {
    ValidadorEdad validador = new ValidadorEdad();
    
    assertThrows(EdadInvalidaException.class, () -> {
        validador.validarEdad(-1);
    });
    
    assertDoesNotThrow(() -> {
        validador.validarEdad(25);
    });
}
```

### Ejemplo Avanzado: Sistema Bancario
```java
public class CuentaBancaria {
    private double saldo;
    
    public void retirar(double cantidad) throws SaldoInsuficienteException {
        if (cantidad > saldo) {
            throw new SaldoInsuficienteException(
                "Saldo insuficiente. Saldo actual: " + saldo + 
                ", Cantidad solicitada: " + cantidad
            );
        }
        saldo -= cantidad;
    }
}

@Test
void testRetiroExitoso() {
    CuentaBancaria cuenta = new CuentaBancaria(100.0);
    assertDoesNotThrow(() -> cuenta.retirar(50.0));
}

@Test
void testRetiroFallido() {
    CuentaBancaria cuenta = new CuentaBancaria(100.0);
    assertThrows(SaldoInsuficienteException.class, 
        () -> cuenta.retirar(150.0)
    );
}
```

## Casos de Uso Reales

### Sistema de Procesamiento de Pagos
```java
@Service
public class ServicioPagos {
    private final Logger logger = LoggerFactory.getLogger(ServicioPagos.class);
    
    @Autowired
    private PasarelaPagos pasarela;
    
    public ResultadoPago procesarPago(DatosPago datos) {
        try {
            validarDatos(datos);
            return pasarela.ejecutarPago(datos);
        } catch (DatosInvalidosException e) {
            logger.error("Datos inválidos: {}", e.getMessage());
            return new ResultadoPago(false, "Datos inválidos");
        } catch (ErrorPasarelaException e) {
            logger.error("Error en pasarela: {}", e.getMessage());
            return new ResultadoPago(false, "Error de procesamiento");
        }
    }
}

@SpringBootTest
class ServicioPagosTest {
    @Mock
    private PasarelaPagos pasarela;
    
    @InjectMocks
    private ServicioPagos servicioPagos;
    
    @Test
    void testPagoExitoso() {
        DatosPago datos = new DatosPago("4532-xxxx-xxxx-xxxx", 100.0);
        when(pasarela.ejecutarPago(any(DatosPago.class)))
            .thenReturn(new ResultadoPago(true, ""));
            
        ResultadoPago resultado = servicioPagos.procesarPago(datos);
        assertTrue(resultado.isExitoso());
    }
}
```