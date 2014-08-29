Fragen
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
//thread implemetation
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
	public SimpleThread(String label, int nofIt) {
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
Deterministischer Aufruf:
- main finished
- A1 
- A2
- A3
- …
- A10
- B1
- B2
- …
- B10

1.8 Stimmt die folgende Aussage: "Treads in einer JVM laufen immer Deterministisch"
Meistens, jedoch nicht immer, Threads laufen ohne Vorkehrungen beliebig verzahnt oder parallel. Viele JVMs realisieren einzelne System Outputs ohne Verzahnung/Thread-Fehler -aber es ist nicht spezifiziert!

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
- Thread.yield() wird für das Testen verwendet.

1.11 Was passiert mit dem Thread wenn Thread.sleep(2000) aufgerufen wird?
- Thread geht in den Wartezustand für 2sec. Anschliessend ist er wieder Ready.

1.12 Welche Exception wirft Thread.sleep(miliseconds)?
- InterruptedException

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
Thread 1 | Balance | Thread 2
--------------- | --------------- | ---------------
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
	private boolean locked= false;
	public void deposit(int amount) {
		while(locked) { } // busy waiting loop locked= true; 
		this.balance+= amount;// enter critical section
		locked= false;// exit critical section
	}
}
```
Schlaufe und Zuweisung ist nicht atormar

Thread 1 | locked | Thread 2
--------------- | --------------- | ---------------
Read locked in loop(locked== false) | false | Read locked in loop(locked == false)
Write locked= true | true | Write locked = true
Critical section | - | Critical section

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

2.13 Warum muss eine Schlaufe um die wait() Methode sein (Vorlesung 2 Folie 36)? 
```java
while ( Bedinung ) { wait(); }
```
Weil die Bedingung nicht zwingen erfüllt sein muss, wenn er von einem anderen Thread aufgeweckt wird.

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
--------------- | --------------- | ---------------
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
--------------- | ---------------
notifyAll() | release(N), gibt N Resourcen frei, benarchritig N Wartende.
wait() | acquire() auf semaphore mit 0 Resourcen, Bezieht freie Ressource.
while um wait() | acquire(), Wartet wenn keine Ressource verfügbar ist.
synchronized | Binare Semaphore: Anstatt synchronized wird der Kritischer Block mit binarSemaphore.acquire(); ... binarSemaphore.release(); umgeben.

3.6 Wie wird mit "Lock & Conditions" den Monitor umgesetzt? Folie 22/23
- Lock-Objekt: Sperre für Eintritt in Monitor
	- Äussere Warteliste
- Condition-Objekt: Wait & Signal für bestimmte Bedingung
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
--------------- | --------------- | ---------------
Read | Ja | Nein
Write | Nein | Nein

3.11 Bei Read-Write Lock & Condition hat nur der Write-Lock Conditions (3 Woche Folie 27) Warum ist das so?
![Read-Write lock](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/wr_condition.png)

3.12 Was sind Synchronisationspunkte? Gebe drei Beispiele dazu. (Vorlesung 3 Seite 31/32)
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
- Anzahl Teilnehmer bei Konstruktorfestlegen
	- Nicht mehr änderbar
	- int getParties()
- Passieren bei Barriere
	- int await()
	- Rückgabe: Anzahl noch fehlender Threads bei Barriere
	-- > 0: Warten
	-- == 0: Barriere öffnen und Wartende aufwecken
- «BrokenBarrier»
	- Problem: Exceptionin await, z.B. InterruptedException
	- Alle sind betroffen => BrokenBarrierException

3.17 Vergleich Latch mit Barriere.
![Latch vs Barrier](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/lb.png)
- | Latch | Barriere
--------------- | --------------- | ---------------
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
- V exchange(V x)
	- Blockiert, bis anderer Thread auch exchange() aufruft
	- Liefert Argument x des jeweils anderen Threads
![Barrier vs Exchanger](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/be.png)

3.20 Gebe ein Beispiel für das Rendez-Vous Prinzip.
```java
Exchanger<Integer> exchanger = new Exchanger<>();
for (int k = 0; k < 2; k++) {
	new Thread(() -> {
		for (int in = 0; in < 5; in++) {
			try {
				int out = exchanger.exchange(in);
				System.out.println(Thread.currentThread().getName() + " got " + out);
			} catch (InterruptedException e) { }
		}
	}).start();
}
```

3.21 Welche ausgabe wird bei 3.20 ausgegeben.
- Ausgabe:
- Thread-0 got 0
- Thread-1 got 0
- Thread-0 got 1
- Thread-1 got 1
- Thread-0 got 2
- Thread-1 got 2
- Thread-1 got 3
- Thread-0 got 3
- Thread-0 got 4
- Thread-1 got 4

3.22 Was ist der Unterschied zwischen Java Monitor vs Lock & Condition?
- | Java Monitor | Lock & Condition
--------------- | --------------- | ---------------
Inneren Warteraum | 1 | Mehrere
Implementierung | API | In java integriert

3.23 Was ist der Unterschied zwischen Semaphore vs CountDownLatch?
- | Semaphore | CountDownLatch
--------------- | --------------- | ---------------
Blocking Counter(N) bei | Wenn alle N vergeben sind (N = 0) | Solange N noch verfügbar sind (N > 0)
Counter Eigenschaft | Kann erhöht werden | Kann nicht erhöht werden

## Gefahren der Nebenläufigkeit
4.1 Was sind Race Condition, Deadlocks, Starvation?
- Race Condition
	- Ungenügend synchronisierte Zugriffe auf gemeinsame Ressourcen	
	- Data Races (low-level)
		- Selbe Variable oder Array Element (mind. ein Write Zugriff von einem Thread)
	- Semantisch höhere Race Condition (high-level)
		- Low-Level Data Race mit Synchronisation elimiert
		- Critical Section nicht vollkommen unterstützt
- Deadlocks
	- Gegenseitiges Aussperren von Threads
- Starvation
	- Kontinuierliche Fortschrittsbehinderung eines Threads wegen Fairness-Probleme
	- Liveness/Fairness Problem
Parallele

4.2 Wie bzw wann kann man Synchronisation weglassen?
- Immutabiliy (final, nur lesen)
- Confinement (Einsperrung)
	- Thread Confinement
		- Objekt nur über Referenzen von einem Thread erreichbar
	- ObjectConfimement
		- Objekt in anderem bereits synchronisierten Objekt eingekapselt

4.3 Gebe ein Beispiel zu Immutabiliy.
![Immutable](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/immutable.png)

4.4 Gebe ein Beispiel zu Confinement und worauf muss man achten?
![Confinement](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/confinement.png)
![Confinement](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/confinement2.png)
Bei Confinement muss geschaut werden, dass die die Inneren Objecten nicht:
- von aussen zugreiffbar sein
- nicht an äusseren Objekten zurückgegeben werden
- sie denn Inneren "Raum" nicht verlassen

	
4.5 Welche Java Collection sind Thread save?

Version | Beispiel | Threadsicher
--------------- | --------------- | ---------------
AlteJava 1.0 Collections | Vector,Stack, Hashtable | JA
Moderne Collections(java.util, Java > 1.0) | HashSet, TreeSet, ArrayList, LinkedList, HashMap, TreeMap | NEIN
ConcurrentCollections(java.util.concurrent) | ConcurrentHashMap, ConcurrentLinkedQueue, CopyOnWriteArrayList, … | JA

4.6 Wiso sind die modernen Collection nicht mehr Thread save?
- Oft ist Synchronisation nicht nötig
	- Confinement=> Unnötige Synchronisationskosten
- Synchronisation meist ungenügend
	- Elemente sind nicht synchronisiert
	- Iteration der Elemente ist nicht synchronisiert

4.7 Wie kann man die Moderne Collection (java.util, Java > 1.0) synchronisieren. zb. eine neue synchrone ArrayList generieren?
```java
List list = Collections.synchronizedList(new ArrayList());
```
Achtung Elemente sind nicht synchronisiert!
![Synchronized List Elements](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/sle.png)

4.8 Auf was muss bei Iterieren mit Nebenläufigkeit geachtet werden?
- Iteration einer synchronisierten Collection ist nicht als Ganzes synchronisiert
	- Nur Einzelzugriffe für sich sind synchronisiert
- Anderer Thread kann die Collection parallel ändern
	- Semantisch höhere RaceCondition
	- Liefert vielleicht Exception, kann auch korrumpieren
![Nested LocksDeadlock](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/iter_sync.png)

4.9 Wie kann beim Iterator Cliet-Side Locking implementiert werden?
```java
Collection<T> c = Collections.synchronizedCollection(myCollection);
…
synchronized(c) {
	for (T element : c) {
		// use element
	}
}
Map<String, T> m = Collections.synchronizedMap(myHashMap);
…
Set<String> s = m.keySet(); // Collection view
synchronized(m) { // !!!Muss m locken, nicht s!!!
	for (String key : s) {
		// use key
	}
}
```

4.10 Welches Problem kann hier auftretten?
![Nested Locks](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/nested_locks.png)
Dead Lock: 
![Nested LocksDeadlock](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/nl_deadlock.png)

4.11 Welches Problem kann hier auftretten?
```java
class BankAccount{
	private int balance;
	
