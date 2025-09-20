# Control de Flujo en Java

## Índice
1. [Estructuras Básicas](#estructuras-básicas)
   - if-else
   - switch
   - loops (for, while, do-while)
2. [Ejemplos Prácticos](#ejemplos-prácticos)
   - Básicos
   - Avanzados
3. [Casos de Uso Reales](#casos-de-uso-reales)

## Estructuras Básicas

### If-Else
```java
// Estructura básica
if (condición) {
    // código si es verdadero
} else {
    // código si es falso
}

// If-else encadenado
if (condición1) {
    // código si condición1 es verdadera
} else if (condición2) {
    // código si condición2 es verdadera
} else {
    // código si ninguna condición es verdadera
}
```

### Switch
```java
switch (variable) {
    case valor1:
        // código
        break;
    case valor2:
        // código
        break;
    default:
        // código por defecto
}

// Switch moderno (Java 14+)
switch (variable) {
    case valor1 -> // código;
    case valor2 -> // código;
    default -> // código por defecto;
}
```

### Loops
```java
// For básico
for (int i = 0; i < 5; i++) {
    // código
}

// While
while (condición) {
    // código
}

// Do-While
do {
    // código
} while (condición);

// For-each
for (Tipo elemento : colección) {
    // código
}
```

## Ejemplos Prácticos

### Ejemplo Básico: Calculadora de Notas
```java
public class CalificacionEstudiante {
    public String obtenerCalificacion(int puntuacion) {
        if (puntuacion >= 90) {
            return "A";
        } else if (puntuacion >= 80) {
            return "B";
        } else if (puntuacion >= 70) {
            return "C";
        } else if (puntuacion >= 60) {
            return "D";
        } else {
            return "F";
        }
    }
}
```

### Ejemplo Avanzado: Sistema de Autenticación
```java
public class SistemaAutenticacion {
    private static final int MAX_INTENTOS = 3;
    
    public enum ResultadoLogin {
        EXITO, FALLIDO, BLOQUEADO
    }
    
    public ResultadoLogin autenticarUsuario(String usuario, String password) {
        int intentos = 0;
        boolean usuarioBloqueado = false;
        
        while (intentos < MAX_INTENTOS && !usuarioBloqueado) {
            if (validarCredenciales(usuario, password)) {
                return ResultadoLogin.EXITO;
            }
            
            intentos++;
            if (intentos >= MAX_INTENTOS) {
                bloquearCuenta(usuario);
                usuarioBloqueado = true;
            }
        }
        
        return usuarioBloqueado ? ResultadoLogin.BLOQUEADO : ResultadoLogin.FALLIDO;
    }
    
    private boolean validarCredenciales(String usuario, String password) {
        // Lógica de validación
        return false;
    }
    
    private void bloquearCuenta(String usuario) {
        // Lógica de bloqueo
    }
}
```

## Casos de Uso Reales

### Ejemplo: Sistema de Carrito de Compras
```java
public class CarritoCompras {
    private List<Producto> productos = new ArrayList<>();
    private static final double DESCUENTO_VIP = 0.15;
    private static final double DESCUENTO_REGULAR = 0.05;

    public double calcularTotal(Usuario usuario) {
        double total = 0;
        
        // Calcular subtotal
        for (Producto producto : productos) {
            total += producto.getPrecio() * producto.getCantidad();
        }
        
        // Aplicar descuentos según tipo de usuario
        switch (usuario.getTipo()) {
            case VIP -> {
                double descuento = total * DESCUENTO_VIP;
                total -= descuento;
            }
            case REGULAR -> {
                if (total > 100) {
                    double descuento = total * DESCUENTO_REGULAR;
                    total -= descuento;
                }
            }
            default -> {} // Sin descuento
        }
        
        // Verificar cupones
        if (usuario.tieneCuponActivo()) {
            aplicarCupon(usuario.getCupon());
        }
        
        return total;
    }
    
    private void aplicarCupon(Cupon cupon) {
        // Lógica de aplicación de cupones
    }
}

class Producto {
    private double precio;
    private int cantidad;
    
    // Getters y setters
    public double getPrecio() { return precio; }
    public int getCantidad() { return cantidad; }
}

class Usuario {
    private TipoUsuario tipo;
    private Cupon cupon;
    
    public TipoUsuario getTipo() { return tipo; }
    public boolean tieneCuponActivo() { return cupon != null && cupon.esValido(); }
    public Cupon getCupon() { return cupon; }
}

enum TipoUsuario {
    VIP, REGULAR, NUEVO
}

class Cupon {
    public boolean esValido() {
        // Lógica de validación
        return true;
    }
}
```
