# data-wrangling-puzzles

## Puzzle 1 - Messed up column names need distribution

[Orig link](https://discourse.julialang.org/t/how-would-i-remove-a-column-from-a-dataframe-by-distributing-its-values-among-existing-columns/44265).

From 

```julia
df = DataFrame(:person=>["bob","phil","nick"],:london=>[1,1,0],:spain=>[1,0,0],Symbol("london,spain")=>[1,1,1])
```
to

```julia
df = DataFrame(:person=>["bob","phil","nick"],:london=>[2,2,1],:spain=>[2,1,1])
```

### Julia solution

<details>

```julia
function row_spread(row)
    dict = Dict{String, Int}()
    for (colname, val) in zip(keys(row), values(row))
        if colname == :person
            continue
        end
        for country in split(string(colname), ",")
            dict[country] = get(dict, country, 0) + val
        end
    end
    new_row = hcat(DataFrame(person = row.person), DataFrame(dict))
    new_row
end

new_df = reduce(vcat, row_spread(row) for row in eachrow(df))
```

</details>
