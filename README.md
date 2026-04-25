# CatchCake State Machine

CatchCake provides a lightweight and flexible state machine implementation in Elixir. It allows you to define state transitions, handle events, and manage application context without the complexity of external frameworks.

## Overview

The state machine consists of three main parts:
1.  **Definition**: A map structure describing states, events, and transitions.
2.  **Context**: A map that carries data between events and actions.
3.  **Actions**: Functions executed during transitions that can modify the context or control the flow.

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed
by adding `cc_statemachine` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [
    {:cc_statemachine, "~> 1.0.0"}
  ]
end
```

## Defining the State Machine

A state machine is defined using a map where the keys represent states. The `start` key defines the initial state.

*   **`start`**: The initial state. Contains an `init` transition.
*   **`event`**: Defines what happens when a specific event occurs in a state.
    *   `next`: The target state after the event.
    *   `action`: (Optional) A function receiving `(context, event)`.

### Structure Example

```elixir
%{
  start: %{
    init: %{
      next: :next_state,
      action: fn context, _event -> context end
    }
  },
  next_state: %{
    ignite: %{
      next: :fire,
      action: fn context, {:ignite, data} -> Map.put(context, :data, data) end
    }
  }
}
```

## Usage

### Creating a Machine

Use `CatchCake.StateMachine.new/2` to create a machine with a specific ID.
Use `CatchCake.StateMachine.new/3` to initialize it with a context map.

```elixir
machine = CatchCake.StateMachine.new(definition, "my_machine_id")
```

### Handling Events

Call `CatchCake.StateMachine.handle_event/2` to process an event. This updates the current state and executes the corresponding action.

```elixir
machine = CatchCake.StateMachine.handle_event(machine, :run)
```

### Actions

Actions are functions executed during a transition. They receive the current `context` and the `event`.

*   **Return `context`**: The state machine stops processing.
*   **Return `{event, context}`**: The state machine continues processing the same event again (useful for chaining or conditional logic).
*   **Return `context` with data modified**: The context is updated, and the machine stops.

## Examples

### Example 1: Basic State Transition

A simple machine that moves from `start` to `next` upon initialization.

```elixir
iex> definition = %{
...>   start: %{init: %{next: :next}},
...>   next: %{}
...> }
...> machine = CatchCake.StateMachine.new(definition, "test")
...> machine.state
:next
```

### Example 2: Initializing with Context

You can pass initial data to the machine via the context map.

```elixir
iex> definition = %{
...>   start: %{init: %{next: :next, action: fn context, _event -> context end}},
...>   next: %{}
...> }
...> machine = CatchCake.StateMachine.new(definition, "test", %{user_id: 123})
...> machine.context.user_id
123
```

### Example 3: Handling Events with Data

Events can be atoms or tuples containing data. Actions can modify the context based on this data.

```elixir
iex> definition = %{
...>   start: %{init: %{next: :next, action: fn context, _event -> context end}},
...>   next: %{
...>     ignite: %{
...>       next: :fire,
...>       action: fn context, {:ignite, data} -> Map.put(context, :data, data) end
...>     }
...>   }
...> }
...> machine = CatchCake.StateMachine.new(definition, "test")
...> machine = CatchCake.StateMachine.handle_event(machine, {:ignite, "test"})
...> machine.state
:fire
...> machine.context.data
"test"
```

### Example 4: Chaining Events via Actions

An action can return `{event, context}` to trigger the next event immediately after the current one processes.

```elixir
iex> definition = %{
...>   start: %{
...>     init: %{
...>       next: :next,
...>       action: fn context, :init -> {:ignite, context} end
...>     }
...>   },
...>   next: %{
...>     ignite: %{
...>       next: :fire,
...>       action: fn context, _event -> context end
...>     }
...>   },
...>   fire: %{}
...> }
...> machine = CatchCake.StateMachine.new(definition, "test")
...> machine.state
:fire
```

## Summary

1.  **Define** your states and events in a nested map.
2.  **Create** the machine with `new/2` or `new/3`.
3.  **Process** events using `handle_event/2`.
4.  **Control** flow via action return values (`context` to stop, `{event, context}` to continue).

Documentation can be generated with [ExDoc](https://github.com/elixir-lang/ex_doc)
and published on [HexDocs](https://hexdocs.pm). Once published, the docs can
be found at <https://hexdocs.pm/cc_statemachine>.