	public synchronized void transfer(BankAccount to, int amount) {
		balance -= amount;
		to.deposit(amount);
	}
	
	public synchronized void deposit(int amount) {
		balance+= amount;
	}
}
/**
*	Thread1					Thread2
*	a.transfer(b, 20);		b.trasnfert(a, 50);
*/
```

to.deposit(amount); <- Implizit geschachtelter Lock
![Nested Deadlock](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/dlock.png)

4.12 Was muss alles zutreffen das es zu einem Deadlock kommt?
- Gegenseitiger Ausschluss (Locks)
- Geschachtelte Locks
- Zyklische Warteabhängigkeiten
- Sperren ohne Timeout/Abbruch

4.13 Wie können deadlocks verhindert werden? (Philosoph Übung)
- Lineare Ordnung der Ressourcen einführen
	- Nur geschachtelt in aufsteigender Reihenfolge sperren
![Deadlock verhindern](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/dlock2.png)
- Grobgranulare Locks wählen
	- Wenn Ordnung nicht möglich/sinnvoll	
![Deadlock verhindern](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/dlock1.png)

4.14 Was sind Livelocks?
- Threads haben sich gegenseitig permanent blockiert
	- Führen aber noch Warteinstruktionen aus
	- Verbrauchen CPU während Deadlock
![Livelocks](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/livelock.png)

4.15 Wie kann Starvation vermieden wernde?
- Faire Synchronisationskonstrukte
	- Länger wartende Threads mit erfüllter Bedingung haben Vortritt
	- Fairness einschalten in Java Semaphore, Lock & Condition, Read-Write Lock
- Java Monitor hat Fairness Problem
	- Starvation-anfällig, vor allem bei vielen Threads

4.16 Was muss bei Priority Inversion geachtet wernde?
- Es ist starvation möglich, da die Höherpriorisierten Threads vorrang haben. Wenn dieser jedoch auf ein andere warten muss -> Starvation

4.17 Was ist am folgenden Beispiel falsch, wie kann das Problem gelöst werden?
```java
void acquireMultiLocks(Lock[] lockSet) {
	int i= 0;
	while (i < lockSet.length) {
		if (lockSet[i].tryLock()) { //tryLock: true, falls Sperre erhalten - false, falls von anderem gesperrt
			i++;
		} else {
			while (i> 0) {
				i--;
				lockSet[i].unlock();
			}
		}
	}
}
```

## Thread Pools & Asynchrone Programmierung
5.1 Erkläre das Thread Pool Prinzip.
- Thread Pool ist ein Topf mit einer beschränkter Anzhal Threads.
- Task implementieren potentiele Arbeitspakete, welche als Queue warten bis ein Thread im Pool diese Abarbeitet.

5.2 Was sind die Vorteile von Thread Pool?
- Beschränkte Anzahl von Threads
- Recycling der Threads
- Höhere Abstraktion
- Anzahl Threads individuell pro System konfigurierbar

5.3 Was muss bei den Tasks beachtet werden?
- Task sollten nicht voneinander abhängig sein, denn wenn alle Worker Threads mit wartende Task beschäftigt sind entsteht ein Dead lock.

5.4 Was für zwei Szenarien gibt es bei Thread Pool, die zu einem Deadlock führen können?

1. Alle Worker-Threads sind mit Tasks beschäftigt, die auf eine Bedinung warten.
2. Die Bedingungen können nur von Tasks erfüllt werden, die in der Warteschlange stecken

5.5 Wiso kann der Worker Thread nicht andere Tasks ausführen, wenn ein Task blockiert?
- Weil sonst ein neuer Task aufgerufen werden müsste.

5.6 Beispiel für Task Implementierung & Ausführung
```java
/**
* ComplexCalculation1 = Thread-Pool Auftrag
* Callable<Integer> = Integer ist der Rückgabewert des Tasks
*/
class ComplexCalculation1 implements Callable<Integer>{
	@Override
	public Integer call() throws Exception {
		intvalue = …;
		// long calculation	
		return value;
	}
}

//mit java 8
//Future<T> submit(Callable<T> task)
Future<Integer> future = threadPool.submit(() -> {
	int value = …;
	// long calculation
	return value;
});
```
```java
//Erzeugt Thread-Pool mit zwei Worker Threads
ExecutorService threadPool = Executors.newFixedThreadPool(2);
…
//Handle auf Task Resultate
Future<Integer> future1, future2;

//.submit([Task]) schickt Tasks an den Thread-Pool
future1 = threadPool.submit(new ComplexCalculation1());
future2 = threadPool.submit(new ComplexCalculation2());
…
//Wartet auf Task Ende und holt die Resultate
intresult1 = future1.get();
intresult2 = future2.get();
…
//Stellt Thread-Pool ab.
threadPool.shutdown();
```

5.7 Wie kann ein Future abgebrochen werden?
```java
future.cancel(boolean mayInterruptIfRunning);
```

5.8 Welche Thread-Pool Factory gibt es?
- Executors.newFixedThreadPool(intnofThreads)
	- Neuer Thread-Pool mit fixer Anzahl Worker Threads
	- Meist Runtime.getRuntime().availableProcessors()für nofThreads
- Executors.newCachedThreadPool()
	- Kreiert automatisch so viele Threads wie benötigt
	- Geeignet für Tasks mit blockierenden I/O Calls
- Executors.newSingleThreadExecutor()
	- Nur ein Worker Thread (nofThreads== 1)
	
5.9 Wieso blockiert Programmbeendigung ohne ExecutorService.shutdown()?	
Weil sie als Non-Deamons implementiert sind und daher lassen sie das System am Leben.

5.10 Erkläre das Fire and Forget Prinzip. Auf was muss dabei geachtet werde?
- Tasks starten, ohne dabei das Resultat später abzuholen
	- Callable<Void>
```java
ExecutorService threadPool = Executors.newFixedThreadPool();
...
threadPool.submit(() -> { //implementiert Callable<Void>
	//Task implementation
});
...
threadPool.shutdown();
```
Bei Fire and Forgot muss geachtet werden, dass wenn im Task eine Exception geworfen wird, diese ignoriert wird.

5.11 Folgendes Beispiel  implementiert eine Suche als Task. Schreibe diese mit java8 um
```java
class SearchTask implements Callable<Boolean>{
	private List<String> words;
	private String pattern;

	public SearchTask(List<String> words, String pattern) {
		this.words= words; this.pattern= pattern;
	}

	@Override
	public Boolean call() throws Exception {
		for (String text : words) {
			if (text.matches(pattern)) { return true; }
		}
		return false;
	}
}

