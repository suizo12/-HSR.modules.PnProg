-Fragen
===================

##Grundlage
1.1 Was ist der Unterschied zwischen User-Level Threads und Kernel Level Threads?

1.2 Zeichne die Zusammenhänge zwischen den drei möglichen Threds Zustände auf.
![Thread Zustände](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/threadzustaende.png)
	
1.3 Was sind Deamon Threads?

1.4 Wie kann die JVM Terminiert werden?
- Wenn keine Threads mehr vorhanden sind.
- System.exit()
- Runtime.exit()

1.5 Wie wird ein Thread in Java erstellt?
```java
//thread implementation
class SimpleThread extends Thread {
	@Override
	public void run() {
		// threadbehavior
	}
}

//create thread
Thread myThread= new SimpleThread();
//start thread
myThread.start();

//-------------------------------------------//
//Runnable Implementation
class SimpleLogic implements Runnable{
	@Override
	public void run() {
		// threadbehavior
	}
}

//create thread
Thread myThread= new Thread(new SimpleLogic());
//start thread
myThread.start();

//-------------------------------------------//
//Ad-Hoc implementation entspricht Thread implementation
Thread myThread= new Thread(() -> {
	// threadbehavior
});
//start thread
myThread.start();
```

1.6 Welche Ausgabe ergibt den folgenden Codeabschnitt?
```java
class SimpleThread extends Thread {
	private String label;
	private intnofIt;
	publicSimpleThread(String label, intnofIt) {
		this.label= label; 
		this.nofIt= nofIt;
	}
	@Override
	public void run() {
		for (int i = 0; i < nofIt; i++) {
			System.out.println(i + " " + label);
		}
	}
}

public class MultiThreadTest{
	public static void main(String[] args) {
		new SimpleThread("A", 10).start();
		new SimpleThread("B", 10).start();
		System.out.println("mainfinished");
	}
}
```

Es wird "A 1"…"A 10" und "B 1" … "B 10" ausgegeben die Reihenfolge ist nicht deterministisch. "main finished" kann irgendwann kommen.

1.7 Was passiert wenn anstatt start() run() aufgerufen wird?
Deterministischer Aufruf
"main finished
A1 
A2
A3
…
A10
B1
B2
…
B10"

1.8 Stimmt die folgende Aussage: "Treads in einer JVM laufen immer Deterministisch"
Meistens,jedoch nicht immer, Threads laufen ohne Vorkehrungen beliebig verzahnt oder parallel. Viele JVMs realisieren einzelne System Outputs ohne Verzahnung/Thread-Fehler -aber es ist nicht spezifiziert!

1.9 Was versteht man unter Explizit bzw. Implizit final. Gebe ein Beispiel für jedes:
```java
	//Parameter are implizit final	
	static void startMyThread2(String label, int nofIt) {
		new Thread(() -> {
			for (int i = 0; i < nofIt; i++) {
				System.out.println(i + " " + label);
			}
		}).start();
	}

	//Parameter are Explizit final
	static void startMyThread(final String label, final int nofIt) {
		new Thread() {
			@Override
			public void run() {
				for (int i = 0; i < nofIt; i++) {
					System.out.println(i + " " + label);
				}
			}
		}.start();
	}
```

1.10 Für was wird Thread.yield() hauptzächlich verwendet? 
Thread.yield() wird für das Testen verwendet.

1.11 Was passiert mit dem Thread wenn Thread.sleep(2000) aufgerufen wird?
Thread geht in den Wartezustand für 2sec. Anschliessend ist er wieder Ready.

1.12 Welche Exception wirft Thread.sleep(miliseconds)?
InterruptedException

1.13 Wie sieht ein Thread Lifecycle aus?
![Thread Lifecycle](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/thread_lifecycle.png)

1.14 Warum kann ein Thread nach Terminated() nicht sofot durch den Garbage Collecten aufgeräumt werden?
Da der Thread ein Normale Java Classe ist, die evt. ein Resultat hat.

1.15 Was für Möglichkeiten hat ein laufender ("alive") Thread?
![Thread Lifecycle](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/thread_alive.png)

1.16 Wie kann man einem Thread einem Spezifisch Namen geben? Was ist der Default name?
- Thread.setName(String name), Thread.getName()
- Thread(String name), Thread(Runnable r, String name)
- Default: "Thread-0", "Thread-1", etc...
