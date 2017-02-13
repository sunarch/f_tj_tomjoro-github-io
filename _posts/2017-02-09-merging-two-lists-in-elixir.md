---
layout: post
title: Merging two sorted lists in Elixir
image:
tags: [elixir, functional programming]
comments: true
---

<script src="https://gist.github.com/tomjoro/c53f455cb14a84172589b5d5782e9eb7.js"></script>

What is a continuation?

http://elixir-lang.org/blog/2013/12/11/elixir-s-new-continuable-enumerators/

```elixir
defmodule Interleave do
  def interleave(a, b) do
    step = fn x, acc -> { :suspend, [x | acc] } end
    af = &Enumerable.reduce(a, &1, step)
    bf = &Enumerable.reduce(b, &1, step)
    do_interleave(af, bf, []) |> :lists.reverse()
  end

  defp do_interleave(a, b, acc) do
    case a.({ :cont, acc }) do
      { :suspended, acc, a } ->
        case b.({ :cont, acc }) do
          { :suspended, acc, b } ->
            do_interleave(a, b, acc)
          { :halted, acc } ->
            acc
          { :done, acc } ->
            finish_interleave(a, acc)
        end
      { :halted, acc } ->
        acc
      { :done, acc } ->
        finish_interleave(b, acc)
    end
  end

  defp finish_interleave(a_or_b, acc) do
    case a_or_b.({ :cont, acc }) do
      { :suspended, acc, a_or_b } ->
        finish_interleave(a_or_b, acc)
      { _, acc } ->
        acc
    end
  end
end
```

Interleave.interleave([1, 2], [:a, :b, :c, :d])
#=> [1, :a, 2, :b, :c, :d]