ExecutorService threadPool= Executors.newFixedThreadPool(2);
```
![Paralleles Suchen](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/lr.png)

```java 
Future<Boolean> left = threadPool.submit(new SearchTask(leftPart, pattern));
Future<Boolean> right = threadPool.submit(new SearchTask(rightPart, pattern));
boolean found = left.get() || right.get();
threadPool.shutdown();
```
Java8:
![Paralleles Suchen Java8](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/java812.png)

5.12 Was sind Recursive Tasks? Beispiel dazu? Api?
- Thread Pool für rekursive Tasks (ab Java 7)
	- Tasks können Untertasks starten und abwarten
	- Warten könnte bei ExecutorServiceDeadlock geben
- Tasks erben von RecursiveTask<T>
	- T compute(): Task Implementierung
	- fork(): Starte als Sub-Task in einem anderen Task
	- T join(): Warte auf Task-Ende und frage Resultat ab
- RecursiveAction, falls kein Rückgabetyp
	- void statt T
- T invoke(ForkJoinTask<T>task)
	- Führe Task aus und wartet auf Beendigung des Tasks
	- T ist Resultat des Tasks (Void, wenn kein Resultat)
- submit(ForkJoinTask<T>task)
	- Task einreihen, aber nicht abwarten
- ForkJoinTaskist Basisklasse von RecursiveTask und RecursiveAction

![Rekursive Tasks](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/recursive_task1.png)
```java
class MySuperTask extends RecursiveTask<Integer>{
	@Override
	protected Integer compute(){
		MySuberTask sub = new MySuberTask(…);
		sub.fork(); //Startet Sub-Task
		// other work
		sub.join(); //Wartet auf Beendigung
		return…;
	}
}

//Spezielle Thread-Pool für rekursive Tasks (Deamon Thread)
ForkJoinPool threadPool= new ForkJoinPool();
…
//Führe Haupt-Task asu (wartet auf Beendigung)
threadPool.invoke(newMySuperTask());
```

5.13 Warum ist kein Shutdown bei Fork Join Pool nötig?
Kein Shutdown des Thread-Pool nötig, da diese als Deamon Thread laufen.

5.14 Implementiere das Recursive Finden mit dem Fork Join Pool Prinzip.
```java
class SearchTask extends RecursiveTask<Boolean>{
	private List<String> words; 
	private String pattern;
	
	public SearchTask(List<String> words, String pattern) {
		this.words= words; this.pattern= pattern;
	}
	
