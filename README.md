# Hilos Virtuales en Java

## Introducción

Los **hilos virtuales**, introducidos en Java 19 y estabilizados en Java 21, son hilos ligeros gestionados por la JVM. Permiten ejecutar miles o millones de tareas concurrentes con un bajo consumo de recursos, siendo ideales para aplicaciones con alta latencia (como servidores web o servicios de red).

---

## Conceptos Fundamentales

### Hilos Portadores vs. Hilos Virtuales

- **Hilos Portadores**: Son hilos del sistema operativo que ejecutan múltiples hilos virtuales. La JVM los gestiona en grupos pequeños, como `ForkJoinPool`.
- **Hilos Virtuales**: Hilos ligeros, fáciles de crear y destruir, ideales para operaciones bloqueantes como acceso a red o archivos.

### Bloqueo y Escalabilidad

Cuando un hilo virtual se bloquea (por ejemplo, esperando una respuesta HTTP), la JVM lo suspende automáticamente, liberando el hilo portador para otras tareas. Esto permite manejar miles de tareas concurrentes sin saturar el sistema.

### Creación y Gestión

- Usar `Thread.ofVirtual().start(...)` para lanzar hilos virtuales directamente.
- O usar un `ExecutorService` con:
  ```java
  Executors.newVirtualThreadPerTaskExecutor()
  ```
- No requiere configuraciones complejas ni manejo explícito de pools.

### Concurrencia Estructurada

Organiza tareas concurrentes en jerarquías usando `StructuredTaskScope` (disponible como preview), lo que permite un control claro de la ejecución y finalización de subtareas.

---

## Ventajas

- Ejecuta miles de tareas con mínima memoria y CPU.
- Código más simple: se pueden usar operaciones bloqueantes directamente sin recurrir a APIs asíncronas.
- Compatible con bibliotecas tradicionales de Java.

---

## Desventajas

- No mejora el rendimiento en tareas intensivas en CPU (como cálculos pesados).
- La depuración puede volverse compleja al manejar grandes cantidades de hilos virtuales.
- Aunque ligeros, un exceso de hilos virtuales puede afectar el rendimiento por el uso acumulado de memoria.

---

## Escenarios Ideales

- Servidores web o APIs que manejan múltiples solicitudes simultáneas.
- Aplicaciones que realizan muchas operaciones de entrada/salida (red, archivos, bases de datos).
- Reemplazo de código asíncrono complejo (por ejemplo, con `CompletableFuture`).

---

## Comparación con Otros Lenguajes

| Lenguaje     | Equivalente                      | Notas                                                                 |
|--------------|----------------------------------|-----------------------------------------------------------------------|
| Go           | Goroutines                       | Muy ligeras y gestionadas por el runtime de Go.                       |
| Kotlin       | Coroutines                       | Ideales para tareas asíncronas, muy usadas en Android.                |
| Node.js      | Event loop (bucle de eventos)    | Usa un solo hilo, menos flexible para concurrencia masiva.            |


    
        package ec.edu.utpl.carreras.computacion.pga.clases.s1;
    
        import java.io.*;
        import java.util.ArrayList;
        import java.util.List;
    
        public class UrlAnalyzerApp {
            public static void main(String[] args) {
            String inputPath = "data/urls_parcial1.txt";
            String outputPath = "data/resultados.csv";

        List<String> urls = new ArrayList<>();
        try (BufferedReader reader = new BufferedReader(new FileReader(inputPath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                urls.add(line.trim());
            }
        } catch (IOException e) {
            System.out.println("Error leyendo archivo: " + e.getMessage());
            return;
        }

        List<UrlAnalyzer> analyzers = new ArrayList<>();
        List<Thread> threads = new ArrayList<>();

        for (String url : urls) {
            UrlAnalyzer analyzer = new UrlAnalyzer(url);
            Thread t = Thread.startVirtualThread(analyzer);  // Virtual thread aquí
            threads.add(t);
            analyzers.add(analyzer);
        }

        for (Thread t : threads) {
            try {
                t.join();
            } catch (InterruptedException e) {
                System.out.println("Hilo interrumpido: " + e.getMessage());
            }
        }

        try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputPath))) {
            writer.write("URL,EnlacesInternos\n");
            for (UrlAnalyzer analyzer : analyzers) {
                writer.write(analyzer.getUrl() + "," + analyzer.getInternalLinks() + "\n");
            }
        } catch (IOException e) {
            System.out.println("Error escribiendo archivo: " + e.getMessage());
        }
        }
        }



        package ec.edu.utpl.carreras.computacion.pga.clases.s1;
            
            import org.jsoup.Jsoup;
            import org.jsoup.nodes.Document;
            import org.jsoup.nodes.Element;
            import org.jsoup.select.Elements;
        
            import java.net.URL;
        
            public class UrlAnalyzer implements Runnable {
              private final String url;
              private int internalLinks = 0;
        
            public UrlAnalyzer(String url) {
                this.url = url;
            }
        
            @Override
            public void run() {
                try {
                    Document doc = Jsoup.connect(url).get();
                    URL base = new URL(url);
                    String host = base.getHost();
        
                    Elements links = doc.select("a[href]");
                    for (Element link : links) {
                        String absHref = link.absUrl("href");
                        if (!absHref.isEmpty()) {
                            URL linkUrl = new URL(absHref);
                            if (linkUrl.getHost().equalsIgnoreCase(host)) {
                                internalLinks++;
                            }
                        }
                    }
        
                } catch (Exception e) {
                    System.out.println("Error analizando URL " + url + ": " + e.getMessage());
                }
            }
        
            public String getUrl() {
                return url;
            }
        
            public int getInternalLinks() {
                return internalLinks;
            }
            }
        

Promt usado:
Imagina que eres un expero en programacion de Hilos en java, pero ahora necesitas implementar hilos virtuales en tu programa de java, este mismo debe quitar el hilo, y remplazarlo por hilos virtuales, como resultado quiero que me des el codigo corregido con vitual threads y una explicacion de que fue lo que se remplazo, y por que el remplazar.Tambien beneficios de esto ydatos extra que me sean utilies para entender mejor el caso.
