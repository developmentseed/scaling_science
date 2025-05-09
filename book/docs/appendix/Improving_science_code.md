# Improving Science Code

"Improving code" means not only making code faster and more reliable, but also making code easier to debug, refactor, update, and understand. 

If you are reading this in conjunction with [Assessing Science Code](./Assessing_science_code.md) then you can think of it as a continuous cycle. Assessing the code should give insight about how to improve it, improving the code should make it easier to assess.

Most of this involves rewriting sections of your code. It is not always the right time to do that, but when it is here are some things to keep in mind.

## Code Organization

### Pipelines

Often code is structured around science concepts, but it can be helpful to think instead in terms of pipelines. To organize your code by pipelines ask:

1. What are the inputs to this workflow?
2. What are the outputs from this workflow?
3. What are the steps for getting from inputs to outputs?

When you organize code by pipeline steps rather than science concepts it makes it easier to see steps that can be independent of each other (which means you can parallelize them). It also makes it easier to see what the inputs and outputs are for each step.

Whenever it is practical, you should consider storing the output of each pipeline step. This let's you pick back up in the middle of the workflow if something goes wrong with a later step. Storing intermediary outputs can also help with debugging since you can see what you had at each stage.

### Functions

Has this happened to you? You start reading a function to try to figure out what is going on and then you realize that function relies on another function which relies on another one and another one... 

It's not uncommon for code to be structured in that way and often it has to do with a desire for code to be extremely DRY (Do not Repeat Yourself) or to have a consistent level of abstraction at every level. Both of these concepts are good in theory, but in practice they can make code hard to debug. 

Here are some questions to ask when you are writing a function:

- Is it going to be used more than three times?
- Does it get different inputs each time it is used?
- Does it make the code easier to understand?
- Is the function more than 4 lines long?

If one or two of these are "no" then consider whether a function is really needed.

## Code Contents

### Naming
You can make any code more legible by using variable names that are clear and descriptive. This means using full words and substituting underscores for spaces.

| Do | Don't |
| -- | ----- |
| `region_name = "USA"` | `regnm = "USA"` |


### Constants
Move science parameters into one file and import them directly from there in the places where you need them. Constants should not be passed as parameters to functions. Consider using a library like [pydantic-settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/) to make it easy to override constants.

### Making code faster
Writing and reading code take time just the same way that running code takes time, so if a function works and runs for 10 minutes once a year then maybe it doesn't need to be faster. Another way of putting it is: the more often a piece of code runs, the more valuable it is to make it run fast.

If you have decided that it is worthwhile to speed up your code the first thing to do is look for for-loops. Go to the deepest nested for-loop and see what happens in there. If it is math, see whether you can use vectors (for instance numpy arrays) rather than iterating. If there are function calls, think about what the inputs to the functions will be. Ideally you should only run a function if there are new inputs. That includes IO so if you are repeatedly reading the same file within a for-loop, consider moving the read up and out.

If you aren't sure which approach will be faster create a minimal example (or even better a test) and try it both ways and measure how long each takes.

## Code Output

When writing output make sure to use stable and popular file formats. Here are some examples:
 
 - for dictionary-like data: json
 - for tables: parquet
 - for rasters: tiff or COG
 - for nd arrays: zarr

When writing output you often have control over how the data gets compressed and how each datatype is represented (think 8 bit vs 64 bit). You probably don't need to think about that when you first write a file, but if you are trying to make reads and writes more performant, that is something to consider. 

### The case against Python Pickles
Any reputable file format has a detailed specification and makes strong promises about backwards compatibility. This means that you will always be able to read your data. An example of a storage option that doesn't make such strong guarantees is a Python pickle. Pickles can be read back in _as long as the environment has not changed_. In addition to potentially trapping you with old versions of software, pickles are also terrible for reproducibility because it can be very hard to recreate exactly the same environment on a different machine.

### Write to local - copy to cloud object storage (e.g. s3)
Reading straight from s3 is great! It is a core to cloud-native IO concepts. But when writing, we recommend that you write things locally first and then copy them to s3 - or to any cloud object storage.
