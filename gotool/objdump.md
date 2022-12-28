# go tool objdump

反汇编
> objdump disassembles executable files

```sh
go tool objdump -S -s "main" binary
```

```sh
// for example
go build -gcflags=-l main.go
go tool objdump -S -s "main.example" main
```
