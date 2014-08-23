-Fragen
===================

##Grundlage
Was ist der Unterschied zwischen User-Level Threads und Kernel Level Threads?

Zeichne die Zusammenhänge zwischen den drei möglichen Threds Zustände auf.
![Threadzustände](https://github.com/suizo12/-HSR.modules.PnProg/blob/master/images/threadzustaende.png)
	
Was sind Deamon Threads?

Wie kann die JVM Terminiert werden?
- Wenn keine Threads mehr vorhanden sind.
- System.exit()
- Runtime.exit()

Wie wird ein Thread in Java erstellt?
```java
//thread implementation
class SimpleThread extends Thread {
	@Override
	publicvoidrun() {
		// threadbehavior
	}
}

//create thread
Thread myThread= newSimpleThread();
//start thread
myThread.start();
```
