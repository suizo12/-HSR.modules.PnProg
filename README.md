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

//	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	--//
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

//	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	-	--//
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

1.17 Was geschieht wenn einen Thread.join() aufgerufen wird?
Mit join() wird auf einen anderen Thread gewartet (er wird solange blockiert bis der zweite Thread fertig ist). 

1.18 Was ist die Ausgabe des folgenden Abschnittes
```java
int nofIt=100;
		Thread threadA= new Thread(() -> {
			for (int i = 0; i < nofIt; i++) {
				System.out.println(i + " " + "A");
			}
		});
		Thread threadB= new Thread(() -> {
			for (int i = 0; i < nofIt; i++) {
				System.out.println(i + " " + "B");
			}
		});
		System.out.println("Threads start");
		threadA.start();
		threadB.start();
		try {
			threadA.join();
			threadB.join();
			System.out.println("Threads joined");
		} catch (InterruptedException e) { }
```

Ausgabe:
Threads start
0 [A|B]
1 [A|B]
...
100 [A|B]
Thread joined

1.19  Was passiert bei Thread.currentThread().join()?
Nichts sinnvolles, da der jetztige Thread auf sich selbst warten würde.

##Monitor
2.1 Was ist eine Race Condition?

2.2 Erstelle eine Tabelle damit beim folgenden Code eine Race Condition beweisen kannst.
```java
class BankAccount{
private int balance= 0;
	public void deposit(int amount) {
		this.balance+= amount;
	}
}
```
Thread 1 | Balance | Tgread 2
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
read balance => reg(reg== 0) | 0 |
| 0 | read balance => reg(reg== 0)
reg= reg+ amount (reg== 100) | 0 | 
 | 0 | reg= reg+ amount (reg== 50)
write reg=> balance  | 100
 | 50 *Erwarte: 150* | write reg=> balance
 
2.3 Was ist ein Kirtischer Abschnitt? Zeige den Kritischer abschnitt in der Klasse BankAccount:
```java
public void deposit(intamount) {
	//enter criticalsection
	this.balance+= amount;
	//exit criticalsection
}
```

2.4 Was geht am folgendem Beispiel nicht?
```java
class BankAccount{
	private int balance= 0;
	private bool eanlocked= false;
	public void deposit(int amount) {
		while(locked) { } // busy waiting loop locked= true; 
		this.balance+= amount;// enter critical section
		locked= false;// exit critical section
	}
}
```
Schlaufe und Zuweisung ist nicht atormar
Thread 1 | locked | Tgread 2
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
Read locked in loop(locked== false) | false | Read locked in loop(locked== false)
Write locked= true | true | Write locked= true
Critical section |  | Critical section

2.5 Gegeben sein eine Klasse mit n Sychronisierten Methoden. Ist es möglich das zwei Methoden N der gleichen Instanz gleichzeitig ausgeführt werden?
Nein, nur ein Thread kann eine der synchronisierten Methoden in derselben Instanz zur gleichen Zeit ausführen.

2.6 Was ist ein Synchronized Block Statement?
```java
void someMethod(){
	//Synchronized Block
	synchronized(this){
		//Synchronized
	}
}
```

2.7 Wie kann der folgende Programmcode mit Synchronized Block geschrieben werden?
```java
public class Test {
	synchronized void f() { ... }
	static synchronized void g() { ... }
}
```

```java
public class Test {
void f() {
	synchronized(this) {... }
}
static void g() {
	synchronized(Test.class) {... }
	}
}
```

2.8 Warum geht der folgende Code nicht?
```java
class BankAccount {
	private int balance = 0;

	public synchronized void withdraw(int amount) throws InterruptedException {
		while (amount > this.balance) {
			Thread.sleep(1);
		}
		this.balance -= amount;
	}

	public synchronized void deposit(int amount) {
		this.balance += amount;
	}
}
```

Deadlock wenn withdraw condition "amount > this.balance" nicht erfüllt ist. kommt der Thread nicht aus der aktuellen Schlaufe nicht mehr raus.

2.9 Zeichne, erkläre und Programmiere das Prinzip des Monitors:
Interner gegenseitiger Auschluss. Nur ein Thread operiert gleichzeitig im Monitor.
Dabei werden zwischen zwei Status unterschieden. Die einen Threads warten auf ein "Signal" (notify(), notifyAll()).
Die anderen warten auf den Eintritt in den Monitor. (synchronized key word)
![Monitor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/monitor.png)

