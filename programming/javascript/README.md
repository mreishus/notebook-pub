# Javacript

Javascript: Taking over the world.

## JSON

### JQ Related

- [jq](https://stedolan.github.io/jq/) - Sed for online data.
- [gojq](https://github.com/itchyny/gojq) - Go implementation of JQ.
- [jqview](https://github.com/fiatjaf/jqview) - Simple GUI for inspecting Json
  with JQ.
- [jqq](https://github.com/jcsalterego/jqq/) - Interactive wrapper around JQ.
- [jiq](https://github.com/fiatjaf/jiq) - Another interactive querier.

### JQ Alternatives

- [gron](https://github.com/TomNomNom/gron) - Make Json greppable (or make it
  pretty).  Can't transform like JQ but maybe easier for simpler "I need to read
  this" tasks.
- [jql](https://github.com/cube2222/jql) - Lispy variant.

### Live REPL for JQ

``` sh
echo '' | fzf --print-query --preview "cat *.json | jq {q}"
```

See [this page](https://paweldu.dev/posts/fzf-live-repl/) for more repl
techniques with fzf.

### Other JSON CLI utilities

- [jo](https://github.com/jpmens/jo) - Create JSON from shell output.
- [catj](https://github.com/soheilpro/catj) - Displays JSON files in a flat format.

## Misc

- [RoughNotation](https://roughnotation.com/) Library for hand drawn emphasis.

```
Created:       Thu 28 May 2020 06:08:02 PM CDT
Last Modified: Sat 06 Jun 2020 06:24:12 AM CDT
```
