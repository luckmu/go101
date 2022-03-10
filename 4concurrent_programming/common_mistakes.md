# concurrent common mistakes

time.After

```go
// go longrunning(msgs)
// n loops in each goroutine,
// n timer.After() calls, that is n timers
// m goroutines -> m*n timers
func longrunning(msgs <-chan string) {
    for {
        select {
        case <-time.After(time.Minute):
            return
        case msg := <-msgs:
            fmt.Println(msg)
        }
    }
}

// correct using
// timer.C -> make(chan Time, 1)
// goroutine:timer == 1:1
func longrunningi(msgs <-chan string) {
    timer := time.NewTimer(time.Minute)
    defer timer.Stop()

    for {
        select {
        case <-timer.C:
            return
        case msg := <-msgs:
            fmt.Println(msg)
            // while processing, time.Time is sent into timer.C
            // must be expired rather than stopped
            if !timer.Stop() {
                // clear the cache
                <-timer.C
            }
        }
        timer.Reset(time.Minute)
    }
}
```
