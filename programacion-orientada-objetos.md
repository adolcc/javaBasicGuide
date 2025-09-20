# Programación Orientada a Objetos en Java

## Índice
1. [Conceptos Básicos](#conceptos-básicos)
   - [Clases y Objetos](#clases-y-objetos)
   - [Modificadores de Acceso](#modificadores-de-acceso)
   - [Static](#static)
2. [Herencia y Polimorfismo](#herencia-y-polimorfismo)
3. [Principios SOLID](#principios-solid)
4. [Ejemplos Prácticos](#ejemplos-prácticos)
   - [Básicos](#ejemplos-básicos)
   - [Avanzados](#ejemplos-avanzados)
5. [Caso de Uso Real](#caso-de-uso-real)

## Conceptos Básicos

### Clases y Objetos
Una clase es una plantilla para crear objetos, y un objeto es una instancia de una clase.
```java
public class Persona {
    // Atributos
    private String nombre;
    private int edad;
    
    // Constructor
    public Persona(String nombre, int edad) {
        this.nombre = nombre;
        this.edad = edad;
    }
    
    // Métodos
    public void saludar() {
        System.out.println("Hola, soy " + nombre);
    }
}
```

### Modificadores de Acceso

#### public
- Accesible desde cualquier clase
- Uso común: métodos que necesitan ser accedidos desde fuera de la clase
```java
public void metodoPublico() {
    // Código accesible desde cualquier parte
}
```

#### private
- Solo accesible dentro de la misma clase
- Uso común: atributos de clase y métodos internos
```java
private String datoSensible;
private void metodoInterno() {
    // Código solo accesible dentro de la clase
}
```

#### protected
- Accesible en el mismo paquete y subclases
- Uso común: métodos que necesitan ser heredados
```java
protected void metodoProtegido() {
    // Código accesible en el paquete y subclases
}
```

### Static
- Pertenece a la clase, no a las instancias
- Se comparte entre todas las instancias
```java
public class Contador {
    private static int totalInstancias = 0;
    
    public Contador() {
        totalInstancias++;
    }
    
    public static int getTotalInstancias() {
        return totalInstancias;
    }
}
```

## Herencia y Polimorfismo

### Herencia
```java
// Clase base
public abstract class Animal {
    protected String nombre;
    
    public Animal(String nombre) {
        this.nombre = nombre;
    }
    
    public abstract void hacerSonido();
}

// Clase derivada
public class Perro extends Animal {
    public Perro(String nombre) {
        super(nombre);
    }
    
    @Override
    public void hacerSonido() {
        System.out.println("¡Guau!");
    }
}
```

### Polimorfismo
```java
public class EjemploPolimorfismo {
    public void demostrarPolimorfismo() {
        Animal perro = new Perro("Bobby");
        Animal gato = new Gato("Michi");
        
        // Mismo método, diferente comportamiento
        perro.hacerSonido(); // Imprime: ¡Guau!
        gato.hacerSonido();  // Imprime: ¡Miau!
    }
}
```

## Principios SOLID

### S - Single Responsibility (Responsabilidad Única)
```java
// Bien: Cada clase tiene una única responsabilidad
public class UsuarioRepository {
    public void guardarUsuario(Usuario usuario) {
        // Lógica para guardar en base de datos
    }
}

public class EmailService {
    public void enviarEmail(String destino, String mensaje) {
        // Lógica para enviar email
    }
}
```

### O - Open/Closed (Abierto/Cerrado)
```java
public interface CalculadorDescuento {
    double calcularDescuento(double monto);
}

public class DescuentoRegular implements CalculadorDescuento {
    @Override
    public double calcularDescuento(double monto) {
        return monto * 0.1;
    }
}

public class DescuentoVIP implements CalculadorDescuento {
    @Override
    public double calcularDescuento(double monto) {
        return monto * 0.2;
    }
}
```

### L - Liskov Substitution (Sustitución de Liskov)
```java
public class Ave {
    public void comer() { }
}

public class AveVoladora extends Ave {
    public void volar() { }
}

public class Pinguino extends Ave {
    // No implementa volar porque no todos las aves vuelan
}
```

### I - Interface Segregation (Segregación de Interfaces)
```java
// Mejor tener interfaces pequeñas y específicas
public interface Nadador {
    void nadar();
}

public interface Volador {
    void volar();
}

public class Pato implements Nadador, Volador {
    @Override
    public void nadar() { }
    
    @Override
    public void volar() { }
}
```

### D - Dependency Inversion (Inversión de Dependencias)
```java
public interface NotificacionService {
    void enviarNotificacion(String mensaje);
}

public class EmailNotificacion implements NotificacionService {
    @Override
    public void enviarNotificacion(String mensaje) {
        // Enviar por email
    }
}

public class SMSNotificacion implements NotificacionService {
    @Override
    public void enviarNotificacion(String mensaje) {
        // Enviar por SMS
    }
}
```

## Ejemplos Prácticos

### Ejemplos Básicos
```java
public class CuentaBancaria {
    private double saldo;
    private static final double TASA_INTERES = 0.05;
    
    public CuentaBancaria(double saldoInicial) {
        this.saldo = saldoInicial;
    }
    
    public void depositar(double monto) {
        if (monto > 0) {
            saldo += monto;
        }
    }
    
    public boolean retirar(double monto) {
        if (monto <= saldo && monto > 0) {
            saldo -= monto;
            return true;
        }
        return false;
    }
}
```

### Ejemplos Avanzados
```java
public class SistemaGestionEmpleados {
    public abstract class Empleado {
        protected String nombre;
        protected double salarioBase;
        
        public abstract double calcularSalario();
    }
    
    public class EmpleadoTiempoCompleto extends Empleado {
        private double bono;
        
        @Override
        public double calcularSalario() {
            return salarioBase + bono;
        }
    }
    
    public class EmpleadoTemporal extends Empleado {
        private int horasTrabajadas;
        private double tarifaPorHora;
        
        @Override
        public double calcularSalario() {
            return horasTrabajadas * tarifaPorHora;
        }
    }
}
```

## Caso de Uso Real
```java
// Sistema de Gestión de Biblioteca
public interface Prestable {
    boolean prestar();
    boolean devolver();
    boolean estaDisponible();
}

public abstract class MaterialBiblioteca {
    protected String titulo;
    protected String codigo;
    protected boolean disponible;
    
    // Constructor y métodos comunes
}

public class Libro extends MaterialBiblioteca implements Prestable {
    private String autor;
    private String isbn;
    
    @Override
    public boolean prestar() {
        if (disponible) {
            disponible = false;
            return true;
        }
        return false;
    }
    
    @Override
    public boolean devolver() {
        if (!disponible) {
            disponible = true;
            return true;
        }
        return false;
    }
    
    @Override
    public boolean estaDisponible() {
        return disponible;
    }
}

public class GestorPrestamos {
    private List<Prestable> materialesPrestados;
    private Map<String, Usuario> usuarios;
    
    public boolean realizarPrestamo(String codigoMaterial, String idUsuario) {
        Prestable material = buscarMaterial(codigoMaterial);
        Usuario usuario = usuarios.get(idUsuario);
        
        if (material != null && usuario != null && material.estaDisponible()) {
            if (material.prestar()) {
                registrarPrestamo(material, usuario);
                return true;
            }
        }
        return false;
    }
    
    private void registrarPrestamo(Prestable material, Usuario usuario) {
        materialesPrestados.add(material);
        // Registrar en base de datos, enviar notificación, etc.
    }
}
```