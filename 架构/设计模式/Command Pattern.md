# Command Pattern

### Intent

- Encapsulate a request as an object, thereby letting you parametrize clients with different requests, queue or log requests, and support undoable operations.
- Promote "invocation of a method on an object" to full object status
- An object-oriented callback

### Problem

Need to issue requests to objects without knowing anything about the operation being requested or the receiver of the request.

### Discussion

Command decouples the object that invokes the operation from the one that knows how to perform it. To achieve this separation, the designer creates an abstract base class that maps a receiver (an object) with an action (a pointer to a member function). The base class contains an `execute()` method that simply calls the action on the receiver.

All clients of Command objects treat each object as a "black box" by simply invoking the object's virtual `execute()` method whenever the client requires the object's "service".

A Command class holds some subset of the following: an object, a method to be applied to the object, and the arguments to be passed when the method is applied. The Command's "execute" method then causes the pieces to come together.

Sequences of Command objects can be assembled into composite (or macro) commands.

### Structure

The client that creates a command is not the same client that executes it. This separation provides flexibility in the timing and sequencing of commands. Materializing commands as objects means they can be passed, staged, shared, loaded in a table, and otherwise instrumented or manipulated like any other object.

![Command scheme](https://sourcemaking.com/files/v2/content/patterns/Command.svg)

Command objects can be thought of as "tokens" that are created by one client that knows what need to be done, and passed to another client that has the resources for doing it.

### Example

The Command pattern allows requests to be encapsulated as objects, thereby allowing clients to be parametrized with different requests. The "check" at a diner is an example of a Command pattern. The waiter or waitress takes an order or command from a customer and encapsulates that order by writing it on the check. The order is then queued for a short order cook. Note that the pad of "checks" used by each waiter is not dependent on the menu, and therefore they can support commands to cook many different items.

![Command example](https://sourcemaking.com/files/v2/content/patterns/Command_example1.svg)

### Check list

1. Define a Command interface with a method signature like `execute()`.
2. Create one or more derived classes that encapsulate some subset of the following: a "receiver" object, the method to invoke, the arguments to pass.
3. Instantiate a Command object for each deferred execution request.
4. Pass the Command object from the creator (aka sender) to the invoker (aka receiver).
5. The invoker decides when to `execute()`.

### Rules of thumb

- Chain of Responsibility, Command, Mediator, and Observer, address how you can decouple senders and receivers, but with different trade-offs. Command normally specifies a sender-receiver connection with a subclass.
- Chain of Responsibility can use Command to represent requests as objects.
- Command and Memento act as magic tokens to be passed around and invoked at a later time. In Command, the token represents a request; in Memento, it represents the internal state of an object at a particular time. Polymorphism is important to Command, but not to Memento because its interface is so narrow that a memento can only be passed as a value.
- Command can use Memento to maintain the state required for an undo operation.
- MacroCommands can be implemented with Composite.
- A Command that must be copied before being placed on a history list acts as a Prototype.
- Two important aspects of the Command pattern: interface separation (the invoker is isolated from the receiver), time separation (stores a ready-to-go processing request that's to be stated later).



### Java reflection and the Command design pattern

**Motivation.** "Sometimes it is necessary to issue requests to objects without knowing anything about the operation being requested or the receiver of the request." The Command design pattern suggests encapsulating ("wrapping") in an object all (or some) of the following: an object, a method name, and some arguments. Java does not support "pointers to methods", but its reflection capability will do nicely. The "command" is a black box to the "client". All the client does is call "`execute()`" on the opaque object.

```java
import java.lang.reflect.*;

public class CommandReflect {
   private int state;
   public CommandReflect( int in ) {
      state = in;
   }
   public int addOne( Integer one ) {
      return state + one.intValue();
   }
   public int addTwo( Integer one, Integer two ) {
      return state + one.intValue() + two.intValue();
   }

   static public class Command {
      private Object   receiver;               // the "encapsulated" object
      private Method   action;                 // the "pre-registered" request
      private Object[] args;                   // the "pre-registered" arg list
      public Command( Object obj, String methodName, Object[] arguments ) {
         receiver = obj;
         args = arguments;
         Class cls = obj.getClass();           // get the object's "Class"
         Class[] argTypes = new Class[args.length];
         for (int i=0; i < args.length; i++)   // get the "Class" for each
            argTypes[i] = args[i].getClass();  //    supplied argument
         // get the "Method" data structure with the correct name and signature
         try {      action = cls.getMethod( methodName, argTypes );      }
         catch( NoSuchMethodException e ) { System.out.println( e ); }
      }
      public Object execute() {
         // in C++, you do something like --- return receiver->action( args ); 
         try {     return action.invoke( receiver, args );     }
         catch( IllegalAccessException e    ) { System.out.println( e ); }
         catch( InvocationTargetException e ) { System.out.println( e ); }
         return null;
   }  }

   public static void main( String[] args ) {
      CommandReflect[] objs = { new CommandReflect(1), new CommandReflect(2) };
      System.out.print( "Normal call results: " );
      System.out.print( objs[0].addOne( new Integer(3) ) + " " );
      System.out.print( objs[1].addTwo( new Integer(4),
                                        new Integer(5) ) + " " );
      Command[] cmds = {
         new Command( objs[0], "addOne", new Integer[] { new Integer(3) } ),
         new Command( objs[1], "addTwo", new Integer[] { new Integer(4),
                                                         new Integer(5) } ) };
      System.out.print( "\nReflection results:  " );
      for (int i=0; i < cmds.length; i++)
          System.out.print( cmds[i].execute() + " " );
      System.out.println();
}  }
```

#### Output

```
Normal call results: 4 11     // 1 + 3 = 4     // 2 + 4 + 5 = 11
Reflection results:  4 11
```