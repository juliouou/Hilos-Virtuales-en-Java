##Hilos Virtuales en Java

Introducción
Los hilos virtuales, introducidos en Java 19 y estabilizados en Java 21, son hilos ligeros gestionados por la JVM. Permiten ejecutar miles o millones de tareas concurrentes con bajo consumo de recursos, ideales para aplicaciones con mucha espera, como servidores web.
Conceptos Fundamentales

Hilos Portadores vs. Hilos Virtuales:
Hilos Portadores: Hilos del sistema operativo que ejecutan múltiples hilos virtuales. La JVM los gestiona en un grupo pequeño (usualmente ForkJoinPool).
Hilos Virtuales: Hilos ligeros que la JVM crea y destruye fácilmente. Son perfectos para tareas que esperan (por ejemplo, operaciones de red).


Bloqueo y Escalabilidad: Cuando un hilo virtual se bloquea (por ejemplo, esperando una respuesta HTTP), la JVM lo suspende, liberando el hilo portador para otras tareas. Esto permite manejar miles de tareas sin consumir muchos recursos.
Creación y Gestión: 
Usa Thread.ofVirtual().start() o Executors.newVirtualThreadPerTaskExecutor() para crear hilos virtuales.
No necesitan configuraciones complejas como los hilos tradicionales.

Concurrencia Estructurada: Organiza tareas concurrentes jerárquicamente para que terminen juntas. Usa StructuredTaskScope (en vista previa) con hilos virtuales para un control más claro.

Ventajas:
Ejecuta miles de tareas con poca memoria y CPU.
Simplifica el código: usa operaciones bloqueantes sin APIs asíncronas.
Compatible con bibliotecas existentes de Java.


Desventajas:
No mejora tareas intensivas en CPU (por ejemplo, cálculos complejos).
Depurar muchos hilos virtuales puede ser complicado.
Consume algo de memoria si se crean en exceso.



Escenarios Ideales

Servidores web o APIs que manejan muchas solicitudes simultáneas.
Aplicaciones con tareas de entrada/salida (lectura de archivos, bases de datos).
Reemplazo de código asíncrono complejo (por ejemplo, CompletableFuture).

Comparación con Otros Lenguajes

Go (Goroutines): Hilos ligeros similares, gestionados por el runtime de Go.
Kotlin (Coroutines): Concurrencia ligera para tareas asíncronas, usada en Android.
Node.js: Usa un bucle de eventos de un solo hilo, menos flexible para concurrencia masiva.

Opinión Personal
Los hilos virtuales hacen que Java sea más competitivo para aplicaciones concurrentes, como servidores web. Simplifican el desarrollo y son fáciles de usar. En el futuro, con mejor soporte en frameworks como Spring y más optimizaciones, podrían convertirse en la opción estándar para aplicaciones de servidor en Java.



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


