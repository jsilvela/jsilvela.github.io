+++
tags = ["go", "golang"]
categories = []
title = "A simple reusable scheduler in Go"
date = 2021-05-29T09:29:18+01:00
draft = false
+++

Tagline: You (still) don't need a queueing service.

I recently needed a module that would schedule work for a future time on behalf of users, and would execute it at said time. After some thought, I came to an easy design that leveraged the Go standard library.

Some constraints I wanted satisfied:

1. The scheduling should be driven from the database (SQL in my case.)
1. Users should be able to update their desired scheduled times.
1. The schedule updates should be picked up regularly, but not to-the-minute. Say, every 15 minutes.
1. The starting time of the scheduled work should be honored to the minute.

These last two items were not in conflict. Think of an airport updating the schedule of departures once every hour on the hour, though planes may be slated to leave at any time.

I also didn't want any busy-waiting loops in my design.

I knew it should be possible to leverage Go's goroutines, rather than having to rely on a queueing service plus yet another binary / microservice.

## Representing future work

Scheduled work is represented using Go's timers. \
Specifically `func AfterFunc(d Duration, f func()) *Timer` does just what we want: after duration `d`, a goroutine will execute function `f`. Note that the signature `func ()` is as reusable as could be.

Since we want to be able to change the scheduled time of a task, we need to represent the triggering time too.

Here is the basic type I defined:

``` go
// FutureAction represents some action to be taken
// at a specified future time
type FutureAction struct {
    TriggerTime time.Time
    Action      *time.Timer
}
```

With this structure, we can schedule, execute, and delay work.

## Scheduling work

As mentioned at the beginning, the schedule should be driven from the Database, so we created a goroutine sitting in a loop, waiting for the tick of a `time.Ticker` to query the database for new tasks or updated execution times.

The fact that a task has been scheduled should not be written into the Database, though, but be kept in memory as part of our scheduler. This makes it much easier to reason about crash recovery or process restarts.

So, our scheduler should do, roughly (in Go-inspired pseudocode):

``` pseudocode
for {
  // on new tick from the master Ticker
  WorkToSchedule ← read work to schedule from the DB
  for work in WorkToSchedule {
    if work is overdue {
      Do work
    } else if work already scheduled && new trigger time {
      Reschedule work
    } else if work not scheduled {
      Schedule work
    }
  }
}
```

The nature of the actual work is not important for the structure of the loop, which means if we represent it in a reusable manner, we have ourselves a retargetable scheduler.

Go's interfaces are just the thing to use:

``` go
// Work represents a piece of work that may be
// done now, or scheduled for later
type Work interface {
   DoIt()
   ScheduleIt()
   ResetTimer(triggerTime time.Time)
   GetTriggerTime() time.Time
   IsScheduled() (bool, time.Time)
}
```

## The implementation specific parts

What are the implementation-specific bits of this design?

1. The actual work to be performed
1. How we find/store **scheduled** work

As we mentioned before, the fact that some particular task has already been
scheduled by our scheduler should not be written into the database. Rather, it
is information that the scheduler needs to keep.

Imagine that the work to be done is a wake-up call in a hotel. We should be
able to check whether *Room 121-B*’s wake-up call has been scheduled. It makes sense
for the scheduler to keep a `map[string]FutureAction` to represent scheduled
work.

We could define:

``` go
type Room struct {
    PhoneExtension string
    Number         string
}

// WakeUpCall is the Work of giving a wake-up call to a room
//
// roomRepo is the Room database interface
type WakeUpCall struct {
    room          Room
    phoneSchedule map[string]FutureAction
    phoneService  *services.Phone
    roomRepo      *room.Repository
}
```

For instance, to implement the `IsScheduled` method of `Work`:

``` go
func (wuc WakeUpCall) IsScheduled() (bool, time.Time) {
    action, found := wuc.phoneSchedule[wuc.room.Number]
    if !found {
        return false, time.Time{}
    }
    return true, action.TriggerTime
}
```

Performing the work is easy. We should ensure that once Work has been done,
it will not be re-done.

``` go
func (wuc WakeUpCall) DoIt() {
    done := wuc.phoneService.WakeUpCall(wuc.room.PhoneExtension)
    if done {
        delete(wuc.phoneSchedule, wuc.room.Number)
        wuc.roomRepo.ClearWakeUpCall(wuc.room.Number)
    }
}
```

## Conclusion

This design is simple and reusable, as advertised. It can be easily deployed as
an extra goroutine alongside your existing service(s).

It is not trying to solve the issue of scalability; it is not trying to schedule
work among EC2 instances in a server farm, for example. But it would be easy
to use it as a starting point for something more scalable.

Intentionally, the design does not use queues pub-sub or any such scheme. Those
are used too often already. It is always a good idea to read
[*How do you cut a monolith in half?*](https://programmingisterrible.com/post/162346490883/how-do-you-cut-a-monolith-in-half)

> Using a message broker is a tradeoff. Use them freely knowing they work well on the edges of your system as buffers.
