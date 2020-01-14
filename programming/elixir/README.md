# Elixir

## Notable Libraries

- [Oban Job System](https://github.com/sorentwo/oban)
- [Tesla](https://github.com/teamon/tesla) HTTP Client, better reputation than HTTPoison
- [Jason](https://github.com/michalmuskala/jason) JSON parser/generator,
  better reputation than Poison

## Misc Links

- [Map to Struct, forum thread](https://elixirforum.com/t/how-to-get-struct-from-map-elixir/4543/29)

## Parallel Processing Strategies

- [Task.async_stream/2](https://hexdocs.pm/elixir/Task.html)
- [Flow](https://hexdocs.pm/flow/Flow.html)
- [GenStage](https://github.com/elixir-lang/gen_stage)
- [Broadway](https://github.com/plataformatec/broadway)

## Compile Elixir to Native Code

Add this to the top of an arithmatic based module.

```elixir
    @compile [:native, {:hipe, [:verbose, :o3]}]
```

## Basic GenServer Template

```elixir
defmodule ElixirDay07.ComputerServer do
  use GenServer

  def start do
    GenServer.start(__MODULE__, nil)
  end

  def put(pid, key, value) do
    GenServer.cast(pid, {:put, key, value})
  end

  def get(pid, key) do
    GenServer.call(pid, {:get, key})
  end

  #### Implementation ####

  def init(_) do
    {:ok, %{}}
  end

  def handle_cast({:put, key, value}, state) do
    {:noreply, Map.put(state, key, value)}
  end

  def handle_call({:get, key}, _from, state) do
    {:reply, Map.get(state, key), state}
  end
end
```

```
Created:       Tue 05 Nov 2019 06:45:54 PM CST
Last Modified: Tue 14 Jan 2020 07:19:08 AM CST
```
