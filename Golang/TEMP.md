runtime.Metrics
go tool trace

go tool pprof cpu.prof
	команды профилировщика:
		- top5
		- hide=runtime
		- granularity=lines
		- list=profiling/factorial.Recursive
		- tree

 go tool pprof -http=6060 cpu.prof