```java
class BankAccount {
	private int balance = 0;

	public synchronized void withdraw(int amount) throws InterruptedException {//Trettet in den Monitor ein
		while (amount > balance) {
			wait(); //wartet auf Bedinung
		}
		balance -= amount;
	}// Verlassen des Monitors

	public synchronized void deposit(int amount) {
		balance += amount;
		notifyAll();//Signalisiert Bedinung & Wecket alle im Monitor wartende Threads
	}
}
```

2.10 Was passiert bei der wait() Methode?
1. In Warteraum gehen
2. Monitor freigeben
3. Invaktiv bis zum Wartesignal
4. Monitor neu beziehen

2.11 Was passiert bei der notifyAll() Methode?
1. Weckt alle Threads in wait()
2. Behält Monitor und macht weiter

2.12 Was ist der Unterschied zwischen notify() und notifyAll()?
- notify()wecket einen beliebigen wartenden Thread im Monitor auf
- notifyAll()weckt alle im Monitor wartende Threads auf
- Kein Effekt, falls kein Thread wartet

2.13 Warum muss ein eine Schlaufe um die wait() Methode sein? while ( Bedinung ) { wait(); }
Weil die Bedingung nicht zwingen erfüllt sein muss.

2.14 Was ist Speziell an den Methoden wait(), notify() und notifyAll()?
Sie dürfen nur in einem Synchronized Kontext verwendet werden. Ansonsten IllegalMonitorStateException. 

2.15 Was ist beim folgedem Beispiel falsch?
```java
class BoundedBuffer<T> {
	private Queue<T> queue = new LinkedList<>();
	int limit; // initialize in constructor
	public synchronized void put(T x) throws InterruptedException{
		if (queue.size() == limit) {
			wait(); // await non-full
		}
		queue.add(x);
		notify();// signal non-empty
	}
	
	public synchronized T get() throws InterruptedException{
		if (queue.size() == 0) {
			wait(); // await non-empty
		}
		T x = queue.remove();
		notify();// signal non-full
		return x;
	}
}
```

1. Kein while um wait()
```java
while(!condition) {
	wait();
}
```
2. Single anstatt notifyAll()
```java
notifyAll();
```

2.16 Erkläre warum der Monitor ein Effizienz Problem hat?
- Beim mehreren Wartebedingungen
	- notifyAll(): Wecke prophylaktisch immer alle
	- Alle treten nacheinander in Monitor ein
	- Für einen ist Bedingung erfülllt
	- Andere rufen wieder wait() auf 
- Viele Kontext-Wechsel
- Hohe Synchronisationskosten

2.17 Erläutere warum der Monitor nicht "fair" ist:
- Java: Keine garantierte FIFO-Warteschlange
- Signal-and-Continue
	- Theoretisch möglich, dass ein Thread ewig von anderen Threads überholt wird.

2.18 
```java
/**
* Braucht es die wait()-Schlaufe
* Reicht ein notify()?
* Turnstile = Drehkreuz
**/
public class Turnstile {
	private boolean open = false;
	public synchronized void pass()
			throws InterruptedException{
		while(!open) { wait(); }
		open = false;
	}
	
	public synchronized void open() {
		open = true;
		notify(); // ornotifyAll() ?
	}
}
```
Ja es bruacht die wait()-Schlaufe:
Thread | Method -> Status | Waiting
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
T1 | pass() -> wait() | T1 
T2 | open()-> notify() | T1
T3 |  drausen -> noch kein status | T1, T3
T2 | mit notify kann T1 oder T3 aufrufen. -> T3 wird geweckt | T1 
T3 | pass() -> setzt open auf false | T1
T1 | setzt open wieder auf false | 

notify() genügt da es nur eine semantische Bedinung gibt.

##Semaphore
3.1 Zeichne das Prinzip vom Semaphor auf? Erkläre was die Methoden acquire() und release() bewirken.
![Semaphor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/semaphor.png)

3.2 Was ist ein Binären Semaphor?
```java
//Sempahor mit einem "N"
new Semaphor(1);
```

3.3 Wie wird ein Fairen Sempahor instatiert?
```java
new Semaphor(N, true); //Defaut ist unfair
```

