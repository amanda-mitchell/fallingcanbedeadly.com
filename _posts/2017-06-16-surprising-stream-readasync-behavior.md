---
title: What Happens in the Default Implementation of Stream.ReadAsync Will Shock You
layout: post
---

> tl;dr The base implementation of `Stream.ReadAsync(byte[] buffer, int offset, int count, CancellationToken cancellationToken)` doesn't do what you might reasonably expect with respect to cancellation.
> Make sure you check whether your `Stream` subclass provides a proper implementation before relying on it.

A common pattern when cancelling a `Task` is to then wait for it to complete.
This ensures that any unmanaged resources that the `Task` might be using are then safe to clean up:

```csharp
cancellationTokenSource.Cancel();

try
{
  task.GetAwaiter().GetResult();
}
catch (OperationCanceledException)
{
}

// dispose of other stuff...
```

However, this pattern can lead to application hangs if the `Task` in question doesn't do a good job of supporting cancellation, and the .NET Framework makes it surprisingly easy to fall into Pits of Failure.
One such pit of failure is `Stream.ReadAsync`.

## Background

I recently needed to build a system to communicate between a couple of processes running on the same machine.
Eventually, I settled on using Windows' named pipes, which are exposed in the .NET Framework as `NamedPipeServerStream` and `NamedPipeClientStream`.

For both ends of the pipe, I created a simple manager class that creates a `Task` that handles connecting/reconnecting to the pipe and listening for messages from the other end until canceled.
In the `Dispose` method of the manager, I followed the pattern described above.
In the `Task`, the `CancellationToken` was checked at each iteration, and it was also passed to all async calls.
To a casual reader, the implementation appeared flawless.

The processes also both hung on `Dispose` if they ever succeeded in connecting to each other.
Pausing the process in the debugger indicated that the hanging thread was waiting for the `Task` to finish, and no threads were currently associated with the `Task`.

## Huh?

In terms of the [Win32 API](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365590(v=vs.85).aspx), a named pipe is *mostly* just a file:
named pipe clients can even create them by calling `CreateFile` directly.
This means that under the hood, they support nearly all of the same operations as files, including overlapped I/O and `CancelIoEx`.

If you take a look at .NET Core's implementation, you'll even find that the Windows-specific implementation of
[`ReadAsync`](https://github.com/dotnet/corefx/blob/bfac45a/src/System.IO.Pipes/src/System/IO/Pipes/PipeStream.Windows.cs#L84-L136)
creates an instance of a subclass of `TaskCompletionSource`
[that uses these features](https://github.com/dotnet/corefx/blob/b9953de/src/System.IO.Pipes/src/System/IO/Pipes/PipeCompletionSource.cs#L137-L151).
You could easily be forgiven for assuming that the .NET Framework would be implemented in exactly the same way.

But you'd be wrong.

Instead, the .NET Framework, as of .NET 4.7, provides no override of `ReadAsync`, which means that it delegates to `Stream`'s
[default implementation](https://github.com/dotnet/coreclr/blob/ebda0e6/src/mscorlib/src/System/IO/Stream.cs#L410-L417).
This implementation checks the value of the `CancellationToken` at the beginning of the method;
if it has not been triggered, it discards the token and delegates to a `BeginRead`/`EndRead`-based implementation of async I/O.

This is what caused my hang.

## Workarounds

I'm not aware of a general-purpose workaround that will work for every subclass of `Stream`.

In the case of `NamedPipeClientStream` and `NamedPipeServerStream`, it is possible to `Dispose` the stream during an asynchronous read, which will have the effect of terminating the read.
If you do this, you may also need to be prepared to catch an `ObjectDisposedException` when waiting for your `Task` to complete.

You could also use P/Invoke to drop down directly to the Win32 API to create a named pipe as a `SafeFileHandle` and pass it to the `FileStream` constructor:
`FileStream` *does* override `ReadAsync` in terms of overlapped I/O and `CancelIoEx`, so you could call it without worrying about your `Task` becoming orphaned.
On the other hand, you'd also have to continue dropping to the Win32 API to handle all of the connection-oriented logic.

In any case, if you're writing code that deals with `Stream`s and needs to handle cancellation, make sure your subclass properly supports it!
