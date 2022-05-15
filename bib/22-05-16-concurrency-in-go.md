# Concurrency in go

https://www.cs.princeton.edu/courses/archive/fall16/cos418/docs/P1-concurrency.pdf

## Concurrency

“Concurrency is about dealing with lots of things at once. Parallelism is about
doing lots of things at once” – Rob Pike

En go:

- Soporta dos estilos (a veces CSP no es práctico)
  - Communicating sequential processes (CSP) usa comunicación como primitiva de
    sync
  - Shared mem multithreading usa locks (y cosas de esa naturaleza)
    - `sync.Mutex`
    - `sync.RWMutex`
    - `sync.Once`.

- Razonar sobre concurrencia via partial ordering (happens-before order)
  https://golang.org/ref/mem

## Channels

Sintaxis

```go
ch := make(chan int) // unbuffered channel
ch := make(chan int, 0) // unbuffered channel
ch := make(chan int, 3) // buffered channel with cap 3

// send
ch <- x

// receive
x = <- ch

// close
close(ch)
```

- Cuando se hace un send bloquea hasta que alguna gorutina lo reciba (o hasta
  que esté lleno el buffer si está buffered)

- Cuando se hace un receive bloquea hasta que haya algo para leer (i.e alguna
  gorutina haya hecho send) (puede haber muchas cosas buffereadas).

De esta forma se sincronizan las sending y receiving goroutines.

- Luego de un close, los receives obtienen zero value mientras que los sends
  panic.

## Advanced

- Race detector is part of Go runtime/toolchain
  - Looks for one goroutine accessing shared variable recently written by
  another goroutine without mutex

- Go under the hood
  - Greenthreads with growable stacks multiplexed on OS threads (scheduled by Go
    runtime)
  - Locks wrapped in a threadsafe queue

- When should you use different concurrency models? Can you combine?

## Ejemplos

### Concurrent prime sieve

```go
// A concurrent prime sieve

package main

// Send the sequence 2, 3, 4, ... to channel 'ch'.
func Generate(ch chan<- int) {
	for i := 2; ; i++ {
		ch <- i // Send 'i' to channel 'ch'.
	}
}

// Copy the values from channel 'in' to channel 'out',
// removing those divisible by 'prime'.
func Filter(in <-chan int, out chan<- int, prime int) {
	for {
		i := <-in // Receive value from 'in'.
		if i%prime != 0 {
			out <- i // Send 'i' to 'out'.
		}
	}
}

// The prime sieve: Daisy-chain Filter processes.
func main() {
	ch := make(chan int) // Create a new channel.
	go Generate(ch)      // Launch Generate goroutine.
	for i := 0; i < 10; i++ {
		prime := <-ch
		print(prime, "\n")
		ch1 := make(chan int)
		go Filter(ch, ch1, prime)
		ch = ch1
	}
}
```

https://go.dev/play/p/9U22NfrXeq