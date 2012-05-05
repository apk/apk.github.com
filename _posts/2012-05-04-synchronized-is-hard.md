---
title: synchronized is hard
layout: post
excerpt: How thinking multithreaded isn't easy, and doesn't scale.
---

# synchronized is hard

I had a java problem that looked trivial at first, but quickly becomes
convoluted when taking into account the necessity of thread safety.
The problem: You get tuples of (`Runnable`, `Object`), and want to
serialize the execution of the `run()` method of all Runnables you
get for a given Object. (The object is just an identifier, an in
the given case was an integer identifying a stream to work on.)

The obvious decomposition is to have a map that maps from the
identifier object to a serializer. For each new object we look
up a serializer with the identifier or create a new one if none
is there yet and put it into the map, then throw the runnable into
that serializer.

{% highlight java %}
public class SerialRunner {
  private HashMap <Object, ExecQueue> queuemap =
    new HashMap<Object, ExecQueue> ();
  public void add (final Object id, final Runnable run) {
    synchronized (queuemap) {
      ExecQueue q = queuemap.get (id);
      if (q == null) {
        q = new ExecQueue ();
        queuemap.put (id, q);
      }
      q.add (run);
   }
}
{% endhighlight %}

The serializer itself just needs a method to add runnables to execute
and some internal workings to actually execute those runnables. In
addition we need a way to tell our client that all work is done,
and that it can be removed from the map.

{% highlight java %}
public abstract class ExecQueue {
  private Thread thread = null;
  private Vector<Runnable> list = new Vector<Runnable> ();

  protected abstract void done ();

  private synchronized Runnable getjob () {
    if (queue.size () > 0) {
      return queue.remove (0);
    }
    return null;
  }

  public synchronized void add (Runnable r) {
    list.addElement (r);
    if (thread == null) {
      thread = new Thread () {
        public void run () {
          while (true) {
            Runnable r = getjob ();
            if (r == null) break;
            r.run ();
          }
          done ();
        }
      };
      thread.start (); // the thread
    }
  }
}
{% endhighlight %}

For the exit indication I added the abstract `done()` method which every
client has to override, thus the `SerialRunner` has to implement it:

{% highlight java %}
    synchronized (queuemap) {
      ExecQueue q = queuemap.get (id);
      if (q == null) {
        q = new ExecQueue () {
          public synchronized void done () {
            queuemap.remove (id);
          } 
        }
        queuemap.put (id, q);
      }
      q.add (run);
   }
{% endhighlight %}

And it all works (this is whiteboard code, not compiled). Except
that, once in a while, it will drop a runnable. Can you see why?

The problem is that the `done()` callback is invoked after `getjob()`
noted that the queue is empty, but nothing keeps the `SerialRunner`
from putting another Runnable into the `ExecQueue` between the detection
and the callback, and that one will be lost. `ExecQueue` could check
for that and complain, or execute that Runnable (there may even be more
than one), but still `SerialRunner` will put new runnables for that
object on a different `ExecRunner`, and thus the serialization guarantee
is violated.

If we changed `ExecQueue` so that the `done()` callback is called
from within the synchronized `getjob()` we'd only get into the
hard place. Instead of dropping a Runnable we'd now deadlock: The
`ExexQueue` thread holds its own lock and wants to lock the `SerialRunner`
(by the `done()` callback), which `add()` holds the `SerialRunner` lock
and wants to obtain the lock of `ExecQueue`.

And there is no way to fix that by only juggling with the `synchronized`.
If you release the `SerialRunner` lock before putting the runnable into
the obtained `ExecQueue` you again run the risk of the `ExecQueue` getting
`done()` in between, and thus dropping the Runnable.

You need to change the API; the `done()` callback design, which looks innocent
from a single-threaded perspective simply can't work. Either you change `add()`
of `ExecQueue` to return a boolean whether the queue is actually still active
and have `SerialRunner`s add retry completely (creating a new `ExecQueue`
and map entry), or you need to need to more tightly couple the two classes
so that, instead of the `done()` callback, both objects are locked in the
proper sequence, and the removal is then done. (I chose the latter one.)

## Conclusion

This relatively simple problem very easily gets out of hand in more
complicated applications, and reasoning about possible deadlocks requires
knowledge of *all* the component's synchronization behaviour. As soon
as the flow of control traverses in different directions through the
object graph (which is typical of GUI toolkits) such finegrained
synchronization becomes practically impossible.
