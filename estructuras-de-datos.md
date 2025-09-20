# Estructuras de Datos en Java

## Índice
1. [Collections Framework](#collections-framework)
   - [List](#list)
   - [Set](#set)
   - [Map](#map)
2. [Genéricos](#genericos)
3. [Ejemplos Prácticos](#ejemplos-prácticos)
   - [Básicos](#ejemplos-básicos)
   - [Avanzados](#ejemplos-avanzados)
4. [Casos de Uso Reales](#casos-de-uso-reales)

## Collections Framework

### Concepto General
El Collections Framework de Java proporciona una arquitectura unificada para representar y manipular colecciones de objetos. Las principales interfaces son:
- `List`: Colección ordenada que permite elementos duplicados
- `Set`: Colección que no permite duplicados
- `Map`: Colección de pares clave-valor

### List
```java
// ArrayList: Implementación basada en arrays
List<String> arrayList = new ArrayList<>();
arrayList.add("Elemento 1");
arrayList.add("Elemento 2");

// LinkedList: Implementación basada en nodos enlazados
List<String> linkedList = new LinkedList<>();
linkedList.add("Elemento 1");
linkedList.add("Elemento 2");
```

### Set
```java
// HashSet: No mantiene orden
Set<String> hashSet = new HashSet<>();
hashSet.add("Elemento 1");
hashSet.add("Elemento 1"); // No se añade (duplicado)

// TreeSet: Mantiene orden natural
Set<String> treeSet = new TreeSet<>();
treeSet.add("B");
treeSet.add("A");
// Resultado: [A, B]
```

### Map
```java
// HashMap: No mantiene orden
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("Uno", 1);
hashMap.put("Dos", 2);

// TreeMap: Mantiene orden por clave
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("B", 2);
treeMap.put("A", 1);
// Resultado: {A=1, B=2}
```

## Genéricos

### Concepto
Los genéricos permiten crear clases, interfaces y métodos que pueden trabajar con diferentes tipos de datos mientras mantienen la type-safety.

```java
// Clase genérica
public class Contenedor<T> {
    private T contenido;
    
    public Contenedor(T contenido) {
        this.contenido = contenido;
    }
    
    public T getContenido() {
        return contenido;
    }
}

// Método genérico
public <T> void imprimirArray(T[] array) {
    for (T elemento : array) {
        System.out.println(elemento);
    }
}
```

## Ejemplos Prácticos

### Ejemplos Básicos

#### Gestión de Lista de Tareas
```java
public class ListaTareas {
    private List<String> tareas = new ArrayList<>();
    
    public void agregarTarea(String tarea) {
        tareas.add(tarea);
    }
    
    public void eliminarTarea(String tarea) {
        tareas.remove(tarea);
    }
    
    public List<String> obtenerTareasPendientes() {
        return new ArrayList<>(tareas);
    }
}
```

#### Conjunto de Números Únicos
```java
public class ConjuntoNumeros {
    private Set<Integer> numeros = new HashSet<>();
    
    public boolean agregarNumero(int numero) {
        return numeros.add(numero);
    }
    
    public boolean contieneNumero(int numero) {
        return numeros.contains(numero);
    }
}
```

### Ejemplos Avanzados

#### Cache Genérico con Expiración
```java
public class Cache<K, V> {
    private final Map<K, CacheEntry<V>> cache = new HashMap<>();
    
    public void put(K key, V value, long tiempoExpiracionMs) {
        cache.put(key, new CacheEntry<>(value, 
            System.currentTimeMillis() + tiempoExpiracionMs));
    }
    
    public V get(K key) {
        CacheEntry<V> entry = cache.get(key);
        if (entry == null) return null;
        
        if (System.currentTimeMillis() > entry.tiempoExpiracion) {
            cache.remove(key);
            return null;
        }
        
        return entry.valor;
    }
    
    private static class CacheEntry<V> {
        final V valor;
        final long tiempoExpiracion;
        
        CacheEntry(V valor, long tiempoExpiracion) {
            this.valor = valor;
            this.tiempoExpiracion = tiempoExpiracion;
        }
    }
}
```

## Casos de Uso Reales

### Sistema de Gestión de Biblioteca
```java
public class SistemaBiblioteca {
    private Map<String, List<Libro>> librosPorCategoria = new HashMap<>();
    private Set<String> usuariosRegistrados = new HashSet<>();
    private Map<String, List<Prestamo>> prestamosPorUsuario = new HashMap<>();

    public void agregarLibro(Libro libro) {
        librosPorCategoria.computeIfAbsent(libro.getCategoria(), 
            k -> new ArrayList<>()).add(libro);
    }

    public List<Libro> buscarPorCategoria(String categoria) {
        return new ArrayList<>(librosPorCategoria.getOrDefault(categoria, 
            new ArrayList<>()));
    }

    public boolean registrarPrestamo(String usuario, Libro libro) {
        if (!usuariosRegistrados.contains(usuario)) {
            return false;
        }

        Prestamo prestamo = new Prestamo(libro, LocalDate.now());
        prestamosPorUsuario.computeIfAbsent(usuario, 
            k -> new ArrayList<>()).add(prestamo);
        return true;
    }

    public List<Libro> obtenerLibrosVencidos() {
        List<Libro> librosVencidos = new ArrayList<>();
        LocalDate hoy = LocalDate.now();

        prestamosPorUsuario.values().stream()
            .flatMap(List::stream)
            .filter(p -> p.getFechaVencimiento().isBefore(hoy))
            .map(Prestamo::getLibro)
            .forEach(librosVencidos::add);

        return librosVencidos;
    }
}

class Libro {
    private String titulo;
    private String categoria;
    private String isbn;

    // Constructor y getters
}

class Prestamo {
    private Libro libro;
    private LocalDate fechaPrestamo;
    private static final int DIAS_PRESTAMO = 14;

    public Prestamo(Libro libro, LocalDate fechaPrestamo) {
        this.libro = libro;
        this.fechaPrestamo = fechaPrestamo;
    }

    public LocalDate getFechaVencimiento() {
        return fechaPrestamo.plusDays(DIAS_PRESTAMO);
    }

    public Libro getLibro() {
        return libro;
    }
}
```

Este ejemplo real demuestra:
- Uso de múltiples estructuras de datos
- Genéricos para type-safety
- Operaciones complejas con colecciones
- Manejo de datos relacionados
- Stream API para procesamiento de colecciones