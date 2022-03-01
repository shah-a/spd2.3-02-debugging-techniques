# Debug Log

**Explain how you used the the techniques covered (Trace Forward, Trace Backward, Divide & Conquer) to uncover the bugs in each exercise. Be specific!**

In your explanations, you may want to answer:

- What is the expected vs. actual output?
- If there is a stack trace, what useful information does it contain?
- Which technique did you use, on which line numbers?
- What assumptions did you have about each line of code, and which ones were proven to be wrong?

_Example: I noticed that the program should show pizza orders once a new order is made, and that it wasn't showing any. So, I used the trace forward technique starting on line 13. I discovered the bug on line 27 was caused by a typo of 'pzza' instead of 'pizza'._

_Then I noticed another bug ..._

## Exercise 1

### Bug 1 and 2

Before parsing the code, I ran the program to see what bugs are most apparent. I know there are multiple bugs in this program, so I am going to divide and conquer them into separate parts as I work to solve each of them.

Upon submitting the pizza order form, the server threw an error: `"TypeError: 'topping' is an invalid keyword argument for PizzaTopping"`. The error message specifies that the problem was encountered on line 79. Since we know where the error originates, the strategy I'm choosing to use for debugging is to trace backwards from line 79.

The KWARG `topping` is causing an error because there is no field in our PizzaTopping model named `topping`. It's supposed to be `ToppingType`. So to solve this bug, I rewrote the line as:

```python3
for topping_str in toppings_list:
    pizza.toppings.append(PizzaTopping(topping_type=ToppingType[topping_str]))
```

Note that this above solution also solves a second bug in this program: the original `for` loop was written as `for topping_str in ToppingType`, but this would cause every order to contain every available topping rather than reading the toppings from the form submitted by the user. Instead, I changed the for loop to use the `toppings_list` variable which should contain a list of the user's selected toppings.

### Bug 3

The next bug I encountered was that the `toppings_list` variable was not correctly being parsed as a list of values from the form input. I traced forward from its declaration to try and see what the problem was. The problem was solved by changing the method used to get the form input from `request.form.get` to `request.form.getlist`.

While we're here, it's a good idea to double check that the html form control elements having matching name attributes to the parameters used in the python code to get those values.

There were some incorrect references which would lead to receiving `None` values for some form inputs.

I made the following changes.
- `order_name = request.form.get('name')` -> `order_name = request.form.get('order_name')`
- `pizza_size_str = request.form.get('size')` -> `pizza_size_str = request.form.get('pizza_size')`

### Bug 4

The next bug was the following error: `Could not build url for endpoint '/'. Did you mean 'fulfill_order' instead?`

The strategy I used to solve this bug was to trace backwards from line 84. It appears that the function was being given a URL endpoint, but since `url_for()` is being used, it should be provided the name of the function rather than the URL endpoint.

I fixed the bug by making the following change: `return redirect(url_for('/'))` -> `return redirect(url_for('home'))`

### Bug 5

The last bug I found was that the orders being placed were not being saved to the database correctly. This is suggested by (1) the orders not appearing on the homepage after being placed, and (2) the database not reporting any changes to Git version control when orders are placed. Alternatively, a SQLite database browser could be used to confirm that no changes are being made to the database.

To solve this bug, I traced forward until I found where the database was being manipulated. On line 81, the pizza is being added to the database, but there should be a line following it that performs a `commit` operation. This was missing. I solved the bug by adding it.

## Exercise 2

This program throws an error with the following message: `KeyError: 'name'` and it specifies that the error originates on line 52 in forming the dictionary object using the API call `result_json` variable.

I traced backwards from this line to figure out why the API call result was not being parsed correctly.

On line 42 where the `params` dictionary is defined, the field it uses for the city name has a key of `place`. According to the OpenWeatherMap API, this key should be named `q` rather than `place`. I fixed the bug by renaming it.

I continued to trace backwards and found that lines 39 and 40 are saving the values sent by the user from the html form elements, but the strings for the name attributes don't match up with what's actually in the html elements. I fixed this bug by making the following changes:
- `city = request.args.get('users_city')` -> `city = request.args.get('city')`
- `units = request.args.get('requested_units')` -> `units = request.args.get('units')`

## Exercise 3

For starters, we know we have two functions so we can divide and conquer and solving each one individually. Let's start with `merge_sort`:

### Merge Sort

When running the program, an error is thrown from line 37 saying that the list index is out of range. Since we know the exact location of the problem, we can try tracing backwards from there.

When checking for left over items, one of the while loops uses `i` as its index variable (like so: `arr[k] = right_side[i]`) and the second one is supposed to use `j` (like so: `arr[k] = right_side[j]`). However, in this program, both of the loops are using `i` and this would cause problems with the index count.

I resolved this issue by updating the second while loop to use `j` as intended. There's also an issue in both while loops that the `k` counter is not being incremented as it was done so in the very first while loop where elements are copied to temporary arrays for the left and right side.

If we re-run the program, we see that the list is being sorted correctly. However, its order is reversed (i.e. it sorts from largest to smallest rather than smallest to largest as expected). To solve this, we can trace forward from the beginning of the function until we find the part responsible for making element comparisons.

We find this to be on line 23 and we can resolve the issue by changing `if left_side[i] > right_side[j]:` to `if left_side[i] < right_side[j]:`

Now, to solve `binary_search`:

### Binary Search

When we run the program, we have an error on line 53 that sasys `TypeError: list indices must be integers or slices, not float`

We can trace backwards from line 53 until we find where the list index variable `mid` is assigned a non-integer value. We find that on line 50, the `mid` variable is being assigned the result of a division operator. The intended action here was an integer division, so we can solve the bug by changing `mid = (high + low) / 2` to `mid = (high + low) // 2`

There is also a bug in some cases where a false negative is reported by the searching algorithm.

To find the source of the bug, we can trace forward from the beginning of the algorithm.

When we get to the line where comparisons are being made between the high and low ends of the index range, the condition is `while low < high:`, but this will cause certain scenarios where elements will be skipped.

To resolve this issue, I updated the condition to be: `while low <= high:`
