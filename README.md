This repo is no longer actively maintaned, I created this mostly as a learning tool for zig, and no longer have any interest in maintining/developing a glfw clone in zig, I will leave this archived in case anyone else would like to make their own version of it.
# zlfw
This package is meant to be entirely equivalent to using the C code direcly in terms of performance, while keeping a nice ziggified API.

The advantages it offers over C are pretty clear:
 * __Enums__, unlike GLFW, you always know what a function can accept, for example `window.getKey(.a)` instead of `c.glfwGetKey(window, c.GLFW_KEY_ESCAPE)`.
 * Slices instead of C pointers
 * [packed structs](https://ziglang.org/documentation/master/#packed-struct) for bit masks, allowing for `if (joystick.down)` instead of `if (joystick * c.GLFW_HAT_DOWN)`
 * Methods and structs instead of static functions, e.g. `window.iconify()` instead of `c.glfwIconifyWindow(window)`, this also removes the need for much of the error checking
 * `true` and `false` instead of `c.GLFW_TRUE` and `c.GLFW_FALSE`.
However, this is generally true of any zig binding, and is moreso a benefit of Zig over C. This package not only offers that, but also benefits compared to _other zig bindings_:
 * Direct reimplementation of the C code in Zig. The public glfw API contained in `glfw3.h` does not actually do much on it's own, much of the functionality is hidden in the `*.c` files in `src/`, most of the content in those files has been direcly reimplemented in Zig, meaning that using these bindings is _literally_ equivalent to the C code, and in a few cases, superior, since we can skip a good amount of the C error checking that is not necessary in Zig, one particularly nice example of this is the hints, where in Zig we can set the values direcly, without any switch statments on enums, unlike the C code which requires a switch for every hint
 * Error handling, see the corresponding section for more detail
 * Doc comments, every public piece of the API has a corresponding doc comment, taken from the [official documentation](https://www.glfw.org/).

 This package takes inspiration from [mach-glfw](https://github.com/thedeadtellnotales/mach-glfw/) in it's design,
 however, much of the actual code is vastly different, seeing as mach-glfw is a wrapper over `glfw3.h` and this is a reimplementation of the C files

## Usage
Add this package as a dependency to your `build.zig.zon`, you may do so directly or use the following command
```
zig fetch --save https://github.com/Batres3/zlfw/archive/refs/heads/main.tar.gz // Latest commit
zig fetch --save https://github.com/Batres3/zlfw/archive/COMMIT_HASH.tar.gz // Specific commit
```
Then, add the `zlfw` module to your `build.zig`
```
const zlfw = b.dependency("zlfw", .{
    .target = target,
    .optimize = optimize,
});
your_module.addImport("zlfw", zlfw.module("zlfw"));
```
You may now access `zlfw` anywhere in your module via `@import("zlfw")`! Basic usage examples can be seen in `src/test.zig`.

### How do I use OpenGL, Vulkan, etc. with this?
This is an implementation of glfw, and does not include any graphics api directly. You must bring your own library for your API of choice.

# Error handling
One of the main problems with glfw is that nearly every function in the API has the possibility of returning some error,
however, most of these are what I like to call trivial errors, for example `NotInitialized`, which a person will commit maybe once when first starting their development,
or `PlatformError` and `FeatureUnavailable/FeatureUnimplemented`, which are given when certain glfw features don't exist in a given platform,
again, these are basic errors that will not suddenly pop up in release, but only during development in debug.

Therefore, I have divided errors into two categories: those that are handled by zlfw (the ones mentioned above) and those that are
exposed to the user (via Zig errors!)
## Handled by zlfw
All of this error handling occurs _only_ in debug mode, because, as mentioned above, these are not "surprise" errors that may pop up at any time,
but rather basic questions of initialization, which will be caught and fixed in debug, or features being unimplemented in certain platforms.

So, in debug mode, if one of these errors is caught, the application will panic and send and error message containing the description.
In release mode, it will simply not perform the check, it will assume that you as the developer have checked that you are not calling functions
outside of their allowed scope and that you are not calling unimplemented functions (even if the error does pop up, the application will not panic)

Of course, some people may not want this, as such, the `error_check` build option has been added, which will disable these checks, and the error will simply
stay in the glfw buffer to be caught by either a `zlfw.errorCheck()` call or by using `zlfw.setErrorCallback()`.
## Exposed to the user
These are errors that are fairly unpredictable, things that may actually fail even in a release build, these are exposed via Zig errors
and must be handled by the user like any other Zig error.

Furthermore, the `zlfw.errorCheck()` and `zlfw.setErrorCallback()` functions are also exposed, although they should are not necessary, and I do not recommend most people use them.
