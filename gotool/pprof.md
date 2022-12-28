# pprof

https://www.youtube.com/watch?v=nok0aYiGi

```go
import "github.com/pkg/profile"

defer profile.Start(profile.CPUProfile, profile.ProfilePath(".")).Stop()
// go tool pprof -http=:8080 cpu.pprof

defer profile.Start(profile.MemProfile, profile.ProfilePath(".")).Stop()
// go tool pprof -http=:8080 mem.pprof

defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
// go tool trace trace.out
```
