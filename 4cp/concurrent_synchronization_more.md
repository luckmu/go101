# concurrent synchronization more

## concurrent synchronization techniques provided in the sync standard package

### sync.WaitGroup

`*sync.WaitGroup`: `Add(delta int)`, `Done()`, `Wait()`

### sync.Once

`*sync.Once`: `Do(f func())`

### sync.Mutex & sync.RWMutex

`*sync.Mutex` & `*sync.RWMutex` both implement `sync.Locker`

`*sync.RWMutex`: ..., `RLock()`, `RUnlock()`

## sync.Cond

`*sync.Cond`: `Wait()`, `Signal()`, `Broadcast()`