3.4 Die folgende Klasse wurde mit dem Monitor Realisiert. Schreibe diese um das sie mit fairen Semaphor geht.
```java
class BoundedBuffer<T> {
	private Queue<T> queue = new LinkedList<>();
	int limit; // initialize in constructor
	
	public synchronized void put(T x) throws InterruptedException{
		while(queue.size() == limit) {
			wait(); // await non-full
		}
		queue.add(x);
		notifyAll(); // signal non-empty
	}

	public synchronized T get() throws InterruptedException{
		while (queue.size() == 0) {
			wait();// await non-empty
		}
		T item = queue.remove();
		notifyAll(); // signal non-full
		return item;
	}
}
```

Schritt 1 Semaphor Anwendung
![Semaphor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/semaphor1.png)
Schritt 2 Synchronized ersetzten
![Semaphor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/semaphor2.png)

3.5 Wie wird notify(), wait(), while um wait() und synchonized mit semaphoren ersetzt?
Monitor | Semaphor
	-	-	-	-	-	- | 	-	-	-	-	-	-
notifyAll() | release(), gibt Resource frei, benarchritig Wartende.
wait() | acquire(), Bezieht freie Ressource.
while um wait() | acquire(), Wartet wenn keine Ressource verfügbar ist.
synchronized | Binare Semaphore: Anstatt synchronized wird der Kritischer Block mit binarSemaphore.acquire(); ... binarSemaphore.release(); umgeben.

3.6 Wie wird mit "Lock & Conditions" den Monitor umgesetzt?
-Lock-Objekt: Sperre für Eintritt in Monitor
	- Äussere Warteliste
- Condition-Objekt: Wait& Signal für bestimmte Bedingung
	- Innere Warteliste

3.7 Zeichne die UML Struktur des Lock & Condition"
![Lock & Condition](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/lc_uml.png)

3.8 Wie wird fairness bei Lock & Condition aktiviert?
new ReentrantLock(true); // default ist false

3.9 Wie wird die Klasse Boundbuffer mit Lock & Condition umgesetzt?
```java
class BoundedBuffer<T> {
	private Lock monitor = new ReentrantLock(true);
	private Condition nonFull= monitor.newCondition();
	private Condition nonEmpty= monitor.newCondition();
	
	public void put(T item) throws InterruptedException{
		monitor.lock();
		try {
			//Schlaufe wegen Überholungsproblem & Spurious Wakeup
			while (queue.size() == Capacity) {
				nonFull.await();
			}
			queue.add(item);
			nonEmpty.signal();
		} finally { //finaly falls eine exception geworfen wird, wird der lock noch freigegeben.
			monitor.unlock();
		}
	}
	
	public T get() throws InterruptedException{
		monitor.lock();
		try {
			while (queue.size() == 0) { nonEmpty.await();}
			T item = queue.remove();
			nonFull.signal(); //Gezieltes Einzel-Signal für diese Bedingung
			return item;
		} finally { monitor.unlock();}
	}
}
```

3.10 Welche Read - Write zugriffe können parallel geschene?
Parallel | Read | Write
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
Read | Ja | Nein
Write | Nein | Nein

3.11 Bei Read-Write Lock & Condition hat nur der Write-Lock Conditions (3 Woche Folie 27) Warum ist das so?
![Read-Write lock](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/wr_condition.png)

3.12 Was sind Synchronisationspunkte? Gebe drei Beispiele dazu.
Synchronasionspunkte ist ein Ort wo etwas (Thread) auf den rest (andere Threads) wartet (synchronisiert).
Beispiele:
- Thread.join()
- Count Down Latch

3.13 Was sind CountDown Latch?
Ähnlich wie Semaphoren blockiert bei <= 0 jedoch Count Down Latch blockiert bei > 0.
Threads können runterzählen 
- countDown(): Zähler um 1 dekrementieren

3.14 Implementiere das folgende Beispiel mit CountDown Latches.
```java
Semaphore readyCars = new Semaphore(0);
Semaphore startAllows = new Semaphore(0);

//N Cars:							RaceControl:
readyCars.release(); 		-	-	-	--> 	readyCars.acquire(N);
startAllows.acquire();		<	-	-	-	--	startAllows.release(N);
```
```java
CountDownLatch carsReady = new CountDownLatch(N);
CountDownLatch startSignal = new CountDownLatch(1);

//N Cars:							RaceControl:
carsReady.countDown(); 		-	-	-	--> 	startSignal.await();
carsReady.await();		<	-	-	-	--	startSignal.countDown();
```