	protected Boolean compute() {
		int n = words.size();
		if (n == 0) { return false; }
		if (n == 1) { return words.get(0).matches(pattern); }
		SearchTaskleft = new SearchTask(words.subList(0, n/2), pattern);
		SearchTaskright = new SearchTask(words.subList(n/2, n), pattern);
		left.fork();
		right.fork();
		return right.join() || left.join();
	}
}
```

5.15 Was versteht man unter Work Stealing Thread Pool?
![Thread Stealing](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/thread_stealing.png)
Verteilte Queues für weniger Contention (Wettbewerb)
- Weniger Threads streiten um gleiche Locks
	- Viel effizienter
- FIFO
	- Globale Queue
	- Verteilen zwischen Queues
- LIFO
	- Neue Sub-Tasks kommen zuvorderstin lokaleTask Queue
	- Grund für Schachtelung der fork/join

5.16 Was für ein Fairness-Problem ergibt sicht beim Work Stealing Thread Pool Prinzip?
*Starvation-Gefahr*
Wegen Priorisierung von Neuen Tasks

5.17 Erkläre das Asychrone Aufruf Prinzip.

![Asynchrone](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/async.png)
```java
long result= long Operation(); // Asynchroner Aufruf gewünscht, Aufrufer unnötig blockiert
otherWork();
process(result);
```
Gut für I/O Calls oder aufwendige Berechnungen

5.17 Wie wird der Asynchrone aufruf mit Java8 implementiert?
```java
//Asynchroner Aufruf mittels Thread-Pool		1 Worker-Thread reicht hier für asynchrone Abarbeitung
ExecutorService threadPool					= 	Executors.newSingleThreadExecutor();
Future<Long> future = threadPool.submit(() ->longOperation()); //longOperation, Thread-Pool Task als Lambda
otherWork();
process(future.get // Resultat über Future
threadPool.shutdown();
```

5.18 Wie wird Callback Asynchrone implementiert?
```java
interface CallbackHandler<T> {
	void handleResult(T result);
}

class MyMath{
	ExecutorService threadPool= …;
	
	void asyncLongOperation(long input, CallbackHandler<Long> callback) { //CallbackHandler<Long> as Parameter mitgeben.
		threadPool.submit(() -> {
			long result = longOperation(input);
			//Callback am Schluss der asynchronen Arbeit
			callback.handleResult(result);
		});
	}
}
```

5.19 Von welchem Thread wird der Thread / der callback aufgerufen?
Callback wird von einem anderem Thread als der vom Aufrufen ausgeführt.
Synchronisation ist eventuell nötig zwischen Caller und Worker Thread.
![Asynchrone Synchronisation](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/async_sync.png)

5.20 Wie wird diser Callback mit Synchronisation implementiert?
```java
class Calculator {
	private boolean isRunning= false;
	
	public synchronized void test() {
		if (!isRunning) { //Mehrfachausführung z.B. ignorieren
			isRunning = true;
			MyMath.asyncLongOperation(N, (result) -> {
				show(result);
				synchronized(this) {
					//Kann frühestens eintreten, wenn test() fertig ist
					isRunning= false;
				}
			}
		}
	}
}
```

5.21 Wann braucht es Synchronisation beim Callback?

![Callback Synchronisation](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/callback_sync.png)
- Wenn das Resultat in Zukunft gebraucht wird. 
- Wenn der Rote Thread einen Callback macht und dieser mit dem Laufendem Thread synchronisiert wird.

## .NET

6.1 Was sind die Unterschiede zwischen Java8 und .net c#
- | Java | C#
--------------- | --------------- | ---------------
neuer Thread | new Thread(() -> { // threadbehavior }); | new Thread(() => { //threadbehavior });
Zugriff auf variablen in Runnable(java) / lambdas(c#) | nur read (effectively final) | write / read -> Prädesteniert für Data Races
Synchronize Block | synchronize( Object ){} | lock( syncObject) {}
Exception | nur der Thread bricht ab | Führen bei default zum abbruch des gesamten Programms
Monitor | |
while um wait | nötig | nötig
monitor.wait() | wait(); | Monitor.Wait(syncObject);
notify | notifyAll(); | Monitor.PulseAll(syncObject);
Warteschlange | keine, sporious wake up | FIFO
Überholproblem? | Ja | Ja
synchronisation mit | mit this: synchronize( this ){} | private Object syncObject = new Object(); <br />lock( syncObject) {}
 | | 
Collection theadsave? | siehe 4.5 | Nein, ausser
 
6.2 Gebe ein Beispiel zu .Net Monitor
![.Net Monitor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/.netmonitor.png)

6.3 In Java wurde immer this (best practice) gelockt, in .Net (best praktce) wird ein Hilfsobject verwendet (syncObject) um die Synchronisation zu garantieren. Was sind dabei die Vorteile?
- Vorteile: Mehrere verschiedene Locks innerhalb einer Klasse. Es wird verhindert, dass jemand BankAccount von aussen locken kann.
- Nachteile: Es kann nicht von aussen gelockt werden.

6.4 Was ist hier falsch?
```c#
for(int i = 0; i < 100; i++) {
	Task.Factory.StartNew(() => {
		// use i as input
		Console.WriteLine("Working with" + i);
	});
}
```
Falsch ist, das auf i gelesen und geschrieben wird

6.5 Was ist die Ausgabe des folgenden Code?
```c#
staticvoidMain() {
	Task.Factory.StartNew(() => {
		// somecalculation
		Console.WriteLine("Task finished");
	});
}
```
Kommt darauf an. Task wird evtl. nicht oder unvollständig ausgeführt, da der Task ein Deamon Thread ist.

6.6 Wie lässt sich folgender Code korrigieren?
![.Net Race](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/.netrace.png)
- Lösung
```c#
long totalWords= 0;
Parallel.ForEach<FileInfo>(files, (f) => {
	int subTotal= CountWords(f);
	lock(someLockObj) {
		totalWords+= subTotal;
	}
);
```

6.7 Task Parallelisierung: 
####Start und Wait
```c#
Task task = Task.Factory.StartNew(() => {
	// task implementation
});
// perform other activity
task.Wait(); //Blockiert, bis Task beendet ist
```

####Multi Task Start und Wait
```c#
Task[] taskArray= newTask[] {
	Task.Factory.StartNew(() => Calculation1()),
	Task.Factory.StartNew(() => Calculation2()),
	Task.Factory.StartNew(() => Calculation3())
};
Task.WaitAll(taskArray);//Ende von allen Tasks abwarten
//---------------------------//
Task<double>[] taskArray= newTask<double>[] {
	Task.Factory.StartNew(() => FindInLeft()),
	Task.Factory.StartNew(() => FindInRight())
};

//Index des beendeten Tasks
int index = Task.WaitAny(taskArray); //Ersten beendeten Task abwarten
Console.Write(taskArray[index].Result);
```
Task können Sub-Tasks starten und/oder abwarten
```c#
Task.Factory.StartNew(() => {
	// outer task
	Task<bool> left = Task.Factory.StartNew(() => Search(leftPart));
	Task<bool> right = Task.Factory.StartNew(() => Search(rightPart));
	bool found = left.Result|| right.Result;
	…
});
```

####Rückgabetyp
```c#
//int ist Rückgabetyp
Task<int> task = Task.Factory.StartNew(() => {
	int total = … // somecalculation
	return total;
});
…
Console.Write(task.Result); // Blockiert bis Task Ende und liefert dann Resultat
```

####Task Parameter Übergabe
Entweder direkter Zugriff (anfällig auf Race Conditions) oder
```c#
//Object obj = new Object();
Task task = Task.Factory.StartNew((obj)=> {
	int myParamValue = (int)obj; //cast nötig
	// task implementation using myParamValue
}, 10); // 10 ist das Argument, welches übergeben wird.
```

6.8 Wie lässt sich hierfür ein CompletionCalback/Handling realisieren? Was ist dabei zu beachten?
```c#
Task task = Task.Factory.StartNew(() => DownloadFile(url));
```
```c#
task.Result; // (is a callback. Result blockiert..!!!)
//Oder
task.ContinueWith(otherTask).ContinueWith(otherTask);// führe etwas nach 
```
Wenn url von aussen geändert wird kann eine RaceCondition entstehen.

## .Net Gui und Threading
7.1 Aufgrund welcher Motivation basieren GUI-Frameworks auf einem Single-Thread Modell?
- Deadlock-Risiko
- Synchronisationskosten

7.2 Welche Einschränkungen ergeben sich aus dem GUI-Thread Modell?
- Geschwindigkeitsverlust.
- Programmierer müssen sich dem Bewusst sein.

7.3 Was wird von welchem Thread ausgeführt und wieso?
```java
class MyButtonActionListener implements ActionListener{
	@Override
	public void actionPerformed(ActionEvent arg) {
		new Thread(() -> {
			String text = readHugeFile();
			SwingUtilities.invokeLater(() -> {
				textArea.setText(text);
			});
		}).start();
	}
}
```
![UI dispatching Analyse](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/ui1.png)

7.4 Was eignet sich anstatt des Thread.start()?
Task

7.5 An welchen Stellen sind RaceConditionsmöglich?
```java
public static void main(String[] args) {
	final JFramef = new JFrame("Labels");
	// ...
	f.pack();
	f.setVisible(true);
	// ...
}
```
![UI dispatching Analyse](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/ui2.png)

7.6 Wie kann das Problem von 7.5 gelöst werden?
```java
public static void main(String args[]) {
	final JFrameframe = new JFrame("MyFrame");
	JButton button = new JButton();
	button.setText("Click");
	frame.getContentPane().add(button, ..);
	frame.setSize(200, 100);
	// ...
	// !! pack()& setVisible()im Event DispatchThread ausführen !!
	SwingUtilities.invokeLater(() -> {
		frame.pack();
		frame.setVisible(true);
	});
}
```
pack()& setVisible()im Event DispatchThread ausführen

7.7 Was ist am folgendem Code falsch? Wie kann dies Korrigiert wernde?
```c#
public async Task<bool> IsPrimeAsync(long number) {
	for(long i = 2; i <= Math.Sqrt(number); i++) {
		if(number % i == 0) { return false; }
	}
	return true;
}
```
Diese Methode läuft synchron da kein await gibt, was die methode asynchrom machen würde
```c#
public async Task<bool> IsPrimeAsync(long number) {
	return await Task.Run(() => {
		for (long i= 2; i<= Math.Sqrt(number); i++) {
			if (number % i== 0) { return false; }
		}
		return true;
	});
}
```

7.8 Was ist die Ausgabe des folgenden Code, wenn kein UI / Synchronisationskontext mit Dispatcher vorhanden ist?
```c#
public async Task DownloadAsync() {
	Console.WriteLine("BEFORE "+ Thread.CurrentThread.ManagedThreadId);
	HttpClient client= new HttpClient();
	Task<string> task = client.GetStringAsync("...");
	string result= await task;
	Console.WriteLine("AFTER " + Thread.CurrentThread.ManagedThreadId);
}
```
!!Partielle Nebenläufigkeit berücksichtigen!!
> BEFORE 10
> ...
> AFTER 14

7.9 Beurteile das Async-Await Model.
- Eine Async-Methode => Zwei Aufruffälle
	- Mit und ohne Synchronisationskontext
- Viraler Effekt: async/await-Aufrufer wird auch async
	- Unnötig komplexe Methodensignaturen mit Tasks
	- Kosten wegen interner Methodenzerstückelung
- Synchrone/asynchrone Verwendung sollte meist orthogonal sein
	- "Caller" sollte entscheiden, nicht "Callee"

7.10 Wie kann ein solches Problem(InvalidOperationException Collection was modified) auftreten, zumal es keine echte Parallelität im GUI-Code gibt?
```c#
async void startDownload_Click(…) {
	HttpClient client = newHttpClient();
	foreach(var url in collection) {
		var data= await client.GetStringAsync(url);
		textArea.Content+= data; //InvalidOperationException Collection was modified
	}
}
```

##Memory Models
8.1 Was sind die möglichen Werte für i und j?

![Memory Models](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/m1.png)

j | i
--------- | ---------
1 | 0
0 | 1
1 | 1
0 | 0

denn der Code kann ungeordnert werden (performance optimierung.)

![Memory Models](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/m2.png)

8.2 Beschreibe das Problem bei Nebenläufigkeit bei Memory Models.
- Weak Consistency
- Optimierung

8.3 Erkläre die Begriffe Atomicity, Visibility, Ordering im Zusammenhang mit Java Memory Model 
- Atomicity(Unteilbarkeit) 
	- von Lese-und Schreibzugriffe auf Speicher
- Visibility(Sichtbarkeit) 
	- aktueller Speicherwerte zwischen Threads
- Ordering(Reihenfolge)
	- von Lese-/Schreibeoperationen auf Speicher
	
8.4 Welche Werte in Java Garantieren Atomicity?
- Zugriff auf Variable (Lesen/Schreiben) ist atomar für
	- Primitive Datentypen bis 32 Bit
		- Nicht longund double
	- Objekt-Referenzen
		- Egal ob intern 32 Bit oder 64 Bit
	- Mit volatile Keyword
		- longund double dann auch atomar
		
8.5 Welche Anweisungen sind atomar?

1. longl = -1;
2. double d = 3.14;
3. inti = 1;
4. String s = "first";
5. booleanb = true;
...
6. i = (int)l;
7. l = 42;
8. i = 7;
9. ++i;
10. s = "second";
11. d = d * i;
12. s = null;
13. b = false;

![Atomicity](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/m3.png)

8.6 Was ist das Problem hier?
```java
class Worker extends Thread {
	private booleandoRun= true;

	public void run() {
		while(doRun) {
			…
		}
	}
	
	public void stopRequest() {
		doRun= false;
	}
}
```
- Sehe Änderungen eines anderen Thread eventuell nicht oder viel später
	- Wegen Register (oder im Java Memory Model Cache)
- Compiler-Optimierung
	- Hält Variablenwert eventuell in Register
```
//!!Endlosschlaufe!!
void run() {
	load doRun to reg1;
	while(reg1) {
		…
	}
}
```

8.7 Wenn ist die Visibility (Sichtbarkeit) Garantiert?
- Locks Release & Acquire
	- Schreibender Thread ruft Release und lesender Thread ruft Acquirefür denselben Lock auf
- Volatile Variablen
	- Lese/Schreibe-Zugriff darauf
- Initialisierung von final Variablen
	- Nach Ende des Konstruktors
- Thread-Start und Join
	- Ebenso Task Start und Ende
	
8.8 Wo ist die Sichtbarkeit von x==1 garantiert?

![Visibility](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/m4.png)

1. Garantiert
2. Nicht Garantiert
3. Garantiert (Ausser wenn x während des Constructors gelesen wird.)
4. Garantiert

8.9 Wie kann man das hier korrigieren? Finden Sie zwei Lösungsmöglichkeiten.
```java
class Worker extends Thread {
	private boolean doRun = true;
	
	public void run() {
		while(shouldRun()) {
			…
		}
	}
	
	private boolean shouldRun() {
		return doRun;
	}
	public void stopRequest() {
		doRun = false;
	}
}
```

Lösung 1 mit Synchronized
```java
class Worker extends Thread {
	private boolean doRun = true;
	
	public void run() {
		while(shouldRun()) {
			…
		}
	}
	
	private synchronized boolean shouldRun() {
		return doRun; //Visibility wegen Lock Acquire
	}
	public synchronized void stopRequest() {
		doRun = false; //Visibility wegen Lock Release
	}
}
```

Lösung 2 mit volatile
```java
class Worker extends Thread {
	private volatile boolean doRun = true;//Visibility mittels volatile Variable
	
	public void run() {
		while(shouldRun()) {
			…
		}
	}
	
	private boolean shouldRun() {
		return doRun;
	}
	public void stopRequest() {
		doRun = false;
	}
}
//!!Volatile verhindert Data Raceauf Variable!!
```

8.8 Was änderet das Keyword volatile alles bezüglich, Atomicity, Visibility und Reordering?
- Atomicity
	- Atomares Lesen & Schreiben auch für longund double
	- Achtung: Andere Operationen sind nicht atomar (i++ etc.)
- Visibility
	- Lese und Schreibzugriffe via Hauptspeicher propagiert
	- Achtung: Kein Sperren im Gegensatz zu Locks
- Reordering
	- Keine Umordnungdurch Compiler / Laufzeitsystem / CPU
	- Achtung: Nicht-volatile Variablen werden evtl. umgeordnet
	
8.9 Wenn kann Ordering angewendet werden?
- Innerhalb eines Threads: «As-If-Serial» Semantik
	- Umsortieren von Instruktionen erlaubt, falls sequentielles Verhalten aus Sicht eines Threads gleich bleibt
- Zwischen Threads: Reihenfolge nur erhalten für
	- Synchronisationsbefehle
		- synchronized, Lock Acquire/ Release
	- Zugriffe auf volatile Variablen
- Nicht-volatile Zugriffe werden nicht über Grenzen von Synchronisation oder volatile Zugriffe optimiert
	- Memory Barriers/ Memory Fences

8.10 Welche Umordnungensind möglich?

![Ordering](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/m5.png)

1. Ja
2. Nein
3. Nein
4. Nein

8.11 Was ist der Unterschied zwischen volatile und Synchonized?

- | Volatile | Synchronized
--------------- | --------------- | ---------------
Atomarität | Einzelnes Lesen/Schreiben | Gesamter Statement Block
Blockierung(Lock & Wait) | Nein | Ja
Garantiert Sichtbarkeit | Ja | Ja
Verhindert Reordering | Ja | Ja

8.12 Welches Problem sehen Sie hier?
```java
do {
	oldValue = var.get(); //Lese aktuellen Wert
	newValue = calculateChanges(oldValue);
} while (!var.compareAndSet(oldValue, newValue)); //compareAndSet: Schreibe, falls gelesener Wert immer noch aktuell ist
```
Anderer Thread schreibt unbemerkt dazwischen
![ABA Problem](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/m6.png)

8.13 Wieso ist Double CheckedLocking ohne volatile falsch?
```java
class LazyCreation{
	private volatile LargeObject instance;
	
	public LargeObject get() {
		if (instance== null) {
			synchronized (this) {
				if (instance == null) {
					instance = new LargeObject();
				}
			}
		}
		return instance;
	}
}
```

Thread 1 | Instance | Thread 2
--------------- | --------------- | ---------------
If (instance == null) | Null | If (instance == null)
Sync -> waiting | Instance = some value | Sync -> ok
 |  | Create instance =
 |  | Return instance
Enter sync block |  | 

Weil der Konstruktor vor dem instance = x aufgerufen werden kann (vom Compiler) 
-> Data Race Condition 
-> wenn T1 -> Instance = x

Theoretisch möglich, dass das Object noch nicht richtig instanziert wurde, jedoch wurde zwischenzeitlich die Instance = x scho geschrieben 
Dies würde für T2 instance == null false ergeben.

##GPU Parallelisierung

9.1 Erkläre die Begriffe Streaming Multiprocessor und Streaming Processor
SM (Streaming Multiprocessors ) sind ein Sammeltopf von SP (Streaming Processor), die SM sind so etwas wie der CPU.
Die SM müssen alle die gleiche Instruktion auf verschiedenen Daten ausführt. (SIMD Single Instruction Multiple Data)

9.2 Welche Programme können damit effizient parallelisiert werden? Bzw. welche nicht?
Welche die gleiche Operation auf verschiedenen Daten ausführen müssen. Games

9.3 GPU vs. CPU

GPU | CPU
--------------- | ---------------
GPU: Video Gaming | CPU: General Purpose
Sehr hohe FLOPs | Niedrige Datenparallelität, viel Verzweigungen
Extrem hohe Datenparallelität | Concurrency mit Synchronisation
Hohe Memory Bandbreite | Wenige, aber mächtige Cores
Einfache, aber viele Cores | Grössere Caches in Chip
Kleine Caches pro Core | 
*Ziel: Hoher Gesamtdurchsatz* | *Ziel: Niedrige Latenz pro Thread*

9.4 Bei einer Vektoradition, welches ist die Optimalste Lösung? Von welchen Kriterien hängt das ab?
![Adition Vektor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/gpu1.png)
![Adition Vektor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/gpu2.png)
Kriterien sind:
- Maximale Anzahl Threads per Block
	- Abhängig von GPU, z.B. 512 oder 1024
- Blockgrösse als Vielfaches von 32
	- Sonst sehr ineffizient
- Überflüssige Threads vermeiden
	- 2 Blöcke à 1024 => 548 unnütze Threads
- Streaming Multiprocessorausschöpfen
	- Limitefür Resident Blöcke und Threads, z.B. 8 und 1536
- Grosse Blockgrösse hat Vorteile
	- Threads können nur in Block interagieren

9.5 Welches ist die optimale Konfiguration?
Gegeben sei:
Vektor Länge  1500
Maximale Anzahl Threads per Block = 512
Maximale Anzahl Resident Blocks = 8
Maximale Anzahl Resident Threads = 1536
![Adition Vektor](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/g3.png)

# Übungen
####2.2.b BankAccount withDraw mit Timeout
Unterstützten Sie für das Abheben einen Timeout. Die Methode withdraw() soll maximal die spezifizierte Zeit (Millisekunden) warten, bis dass das Konto mindestens den Kontostand hat, um den Betrag ohne Negativsaldo abheben zu können. Wenn das Abheben gelingt, wird true zurückgegeben; sonst bei Timeout false.
```java
// returns true if successful, false if failed after timeout.
public synchronized boolean withdraw(int amount, long timeOutMillis)
```
```java
// returns true if successful, false if failed after timeout.
public synchronized boolean withdraw(int amount, long timeOutMillis) {
	if (timeOutMillis < 0) {
		throw new IllegalArgumentException("timeOutMillis is negative");
	}
	long startTime = System.currentTimeMillis();
	long currentTime = startTime;
	while (amount > balance && currentTime - startTime < timeOutMillis) {
		try {
			// wait timeout period must be positive: therefore fix
			// currentTime in temporary variable
			wait(timeOutMillis - currentTime + startTime);
		} catch (InterruptedException e) {
		}
		currentTime = System.currentTimeMillis();
	}
	if (amount <= balance) {
		balance -= amount;
		return true;
	} else {
		return false;
	}
}
```


####2.3 Producer - Consumer Bounded Buffer
Beim Producer-Consumer Szenario gibt es eine Anzahl Producer-Threads und eine Anzahl Consumer-Threads, die mit einem gemeinsamen Buffer interagieren. Die Producer-Threads fügen wiederholt Werte in den Buffer ein, während die Consumer-Threads wiederholt Werte aus dem Buffer herausnehmen.
Der Buffer hat eine beschränkte Kapazität, d.h. das Einfügen put() blockiert so lange, wie der Buffer voll ist. Umgekehrt blockiert das Entnehmen get() so lange, wie der Buffer leer ist.
![Aufgabe1](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/u2.png)
##### Bounded Buffer als Monitor
```java
class BoundedBuffer<T> {
	private final Queue<T> queue = new LinkedList<T>();
	private final int capacity;

	public BoundedBuffer(int capacity) {
		this.capacity = capacity;
	}

	public synchronized void put(T item) throws InterruptedException {
		while (queue.size() == capacity) {
			wait();
		}
		queue.add(item);
		notifyAll();
	}

	public synchronized T get() throws InterruptedException {
		while (queue.size() == 0) {
			wait();
		}
		T item = queue.remove();
		notifyAll();
		return item;
	}
}
```
####4.3 Upgradable ReadWriteLook
![Aufgabe1](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/u4.png)

Wieso soll der Upgrade-Wunsch bereits beim Read Lock speziell mit upgradeableReadLock() bekannt gegeben werden?
	- Angenommen man würde den Upgrade während den Reads ohne weiteres erlauben, könnte es Deadlocks geben. Dies dann, wenn beispielsweise zwei Threads den Read Lock gleichzeitig besitzen und beide dann zu Write erhöhen wollen: Jeder hindert den anderen am Upgrade, weil er selbst schon ein Read Lock hat und den nicht mehr ohne Upgrade freigeben will. Deadlock Szenario (falls gleichzeitige Upgradeable Reads erlaubt wären)
Thread 1 | Thread 2
------------ | ------------
Read Lock |                        
- | Read Lock
Upgrade zu Write gewünscht | 
=> blockiert | 
 | Upgrade zu Write gewünscht
 | => blockiert

```java
package u04.a03;
//Speziell an der Lösung ist, dass ein Read Lock nur dann zu einem Write Lock erhöht werden kann, wenn dies bereits beim Read Lock bekannt gegeben wird (upgradeableReadLock() statt nur readLock()).
public class UpgradeableReadWriteLock {
	int readCount;
	boolean writeLock;
	Thread upgradeableThread;
	boolean upgraded;

	public synchronized void readLock() throws InterruptedException {
		while (writeLock)
			wait();
		readCount++;
	}

	public synchronized void readUnlock() {
		readCount--;
		notifyAll();
	}

	public synchronized void upgradeableReadLock() throws InterruptedException {
		while (writeLock || upgradeableThread != null)
			wait();
		readCount++;
		upgradeableThread = Thread.currentThread();
	}

	public synchronized void upgradeableReadUnlock() {
		if(upgraded) {
			writeLock = false;
			upgraded = false;
		}
		upgradeableThread = null;
		readCount--;	
		notifyAll();
	}

	public synchronized void writeLock() throws InterruptedException {
		while (writeLock || readCount > 1 || (readCount == 1 && upgradeableThread != Thread.currentThread()))
			wait();
		writeLock = true;
		if(upgradeableThread == Thread.currentThread()) {
			upgraded = true;
		}
	}

	public synchronized void writeUnlock() {
		if(upgraded)
			upgraded = false;
		writeLock = false;
		notifyAll();
	}
}
```

####5.1 Quicksort
##### Sequenziell
```java
// sorts the entire integer array in ascending order
void sort(int[] array) {
	quickSort(array, 0, array.length - 1);
}

// sorts the partition between array[left] and array[right]
void quickSort(int[] array, int left, int right) {
	int i = left, j = right;
	long m = array[(left + right) / 2];
	do {
		while (array[i] < m) {
			i++;
		}
		while (array[j] > m) {
			j--;
		}
		if (i <= j) {
			int t = array[i];
			array[i] = array[j];
			array[j] = t;
			i++;
			j--;
		}
	} while (i <= j);
	if (j > left) {
		quickSort(array, left, j);
	}
	if (i < right) {
		quickSort(array, i, right);
	}
}
```
##### Parallel
```java
package aufgabe1;

import java.util.Random;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;

class QuickSortTask extends RecursiveAction {
	private static final long serialVersionUID = 1L;
	private static final int THRESHOLD = 100000;

	private int[] array;
	private int left, right;

	public QuickSortTask(int[] array, int left, int right) {
		this.array = array;
		this.left = left;
		this.right = right;
	}

	// sorts the partition between array[l] and array[r]
	private void quickSort(int l, int r) {
		int i = l, j = r;
		long m = array[(l + r) / 2];
		do {
			while (array[i] < m) {
				i++;
			}
			while (array[j] > m) {
				j--;
			}
			if (i <= j) {
				int t = array[i];
				array[i] = array[j];
				array[j] = t;
				i++;
				j--;
			}
		} while (i <= j);
		if (j - l > THRESHOLD && r - i > THRESHOLD) {
			invokeAll(
					new QuickSortTask(array, l, j), 
					new QuickSortTask(array, i, r)
					);
		} else {
			if (j > l) {
				quickSort(l, j);
			}
			if (i < r) {
				quickSort(i, r);
			}
		}
	}

	protected void compute() {
		quickSort(left, right);
	}
}

public class QuickSortSample {
	private static final int NOF_ELEMENTS = 10_000_000;

	private static int[] createRandomArray(int length) {
		Random random = new Random(4711);
		int[] numberArray = new int[length];
		for (int i = 0; i < length; i++) {
			numberArray[i] = random.nextInt();
		}
		return numberArray;
	}

	private static void checkSorted(int[] numberArray) {
		for (int i = 0; i < numberArray.length - 1; i++) {
			if (numberArray[i] > numberArray[i + 1]) {
				throw new RuntimeException("Not sorted");
			}
		}
	}

	public static void main(String[] args) {
		int[] numberArray = createRandomArray(NOF_ELEMENTS);
		long startTime = System.currentTimeMillis();
		ForkJoinPool pool = new ForkJoinPool();
		pool.invoke(new QuickSortTask(numberArray, 0, NOF_ELEMENTS - 1));
		long stopTime = System.currentTimeMillis();
		System.out.println("Total time: " + (stopTime - startTime) + " ms");
		checkSorted(numberArray);
	}
}
```

####5.2 Download mit Future
```java
package aufgabe2a;

import java.io.IOException;
import java.io.InputStreamReader;
import java.io.Reader;
import java.net.URL;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class WebDownload {
	ExecutorService threadPool = Executors.newCachedThreadPool();

	public Future<String> asyncDownloadUrl(final String link) {
		return threadPool.submit(new Callable<String>() {
			@Override
			public String call() {
				try {
					URL url = new URL(link);
					StringBuffer stringBuffer = new StringBuffer();
					try (Reader reader = new InputStreamReader(url.openStream())) {
						int i;
						while ((i = reader.read()) >= 0) {
							stringBuffer.append((char) i);
						}
					}
					return stringBuffer.toString();
				} catch (IOException e) {
					throw new RuntimeException(e);
				}
			}
		});
	}

	public void shutdown() throws InterruptedException {
		threadPool.shutdown();
	}
}
```

```java
package aufgabe2a;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class TestDownload {
	private static final String[] links = new String[] {
			"http://www.google.com", 
			"http://www.bing.com",
			"http://www.yahoo.com", 
			"http://www.microsoft.com",
			"http://www.oracle.com" 
		};

	public static void main(String[] args) throws IOException,
			InterruptedException, ExecutionException {
		long startTime = System.currentTimeMillis();
		WebDownload downloader = new WebDownload();
		List<Future<String>> futures = new ArrayList<>();
		for (int i = 0; i < links.length; i++) {
			futures.add(downloader.asyncDownloadUrl(links[i]));
		}
		for (int i = 0; i < links.length; i++) {
			String result = futures.get(i).get();
			System.out.println(String.format("%s downloaded (%d characters)", links[i], result.length()));
		}
		downloader.shutdown();
		long endTime = System.currentTimeMillis();
		System.out.println(String.format("total time: %d ms", endTime - startTime));
	}
}
```

####5.2 Download mit Callback
```java
package aufgabe2b;

public interface CompletionHandler<T> {
	void completed(T result);
}
```

```java
package aufgabe2b;

import java.io.IOException;
import java.io.InputStreamReader;
import java.io.Reader;
import java.net.URL;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class WebDownload {
	ExecutorService threadPool = Executors.newCachedThreadPool();

	public void asyncDownloadUrl(final String link,
			final CompletionHandler<String> handler) {
		threadPool.submit(new Runnable() {
			@Override
			public void run() {
				try {
					URL url = new URL(link);
					StringBuffer stringBuffer = new StringBuffer();
					try (Reader reader = new InputStreamReader(url.openStream())) {
						int i;
						while ((i = reader.read()) >= 0) {
							stringBuffer.append((char) i);
						}
					}
					handler.completed(stringBuffer.toString());
				} catch (IOException e) {
					throw new RuntimeException(e);
				}
			}
		});
	}

	public void shutdown() throws InterruptedException {
		threadPool.shutdown();
	}
}
```

```java
package aufgabe2b;

import java.io.IOException;
import java.util.concurrent.ExecutionException;

public class TestDownload {
	private static final String[] links = new String[] {
			"http://www.google.com", 
			"http://www.bing.com",
			"http://www.yahoo.com", 
			"http://www.microsoft.com",
			"http://www.oracle.com" 
		};

	public static void main(String[] args) throws IOException,
			InterruptedException, ExecutionException {
		final long startTime = System.currentTimeMillis();
		WebDownload downloader = new WebDownload();
		for (int i = 0; i < links.length; i++) {
			final String link = links[i];
			downloader.asyncDownloadUrl(link, new CompletionHandler<String>() {
				@Override
				public void completed(String result) {
					long endTime = System.currentTimeMillis();
					System.out.println(String.format("%s downloaded (%d characters) after %d ms", 
							link, result.length(), endTime - startTime));
				}
			});
		}
		downloader.shutdown();
	}
}
```
####8.1
Der Peterson Algorithmus bezweckt den fairen gegenseitigen Ausschluss von Threads mit lediglich Lese-und Schreibzugriffen. Untenstehend ist der Algorithmus für den Fall von 2 Threads angegeben. Der Algorithmus ist zwar in seinem Ablauf prinzipiell korrekt durchdacht, funktioniert aber wegen des Speichermodells basierend auf „Weak Cache Consistency“ nicht richtig.

Welche Effekte dieses Speichermodells können hier zum Fehlverhalten führen?

![Aufgabe1](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/u8.png)

Die Sichtbarkeit der turn Variable in den Threads ist nicht garantiert.
Der Zugriff auf die turn Variable in der while Schleife wird wegoptimiert, daher bleibt der Algorithmus hängen.
```java
package aufgabe1;

public class PetersonMutex {
	private volatile boolean state0 = false;
	private volatile boolean state1 = false;
	private volatile int turn = 0;

	// acquire lock by thread 0
	public void thread0Lock() {
		state0 = true;
		turn = 1;
		while (turn == 1 && state1);
	}

	// release lock by thread 0
	public void thread0Unlock() {
		state0 = false;
	}

	// acquire lock by thread 1
	public void thread1Lock() {
		state1 = true;
		turn = 0;
		while (turn == 0 && state0);
	}

	// release lock by thread 1
	public void thread1Unlock() {
		state1 = false;
	}
}
```

####8.2 Bank Account mit Atomic
```java
package aufgabe2;
//Mit Monitor
public class BankAccount {
	private int balance = 0;

	public synchronized void deposit(int amount) {
		balance += amount;
	}

	public synchronized boolean withdraw(int amount) {
		if (amount <= this.balance) {
			balance -= amount;
			return true;
		} else {
			return false;
		}
	}

	public synchronized int getBalance() {
		return balance;
	}
}
```
```java
package aufgabe2;

import java.util.concurrent.atomic.AtomicInteger;

public class BankAccountAtomic {
	private AtomicInteger balance = new AtomicInteger(0);

	public void deposit(int amount) {
		balance.getAndAdd(amount);
	}

	public boolean withdraw(int amount) {
		int oldValue;
		do {
			oldValue = balance.get();
			if(oldValue < amount){
				return false;
			}
		} while (!balance.compareAndSet(oldValue, oldValue - amount));
		return true;
	}

	public int getBalance() {
		return balance.get();
	}
}
```

####8.3 Lock-freier Stack
```java
package aufgabe3;

import java.util.concurrent.atomic.AtomicReference;

public class FastLockFreeStack<T> implements Stack<T> {

	private AtomicReference<Node<T>> head = new AtomicReference<>(null);

	public void push(T value) {
		Node<T> newNode = new Node<>(value);
		Node<T> current = null;
		do
	    {
	        current = head.get();
	        newNode.setNext(current);
	    }while(!head.compareAndSet(current, newNode));
	}

	public T pop() {
	    Node<T> currentHead = null;
	    Node<T> futureHead = null;
	    do
	    {
	        currentHead = head.get();
	        if(currentHead == null)
	        {
	            return null;
	        }
	        futureHead = currentHead.next;
	    }while(!head.compareAndSet(currentHead, futureHead)); 
	    return currentHead.data;
	}

	class Node<T> {
		private final T data;
		private Node<T> next;

		private Node(T data) {
			this.data = data;
		}

		private void setNext(Node<T> next) {
			this.next = next;
		}
	}
}
```

####11.1
```java
package aufgabe1;

import java.util.concurrent.TimeUnit;

import scala.concurrent.duration.Duration;
import scala.concurrent.duration.FiniteDuration;
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;
import akka.actor.UntypedActor;
/**
a) Ergänzen Sie im Code die Stellen die mit einem TODO markiert sind. Tipp: mit getSender 
	und getSelf erhalten Sie die ActorRef des Senders, bzw. von sich selbst.
b) Erweitern Sie das Programm so, dass das Programm beendet wird sobald der Zähler den Wert 10 erreicht.
c) Im Moment ist unser Actor System noch nicht sehr interessant. Wir ergänzen unser System so, 
	dass in Regelmässigen Abständen eine Reset Message an einen Actor geschickt wird damit dieser den Zähler wieder auf 0 zurück setzt.
*/
public class PingPong {

	static final FiniteDuration FIVE_SECONDS = Duration.create(5,
			TimeUnit.SECONDS);

	public static void main(String[] args) {
		ActorSystem system = ActorSystem.create("PingPong");

		ActorRef p1 = system.actorOf(Props.create(PingPongActor.class), "P1");
		ActorRef p2 = system.actorOf(Props.create(PingPongActor.class), "P2");

		//Send start message
		p1.tell(new Start(), p2);

		// Aufgabe c) Sending a reset message
		system.scheduler().schedule(FIVE_SECONDS, FIVE_SECONDS, p1,
				new Reset(), system.dispatcher(), ActorRef.noSender());
	}

	static class Start {
	}

	static class Reset {
	}

	static class Ping {
		final int count;

		public Ping(int count) {
			this.count = count;
		}
	}

	static class PingPongActor extends UntypedActor {

		private boolean reset = false;

		/**
		* Receive Messagge, handle depending on type.
		*/
		public void onReceive(Object message) {
			if (message instanceof Start) {
				handleStart((Start) message);
			} else if (message instanceof Ping) {
				handlePing((Ping) message);
			} else if (message instanceof Reset) {
				handleReset();
			} else {
				unhandled(message);
			}
		}

		private void handleReset() {
			reset = true;
		}

		private void handlePing(Ping msg) {
			System.out.println(getSelf().path().name() + ": Ping " + msg.count);
			try {
				Thread.sleep((long) (Math.random() * 1000) + 300);
			} catch (InterruptedException e) {
			}

			// Aufgabe b
			if(reset) {
				reset = false;
				getSender().tell(new Ping(0), getSelf());
			} else if (msg.count == 10) {
				//Woche 11 Folie 42
				getContext().system().shutdown();
			} else {
				getSender().tell(new Ping(msg.count + 1), getSelf());
			}
		}

		private void handleStart(Start message) {
			System.out.println("Starting ...");
			getSender().tell(new Ping(0), getSelf());
		}
	}
}
```

####13.4 Vergleich mit anderen Modellen (Theorie)
- Nachdem Sie erste Erfahrungen mit Reactive Programmierung gemacht haben, vergleichen Sie es mit
anderen Parallel-Programmiermodellen:
	- Producer/Consumer mit Threads
	- Asynchrone Programmierung mit Futures
	- Asynchrone Programmierung mit Callbacks
	- Parallele LINQ-Abfragen
	- Actor Model

Welche Ähnlichkeiten und welche Unterschiede haben diese in Bezug auf Reactive Programming?

![Vergleich](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/w13.png)

####14.2 Das Philosophen-Problem (siehe Vorlage) lässt sich klassisch mit binären Semaphoren modellieren.
a) Ändern Sie das Programm, so dass es Software Transaktionen statt Semaphoren einsetzt. Ändern Sie hierzu die Klasse Fork ab.
b) Um die Deadlocks zu vermeiden, ist in der Ursprungsversion eine lineare Sperrordnung nötig:
Fork left = forks[Math.min(leftNo, rightNo)];
Fork right = forks[Math.max(leftNo, rightNo)];
Entfernen Sie dieses Artefakt, indem Sie geschachtelte Transaktionen einsetzen.


#####Semaphor
```java
package task2;

import java.util.concurrent.Semaphore;

class Fork {
	// TODO: Replace semaphore by usage of software transactions
	private Semaphore semaphore = new Semaphore(1);
	
	public void acquire() {
		try {
			semaphore.acquire();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public void release() {
		semaphore.release();
	}
}

class PhilosopherThread extends Thread {
	private final int meals;
	private final Fork left;
	private final Fork right;
	
	
	public PhilosopherThread(int meals, Fork left, Fork right) {
		this.meals = meals;
		this.left = left;
		this.right = right;
	}

	@Override
	public void run() {
		for (int m = 0; m < meals; m++) {
			pickUpForks();
			// eat
			putDownForks();
		}
	}
	
	private void pickUpForks() {
		left.acquire();
		right.acquire();
	}
	
	private void putDownForks() {
		left.release();
		right.release();
	}
}

public class DiningPhilosophers {
	private static long measure(int tableSize, int meals) throws InterruptedException {
		Fork[] forks = new Fork[tableSize];
		for (int i = 0; i < tableSize; i++) {
			forks[i] = new Fork();
		}

		PhilosopherThread[] threads = new PhilosopherThread[tableSize];
		for (int i = 0; i < tableSize; i++) {
			int leftNo = i;
			int rightNo = (i + 1) % tableSize;
			
			// TODO: Remove lock ordering for deadlock avoidance in STM solution
			Fork left = forks[Math.min(leftNo, rightNo)];
			Fork right = forks[Math.max(leftNo, rightNo)];			
		
			threads[i] = new PhilosopherThread(meals, left, right);
		}

		long start = System.currentTimeMillis();

		for (PhilosopherThread thread : threads) {
			thread.start();
		}

		for (PhilosopherThread thread : threads) {
			thread.join();
		}

		return System.currentTimeMillis() - start;
	}

	public static void main(String[] args) throws InterruptedException {
		final int meals = 1_000_000;
		for (int philosopherNo = 0; philosopherNo < 5; philosopherNo++) {
			long elapsed = measure(5, meals);
			System.out.printf("%3.1f us/meal\n", (elapsed * 1000.0) / meals);
		}
	}
}

```

#####Atomic
```java
package task2;

import scala.concurrent.stm.Ref;
import scala.concurrent.stm.japi.STM;
/**
Auch hier ist die STM-Lösung substantiell langsamer; 
dafür wird ein Deadlock mit einer geschachtelten Transaktion in pickUpForks() automatisch vermieden. 
Es braucht daher, die lineare Sperrordnung mit min/max der Gabelnummern nicht mehr.
*/
class Fork {
	private Ref.View<Boolean> inUse = STM.newRef(false);
	
	public void acquire() {
		STM.atomic(() -> {
			if (inUse.get()) {
				System.out.println("Retry");
				STM.retry();
			}
			inUse.set(true);
		});
	}
	
	public void release() {
		inUse.set(false);
	}
}

class PhilosopherThread extends Thread {
	private final int meals;
	private final Fork left;
	private final Fork right;

	public PhilosopherThread(String name, int meals, Fork left, Fork right) {
		this.meals = meals;
		this.left = left;
		this.right = right;
	}

	@Override
	public void run() {
		for (int m = 0; m < meals; m++) {
			pickUpForks();
			// eat
			putDownForks();
		}
	}
	
	private void pickUpForks() {
		STM.atomic(() -> {
			left.acquire();
			right.acquire();
		});
	}
	
	private void putDownForks() {
		left.release();
		right.release();
	}
}

public class DiningPhilosophers {
	private static long measure(int tableSize, int meals) throws InterruptedException {
		Fork[] forks = new Fork[tableSize];
		for (int i = 0; i < tableSize; i++) {
			forks[i] = new Fork();
		}

		PhilosopherThread[] threads = new PhilosopherThread[tableSize];
		for (int i = 0; i < tableSize; i++) {
			int leftNo = i;
			int rightNo = (i + 1) % tableSize;
			
			Fork left = forks[leftNo];
			Fork right = forks[rightNo];			
		
			String name = "Philosopher " + i;
			threads[i] = new PhilosopherThread(name, meals, left, right);
		}

		long start = System.currentTimeMillis();

		for (PhilosopherThread thread : threads) {
			thread.start();
		}

		for (PhilosopherThread thread : threads) {
			thread.join();
		}

		return System.currentTimeMillis() - start;
	}

	public static void main(String[] args) throws InterruptedException {
		final int meals = 1000_000;
		for (int philosopherNo = 0; philosopherNo < 5; philosopherNo++) {
			long elapsed = measure(5, meals);
			System.out.printf("%3.1f us/meal\n", (elapsed * 1000.0) / meals);
		}
	}
}
```

#### RecursiveTask
```java
//Sequenziell
public int getMaximum(int[] array) {
	int max = Integer.MIN_VALUE;
	for (int value: array) {
		max = Math.max(value, max);
	}
	return max;
}
```



```java
//Einsatz des RecursiveTask
public int getMaximum(int[] array) {
	MaxSearchTask task = new MaxSearchTask(array, 0, array.length);
	ForkJoinPool pool = new ForkJoinPool();
	return pool.invoke(task);
}

class MaxSearchTask extends RecursiveTask<Integer> {
	public static final int THRESHOLD = 1000;
	private int[] array;
	private int left, right;

	public MaxSearchTask(int[] array, int left, int right) {
		this.array = array;
		this.left = left;
		this.right = right;
	}

	@Override
	protected Integer compute() {
		if (right - left < THRESHOLD) {
			return sequentialMaximum();
		} else {
			int middle = (left + right) / 2;
			MaxSearchTask taskLeft = new MaxSearchTask(array, left, middle);
			MaxSearchTask taskRight = new MaxSearchTask(array, middle, right);
			taskLeft.fork();
			taskRight.fork();
			int valueLeft = taskLeft.join();
			int valueRight = taskRight.join();
			return Math.max(valueLeft, valueRight);
		}
	}

	private int sequentialMaximum() {
		int max = Integer.MIN_VALUE;
		for (int k = left; k < right; k++) {
			max = Math.max(array[k], max);
		}
		return max;
	}
}
```

####Count Down mit Semaphor
```java
public class CountDownLatch {
	private Semaphore signal = new Semaphore(0);
	private Semaphore mutex = new Semaphore(1, true);
	private int count;
	private int waiting = 0;

	public CountDownLatch(int initialCount) {
		count = initialCount;
	}

	public void await() throws InterruptedException {
		mutex.acquire();
		if (count > 0) {
			waiting++;
			mutex.release();
			signal.acquire();
		} else {
			mutex.release();
		}
	}

	public void countDown() throws InterruptedException {
		mutex.acquire();
		if (count > 0) {
			count--;
			if (count == 0) {
				signal.release(waiting);
			}
		}
		mutex.release();
	}
}
```