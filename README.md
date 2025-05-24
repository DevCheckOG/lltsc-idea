# LLVM TypeScript Compiler (``lltsc``)

The idea is to develop a compiler that compiles TypeScript source code into native code, enabling interoperability with JavaScript via a JIT, which is the most accessible solution.

## Why?

There are many reasons why returning to TypeScript as a true language and compiling to native code would have a huge impact on the software world.

Among these reasons, we have:

- ``TypeScript`` is a ghost language regarding execution, as it is usually transferred to JavaScript, having the same performance as JavaScript but with another compilation on top: TS -> JS -> V8 JIT (for example).
- ``TypeScript`` had to be rewritten in ``Go`` because it compiled too long. However, the runtime is the same.
- ``TypeScript`` lacks capabilities for __``embedded systems``__; enabling this would improve the quality of the language and its uses.
- ``TypeScript`` has a large codebase that can be used in areas not explored by very high-level languages ​​like JavaScript.

## How do it?

First of all, we need to consider the language we're going to use to build the compiler. For that, we have two options: Rust or C#. However, C# would be preferable to avoid extreme complexity and verbosity when developing the compiler for Rust.

We will use the LLVM-C API, to have a solid interoperability with ``C``.

We can find bindigs for this at: https://github.com/dotnet/LLVMSharp

#### How will we make the compiler work with JavaScript?

First, by default, it will compile to native code if there are only TypeScript packages around the project. This would be AOT, since it won't need to interoperate with JS at any point during compilation.

To interoperate with JavaScript, it would have to be JIT compiled, for several obvious reasons because it is a very dynamic language:

- Compiling JavaScript from AOT to machine code is a complete waste, due to the dynamic nature of JavaScript, which would require creating multiple versions of the same function. For example:

```js
// Function that processes different types of arguments
function processInput(input) {
  // Check if input is a string
  if (typeof input === 'string') {
    return `String received: ${input.toUpperCase()}`;
  }
  // Check if input is a number
  else if (typeof input === 'number') {
    return `Number received: ${input * 2}`;
  }
  // Check if input is an array
  else if (Array.isArray(input)) {
    return `Array received: ${input.join(', ')}`;
  }
  // Check if input is an object (but not an array or null)
  else if (input && typeof input === 'object' && !Array.isArray(input)) {
    return `Object received: ${JSON.stringify(input)}`;
  }
  // Check if input is a function
  else if (typeof input === 'function') {
    return `Function received: ${input()}`; // Execute the function
  }
  // Handle undefined or other types
  else {
    return `Unknown input type: ${input}`;
  }
}

// Test cases with different argument types
console.log(processInput("hello")); // String received: HELLO
console.log(processInput(42)); // Number received: 84
console.log(processInput([1, 2, 3])); // Array received: 1, 2, 3
console.log(processInput({ name: "Alice", age: 30 })); // Object received: {"name":"Alice","age":30}
console.log(processInput(() => "I'm a callback!")); // Function received: I'm a callback!
console.log(processInput(undefined)); // Unknown input type: undefined
```

- This is a vague example of why compiling JS in an AOT manner is difficult and inefficient, because the programmer can be stupid enough to pass different data types to a specific function, resulting in having to create an approximate variant of the type (speculative typing); you can see it in: https://webkit.org/blog/10308/speculation-in-javascriptcore/

#### First solution

The first solution would be to JIT compile to JS and find a way to link it at compile time with the output of the TypeScript compiler, for example, in the code generation part.

#### Second solution

The second solution would be to compile JS to WebAssembly, also known as TS, allowing them to be linked at link time in WASM. This is an easier and simpler solution, but it limits the capabilities of the compiler's JS model to the limitations of WASM.

#### Is it a lot of work?

Yes, but it can be minimized when choosing the language where we are going to carry it out.