3.15 Zeichne ein Beispiel der CountDown Latch auf.
![Count Down Latch](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/ctl.png)

3.16 Wie funktioniert das Barriere prinzip.
-Anzahl Teilnehmer bei Konstruktorfestlegen
	-Nicht mehr änderbar
	-intgetParties()
-Passieren bei Barriere
	-int await()
	-Rückgabe: Anzahl noch fehlender Threads bei Barriere
	-- > 0: Warten
	-- == 0: Barriere öffnen und Wartende aufwecken
- «BrokenBarrier»
	- Problem: Exceptionin await, z.B. InterruptedException
	- Alle sind betroffen => BrokenBarrierException

3.17 Vergleich Latch mit Barriere.
![Latch vs Barrier](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/lb.png)
- | Latch | Barriere
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
Initialisierung | new CountDownLatch(N); | new CyclicBarrier(N);
N = Zähler bedeutet? | Initialwert für den Counter | Anzahl 
Wie wird n runtegezählt? | .countDown(); | .await(); 
Geht weiter wenn? | N <= 0 ist | Wenn N Threads await() aufgerufen haben.
Wiederverwendbarkeit | Nein | Ja

3.18 Wie funktioniert das Phaser Prinzip?
- Verallgemeinerte CyclicBarrier
	- arriveAndAwaitAdvance(): Barriere passieren
	- Kann Teilnehmer später anmelden / abmelden
	-- register()bzw. arriveAndDeregister()
	-- Wird bei nächster Warterunde wirksam
![Phaser](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/phaser.png)

3.19 Wie funktioniert das Rendez-Vous Prinzip?
- Barriere mit Informationsaustausch
	- Spezialfall: Nur 2 Parteien
- Zwei Threads treffen sich und tauschen Objekte aus
	- Ohne Austausch: newCyclicBarrier(2)
	- Mit Austausch: Exchanger.exchange(something)
- V exchange(Vx)
	- Blockiert, bis anderer Thread auch exchange() aufruft
	- Liefert Argument x des jeweils anderen Threads
![Barrier vs Exchanger](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/be.png)

3.20 Gebe ein Beispiel für das Rendez-Vous Prinzip.
```java
Exchanger<Integer> exchanger = new Exchanger<>();
for (intk = 0; k < 2; k++) {
	new Thread(() -> {
		for (int in = 0; in < 5; in++) {
			try {
				intout = exchanger.exchange(in);
				System.out.println(Thread.currentThread().getName() + " got " + out);
			} catch (InterruptedExceptione) { }
		}
	}).start();
}
```

3.21 Welche ausgabe wird bei 3.20 ausgegeben.
Ausgabe:
Thread-0 got 0
Thread-1 got 0
Thread-0 got 1
Thread-1 got 1
Thread-0 got 2
Thread-1 got 2
Thread-1 got 3
Thread-0 got 3
Thread-0 got 4
Thread-1 got 4

3.22 Was ist der Unterschied zwischen Java Monitor vs Lock & Condition?
- | Java Monitor | Lock & Condition
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
Inneren Warteraum | 1 | Mehrere
Implementierung | API | In java integriert

3.23 Was ist der Unterschied zwischen Semaphore vs CountDownLatch?
- | Semaphore | CountDownLatch
	-	-	-	-	-	- | 	-	-	-	-	-	- | 	-	-	-	-	-	--
Blocking Counter(N) bei | Wenn alle N vergeben sind (N = 0) | Solange N noch verfügbar sind (N > 0)
Counter Eigenschaft | Kann erhöht werden | Kann nicht erhöht werden

## Gefahren der Nebenläufigkeit
4.1 Was sind Race Condition, Deadlocks, Starvation?
- Race Condition
	- Ungenügend synchronisierte Zugriffe auf gemeinsame Ressourcen
	- Zwei Stuffen
		- Data Races (low-level)
		- Semantisch höhere Race Condition (high-level)
- Deadlocks
	- Gegenseitiges Aussperren von Threads
- Starvation
	- Kontinuierliche Fortschrittsbehinderung eines Threads wegen Fairness-Probleme
Parallele
