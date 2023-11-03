- Feature Name: The Blight Programming Language
- Start Date: 2023-11-03
- RFC PR: [blight-js/rfcs#0000](https://github.com/blight-js/rfcs/pull/1)
- Blight Issue:
  [blight-js/blight#0000](https://github.com/blight-js/blight/issues/1)

# Summary

[summary]: #summary

Javascript today feels very hard to write and this is no doubt due to a number
of bad decisions and bad features that continues to haunt the web to this day. I
believe we can do better as a community, if we start from scratch with a better
language. That being said, I strongly believe that Javascript as a language in
browsers is going nowhere and will stay the way it is forever.

Without browser support, it's impossible for us to get a good language in
browsers through traditional means, so until we get a big enough community to
back us entering browsers by default, we will happily transpile all Blight code
to Javascript that can run on the server or in the browser.

TLDR; Blight is Typescript 2, inspired by Rust syntax and semantics.

# Motivation

[motivation]: #motivation

We are doing this because we think that with the current landscape of browsers,
good language features are easy to come by, especially with inspiration from the
sources. We believe that a language that is designed around scripting with some
safety checks involved would be amazing and we are committed to making it
happen.

To be very clear, we do not plan on having any ecosystem to begin. An attack
strategy is being devised to get around that. We do want to support some sort of
interface with JavaScript and we plan to do that in a future proposal.

In the best timeline, once we get the compiler up and running, we are able to
create a runtime pretty fast and kick off a hackathon to get an ecosystem
running. If we get some key members of the JS community together, we may be able
to get most of the core necessary modules done pretty fast.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

This proposal is mainly to set up the motivation, goals, and non-goals of the
language. The motivation has been thoroughly explained in the previous section,
but let's go over goals and non-goals.

Goals:

- ESM-like modules
- JS-like async
- Rust-like syntax
- Macros
- Transpiles to JS
- Fast to compile and produces relatively fast JS
- Standard library to interact with raw JS APIs safely
- Zig-like "safe-but-slow" debug build and "fast-but-has-UB" production build

Non-Goals

- Support for any JS libraries outside of wrappers (would just be a worse
  Typescript)
- Compile to standalone executable (we'd just be writing Rust again)

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

The main consideration for this RFC on the technical side is how will this all
be implemented. My thinking at the current moment is that we write a
transpiler + type checker in Rust just for performance sake. I don't think there
is a universe where self-hosting makes sense unless we somehow get a compiler
backend.

We could choose other languages as well, but I believe that any other choice is
the wrong move:

- Zig: Cool technical features, but too unstable to use. Bun suffers actively
  with breaking changes.
- C++: It's C++. If I wanted to shoot myself in the foot I wouldn't be in
  support of Blight.
- C: Too low level, too much UB, hard to port.
- Javascript/Typescript/Python: Too slow, too big of a binary

# Drawbacks

[drawbacks]: #drawbacks

Maybe starting from scratch with no ecosystem is the wrong play. I could be
entirely wrong about this. Deno regretted starting from scratch and there is a
strong likelihood we will be in the same boat. If we can weather the storm, the
waters look much clearer on the other side.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

There are some alternatives that are tempting for sure. We could just... not do
this project and accept the status quo. We could make something else, maybe this
problem is too hard. I think "it's easier to do the other thing" is the wrong
way of making long-term decisions. I truly believe starting from scratch is the
right move here.

I have considered (and gotten decently far) at implementing alternative runtimes
for Javascript. I thought that maybe it was an API-level issue, but it's simply
not. At a fundamental language level, Javascript is broken. It is unsafe. It is
a footgun.

Not doing this would leave the world in the same state as it is today. Being a
web developer is seen as a nightmare because no one knows the idiosyncrasies of
Javascript. Except me. And maybe like... ten other people who really care.

# Prior art

[prior-art]: #prior-art

There is a lot of prior art on languages that transpile to JS. All but one
really succeeded, which is probably a bad sign. Let's go over each one I'm aware
of and their philosophies:

- Typescript: The only relevant player today. TS's philosophy is interopt,
  interopt, interopt. Uh oh.
- Coffeescript: Was revolutionary at it's time, but simply didn't get the
  adoption it deserved. It also didn't have a type system, so that might've made
  it a lot harder.
- Flow: Like Typescript, but with less focus on keeping the JS syntax. It's main
  downside was performance, it was really hecking slow.
- Rescript: Similar to what we're doing right now, but the semantics are
  completely off. The module system is awful, and their solution to interopt is
  really bad. I don't see this going anywhere.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

Everything is unresolved. Let's write the future.

# Future possibilities

[future-possibilities]: #future-possibilities

The entire language is future possibilities! I haven't proposed syntax on
purpose, as this is just to try and get the motivations and the philosophy
aligned with everyone who is working on this project. This isn't set in stone.
Let's build a language we all want to use, and let's build it together!

Just off of the top of my head, the following RFCs need to be made at some
point. All of these are unspec'd and may not even end up in the final language:

- How do native types work? Do we want something stricter or something looser?
- How do modules work?
- How do functions work?
- How does control flow work?
- How do errors work?
- How do macros work?
- How do unsafe blocks work, especially around the type system?
