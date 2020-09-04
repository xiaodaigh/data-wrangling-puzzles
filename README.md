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

## Puzzle 2 - Pivot a dataframe to wide format with values in multiple columns

See https://discourse.julialang.org/t/pivot-a-dataframe-to-wide-format-with-values-in-multiple-columns/45916

<details>
```
wide = DataFrame(x = 1:12,
       a  = 2:13,
       b  = 3:14,
       val1  = randn(12),
       val2  = randn(12),
       cname = repeat(["c", "d"], inner =6)
       )

12×6 DataFrame
│ Row │ x     │ a     │ b     │ val1      │ val2      │ cname  │
│     │ Int64 │ Int64 │ Int64 │ Float64   │ Float64   │ String │
├─────┼───────┼───────┼───────┼───────────┼───────────┼────────┤
│ 1   │ 1     │ 2     │ 3     │ 1.51014   │ -1.18548  │ c      │
│ 2   │ 2     │ 3     │ 4     │ 0.0845411 │ -0.370083 │ c      │
│ 3   │ 3     │ 4     │ 5     │ 0.826283  │ -1.00423  │ c      │
│ 4   │ 4     │ 5     │ 6     │ -0.53175  │ -1.16659  │ c      │
│ 5   │ 5     │ 6     │ 7     │ -1.77975  │ 0.336333  │ c      │
│ 6   │ 6     │ 7     │ 8     │ 0.632577  │ 0.236621  │ c      │
│ 7   │ 7     │ 8     │ 9     │ -0.681532 │ 1.14869   │ d      │
│ 8   │ 8     │ 9     │ 10    │ -0.775619 │ 0.393475  │ d      │
│ 9   │ 9     │ 10    │ 11    │ -0.533034 │ 0.059624  │ d      │
│ 10  │ 10    │ 11    │ 12    │ 0.496152  │ -1.23507  │ d      │
│ 11  │ 11    │ 12    │ 13    │ 0.834099  │ 2.12115   │ d      │
│ 12  │ 12    │ 13    │ 14    │ 0.532357  │ -0.369267 │ d      │
```

I am trying to mimic the pivot_wider function in R:

`wide %>% pivot_wider(names_from = cname, values_from = c(val1,val2))`

===  ===  ===  ==========  ==========  ==========  ==========
  x    a    b      val1_c      val1_d      val2_c      val2_d
===  ===  ===  ==========  ==========  ==========  ==========
  1    2    3   1.0174232          NA  -0.6611959          NA
  2    3    4   0.6590795          NA  -2.0954505          NA
  3    4    5   1.2939581          NA   1.6350356          NA
  4    5    6  -1.9395356          NA   0.7813238          NA
  5    6    7   0.3558087          NA   0.9789414          NA
  6    7    8   0.9859100          NA  -0.9803336          NA
  7    8    9          NA   0.4949224          NA  -0.0659333
  8    9   10          NA   0.5024755          NA  -0.2317832
  9   10   11          NA   1.6926897          NA  -0.3840687
 10   11   12          NA  -0.4324705          NA  -0.0901276
 11   12   13          NA  -0.6415260          NA   0.0014151
 12   13   14          NA   1.2406868          NA  -2.1959740
===  ===  ===  ==========  ==========  ==========  ==========
```
</details>


## Puzzle 3

<details>
<summary>Keep only certain rows based on data in group</summary>

I need to select groups of observations from a large dataframe (about 2.9 mio rows) using a number of conditions which apply to different observations in each group (so I cannot select on individual rows only).

Using a small dataframe as a starting point, I wrote an algorithm which applies the conditions and generates the result dataframe with my desired groups within a loop. See mwe below.

If I apply this loop to the large dataframe, performance becomes a (serious) problem.

I haven’t used the split/apply/combine approach before so I am learning about it right now. I worked through the documentation but I haven’t been able to write code for my problem yet.

For example, I am struggling to understand how to select different rows of groupeddataframes. I figured out how to get the age of the status1 row select(combine(first, gdf), :status => :obs1_status) but not for the status2 row.

Any hints/guidance on how to implement selection conditions on groupeddataframes using combine/select/transform commands? (I.e. generate df_result in a faster way?)

Thanks a lot!

```
using DataFrames

# generate sample dataframe
df = DataFrame(id = [1,1,1,2,2,3,4], age = [53,52,17,31,29,22,71], status = [1,2,3,1,2,1,1])

# initialize result dataframe
df_result = copy(df[1:2,:]; copycols=true);

for k = 1:maximum(df.id)
```

</details>

<details>
    <summary> Sample Data </summary>

```julia
df = DataFrame(id = [1,1,1,2,2,3,4], age = [53,52,17,31,29,22,71], status = [1,2,3,1,2,1,1])
```

```
7×3 DataFrame
│ Row │ id    │ age   │ status │
│     │ Int64 │ Int64 │ Int64  │
├─────┼───────┼───────┼────────┤
│ 1   │ 1     │ 53    │ 1      │
│ 2   │ 1     │ 52    │ 2      │
│ 3   │ 1     │ 17    │ 3      │
│ 4   │ 2     │ 31    │ 1      │
│ 5   │ 2     │ 29    │ 2      │
│ 6   │ 3     │ 22    │ 1      │
│ 7   │ 4     │ 71    │ 1      │
```
</details>

<details>
    <summary> Julia Solutions </summary>
    ```julia
    using Pipe, PairAsPipe, DataFramesMeta
df_result = @pipe df |>
    groupby(_, :id) |>
    combine(_,
        @pap(status1and2 = sum(in(1:2), :status)),
        @pap(not_wokring_age = sum(:status .== 1 .& (:age .< 25 .| :age .> 61)))
    ) |>
    @where(_, :status1and2 .== 2, :not_wokring_age .== 0) |>
    @select(_, :id) |>
    innerjoin(_, df; on = :id)
    ```
</details>
